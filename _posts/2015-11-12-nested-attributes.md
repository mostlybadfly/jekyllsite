---
layout: post
title: Nested Attributes and a has_many :through Relationship.
comments: true
---

Over the past week, I have been working on upated [Recetera](https://github.com/mostlybadfly/recetera), my recipe app.  This is somewhat of a toy project I've been working on for a while to keep challenging myself and to learn new aspects of Rails and developing a web app.  Some of the recent updates included updating the Gemfile and adding some partials to clean up 'layouts/application.html.erb'.

What is probably the more complicated change has been creating a new model in addition to `Recipe` and `Ingredient`: `RecipeItem`.  The need for this new model came about because I was storing amount attributes beneath `Ingredient` and I wanted an intermediary object to hold onto that information when it is relevant to `Recipe`.  I still wanted to keep `Ingredient` separate for later use as a way to look up recipes using a certain ingredient, or to keep inventory of ingredients in stock.  Apart from this, I wanted a way for a new `Ingredient` to be saved to the database when a user submits a new `Recipe` with said `Ingredient` in it's nested fields.

To begin, I created a new migration as seen below.  I wanted this to reference both `Recipe` and `Ingredient`, it seems like this makes it a "joining model" according to different sources, that seems fitting since it is the meeting point between these two models in a way. I also added the `:amount` attribute as a string for entries such as "1 Cup" or "2 Tablespoons".

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

I next added the appropriate associations in the models.  Since this was a change to the already existing models they all had a little change. First, the newest model, `RecipeItem`.  I set this to belong to both `Recipe` and `Ingredient` so that the corresponding attributes for `RecipeItem` are referenced for each ingredient in the recipe.  I have also set `accepts_nested _attributes_for :ingredient` for later use in our form.  This makes it possible for me to include/create a new `Ingredient` when building my `RecipeItem` in the eventual form.

{% highlight ruby %}
class RecipeItem < ActiveRecord::Base
  belongs_to :recipe
  belongs_to :ingredient
  accepts_nested_attributes_for :ingredient
end
{% endhighlight %}

For both `Recipe` and `Ingredient` models I have them set to both have many `RecipeItem`s. I also used the `has_many :through` association so that these models are both linked to each other through `RecipeItem` so that they can reference each other.

{% highlight ruby %}
class Recipe < ActiveRecord::Base
  ...
  has_many :recipe_items
  has_many :ingredients, through: :recipe_items
  accepts_nested_attributes_for :recipe_items
  ...
  end
{% endhighlight %}

{% highlight ruby %}
class Ingredient < ActiveRecord::Base
  has_many :recipe_items
  has_many :recipes, through: :recipe_items
end
{% endhighlight %}

My `RecipesController` is below. This needed two changes.  First, I added build methods to @recipe so that both `RecipeItem` and `Ingredient` are being built so that I can actually use them to create a new `Recipe`.  I have also updated `recipe_params` to take the nested attributes.  Notice how the relevant attributes are nested within each other. Without this the `recipe_items` and `ingredient` within that will not save to the database.

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

Next to the forms.  First, with in my `@recipe` form, I call `fields_for :recipe_items` and render another partial specifically for this model to be build.  Notice `link_to_add_association` which is a helper method provided by the [cocoon gem](https://github.com/nathanvda/cocoon) to dynamically add a new set of `recipe_item_fields`.

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

Within `recipe_item_fields` I included a text field for the amount as well as nested `fields_for :ingredient` to capture the name and create a new `Ingredient`.  In between these, the line `<% f.object.build_ingredient unless f.object.ingredient %>` is used in order to build the association so that when `link_to_add_assocaition` is clicked, both fields render as a group.  The `unless` statement is there so that it doesn't reset the fields in other views such at edit.

{% highlight erb %}
<div class="nested-fields form-inline">
  <%= f.label :amount %>
  <%= f.text_field :amount, :class => 'form-control' %>
  
  <% f.object.build_ingredient unless f.object.ingredient %>

  <%= f.fields_for :ingredient do |ff| %>
    <%= ff.label :name %>
    <%= ff.text_field :name, :class => 'form-control' %>
  <% end %>
  <%= link_to_remove_association "remove ingredient", f %>
</div>
{% endhighlight %}

Finally, the show view renders once the new recipe is submitted.  Here I am calling `@recipe.recipe_items` as a collection under which you can see that `recipe_item.ingredient.name` is being used in order to pull the name of the `Ingredient` for each specific `Recipe`.

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

Admittedly, setting up the models and figuring out the associations was somewhat frustrating. I sought out help from some friends and I found that just discussing the issues involved gave me perspective on what needed to be done next.  I also found very little posts on setting up these types of associations, so I wanted to include my process specific for Recetera.  I definitely would like to hear others' thoughts on this and get feedback on how I have these set up if possible.
