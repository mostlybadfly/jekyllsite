---
layout: post
title: revisiting fields and nested attributes in rails
comments: true
---

  I have been working more on Recetera lately, and I wanted to provide a bit of an update on what I've been doing.  This mostly has to do with all I've learned and ran into when working with nested attributes and fields.

  Since my last post regarding the recipe app, I created an `Ingredient` model with `quantity` and `name` attributes.  This belongs to `Recipe` which can use the attributes for `Ingredient`.  My models are below:

{% highlight ruby %}
class Recipe < ActiveRecord::Base
  has_many :ingredients

  accepts_nested_attributes_for :ingredients, allow_destroy: true
  end
{% endhighlight %}
{% highlight ruby %}
class Ingredient < ActiveRecord::Base
  belongs_to :recipe
end
{% endhighlight %}

  The line `accepts_nested_attributes_for :ingredients, allow_destroy: true` makes it possible for `Recipe` to save with attributes belonging to `Ingredient` as well as allows for these nested attributes to be deleted independently of the associated model. It is worth noting that when generating the model for `Ingredient`, I made sure to include `recipe_id:integer` as an attribute.  This allows for the association to occur between the models.

  When building the form to use for a new recipe, I created a separate set of fields impacting the `Ingredient` model like so:

{% highlight erb %}
<%= form_for @recipe do |f| %>
  <div class="form-group">
    <%= f.label :title %>:
    <%= f.text_field :title, :class => 'form-control' %>
  </div>

  <div class="form-group">
    <label>Ingredients:</label>
    <%= f.fields_for :ingredients do |builder| %>
      <%= render 'ingredient_fields', f: builder %>
    <% end %>
    <%= link_to_add_fields "Add Ingredient", f, :ingredients %><br/>
  </div>

  <div class="form-group steps">
    <%= f.label :instructions %>:
    <% unless @instructions.nil? %>
      <% @instructions.each do |instruction| %>
        <%= text_field_tag 'recipe[instructions][]', instruction %>
      <% end %>
    <% end %>
    <%= text_field_tag 'recipe[instructions][]', nil, class: 'form-control', placeholder: 'Type the first step' %>
  </div>
  <%= link_to "Next Step", "javascript:void(0)", id: "new_step" %><br/>
  <div class="form-group"><br/>
    <%= f.submit "Submit", :class => 'btn btn-default' %>
  </div>
<% end %>
{% endhighlight %}

  In the ingredients partial:

{% highlight erb %}
<div class="form-inline">
  <%= f.label :quantity %>
  <%= f.text_field :quantity, :class => 'form-control' %>

  <%= f.label :name %>
  <%= f.text_field :name, :class => 'form-control' %>
</div>
{% endhighlight %}

  With this set up, when a user submits the form, the ingredients attributes `:quantity` and `:name` both become associated witht he `recipe_id` of the new recipe we are creating. I actually ran into a bit of difficulty getting the association for work, but this is ultimately what worked with me after playing around with it for a while.

  If you wish to review the source, it is available on github.  It is a work in progress but please check it out ![here](http://www.github.com/mostlybadfly/recetera) .

  I will be providing other short posts on some other updates and things I've been learning including serializing the instruction input and headaches involving incorporation of javascript, which is still a little slow going for me.
