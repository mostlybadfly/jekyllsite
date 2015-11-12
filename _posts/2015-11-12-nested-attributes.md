---
layout: post
title: Nested Attributes and a has_many :through Relationship.
comments: true
---
Working on adding a new model so that Ingredient can be saved individually, but still associated with it's recipes.

{% highlight ruby %}
#Edited to fit with what eventually  was added to the database. Accidentally associated with User and set :amount as an integer.
class CreateRecipeItems < ActiveRecord::Migration
  def change
    create_table :recipe_items do |t|
      t.string :amount
      t.references :recipe, index: true
      t.references :ingredient, index: true

      t.timestamps
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
class Recipe < ActiveRecord::Base
  belongs_to :user

  has_many :recipe_items
  has_many :ingredients, through: :recipe_items
  accepts_nested_attributes_for :recipe_items

  validates :title, presence: true
  validates :instructions, presence: true
end
{% endhighlight %}

{% highlight ruby %}
class Ingredient < ActiveRecord::Base
  has_many :recipe_items
  has_many :recipes, through: :recipe_items
end
{% endhighlight %}

{% highlight ruby %}
class RecipeItem < ActiveRecord::Base
  belongs_to :recipe
  belongs_to :ingredient
  accepts_nested_attributes_for :ingredient
end
{% endhighlight %}

{% highlight ruby %}
class RecipesController < ApplicationController
...
 def new
   @recipe = Recipe.new
   @recipe.recipe_items.build.build_ingredient
  end
...
private
    def recipe_params
      params.require(:recipe).permit(:title, :instructions,
                                     recipe_items_attributes: [:id, :amount, 
                                     ingredient_attributes:[:name]])
    end
end
{% endhighlight %}

{% highlight erb %}
<%= form_for @recipe do |f| %>
...
  <div id="ingredients" class="form-group">
    <label>Ingredients:</label>
    <%= f.fields_for :recipe_items do |builder| %>
      <%= render 'recipe_item_fields', f: builder %>
    <div id="links">
      <%= link_to_add_association "add Ingredient", f, :recipe_items %><br/>
    </div>
    <% end %>
  </div>
...
<% end %>
{% endhighlight %}

{% highlight erb %}
<div class="nested-fields form-inline">
  <%= f.label :amount %>
  <%= f.text_field :amount, :class => 'form-control' %>

  <%= f.fields_for :ingredient do |ff| %>
    <%= ff.label :name %>
    <%= ff.text_field :name, :class => 'form-control' %>
  <% end %>
  <%= link_to_remove_association "remove ingredient", f %>
</div>
{% endhighlight %}

{% highlight erb %}
...
<p>
  <strong>Ingredients:</strong><br/>
  <% @recipe.recipe_items.each do |recipe_item| %>
    <%= "#{recipe_item.amount} #{recipe_item.ingredient.name}" %><br/>
  <% end %>
</p>
...
{% endhighlight %}
