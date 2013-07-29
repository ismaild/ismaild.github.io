---
layout: post
title: "Editing MongoDB Arrays in Rails"
date: 2013-07-29 22:32
comments: true
categories: [mongodb, ruby, rails]
---
One of the awesome features of MongoDB is the ability to store arrays directly in your documents, which maps directly with ruby/python/js arrays etc. 

An example Post model:

{% codeblock data.json %}
{
  title: 'Editing MongoDB Arrays in Rails'
  tags: ['ruby', 'rails', 'mongodb']
}
{% endcodeblock %}

Though when working with MongoDB arrays in rails and forms to edit the data, you could end up doing a bunch of controller logic and/or js logic just to get the data in.

This is especially true if you want to just store a bunch of strings in the array. I tried many different ways of doing this, but the simplest solution with out any changes to your controller or forms is to just use a virtual attribute on your model.

{% codeblock Post.rb %}
class Post
  include Mongoid::Document

  field :title, type: String
  field :tags, type: Array, default: []

  def tags_list=(arg)
    self.tags = arg.split(',').map { |v| v.strip }
  end

  def tags_list
    self.tags.join(', ')
  end
end
{% endcodeblock %}

Then in your view you just use tags_list:

{% codeblock _form.html.erb %}
<%= f.text_field :tags_list %>
{% endcodeblock %}

In the shell:

{% codeblock lang:ruby %}
post = Post.new
post.tags_list = 'tag1, tag2, tag3'
=> "tag1, tag2, tag3" 

post.tags
=> ["Tag1", "Tag2", "Tag3"]  
{% endcodeblock %} 

Any comma seperated values entered will be stored correctly in MongoDB, you could also use this to apply any transformations to the data before it saved, i.e captilizing each value.

We ended up using this bit of code quite often, so i extracted it out to lib, and then [packaged it into a gem](https://github.com/ismaild/mongoid-arraylist).

