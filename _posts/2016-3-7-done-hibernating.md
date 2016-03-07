---
layout: post
title: Out of Hibernation
comments: true
published: true
---


I took somewhat of a break from blogging over the Winter. I slowed down a bit in terms of my learning process and seeking opportunities.  I was feeling a bit discouraged about my progress and if I'd ever move on to something new.  I took a somewhat conscious step of not worrying so much and just letting things come naturally.  I can say that even though I haven't been super active in terms of coding and development, I am a lot more comfortable with where I am.  

Today is seems to be the first day in a series of warmer weather, I figure it is time to provide an update and look forward to the spring ahead. So what have I been doing?

### Cooking

  I've been very proud of some of the dishes I made over the winter.  Understandably, there was a focus on soups and stews.  Here are some of the things I've made over the past few months:
  
  * [Puerco Guisado en Leche con Frijoles y Chochoyotes](http://www.comalcaliente.com/2015/12/puerco-guisado-en-leche-con-frijoles.html)
  * [Two Soups, One Broth](http://www.comalcaliente.com/2016/01/two-soups-one-broth.html)
  * [Random Casserole](http://www.comalcaliente.com/2016/01/random-casserole.html)
  * [Guisado de Puerco](http://www.comalcaliente.com/2016/01/guisado-de-puerco.html)
  * [New Salsa](http://www.comalcaliente.com/2016/02/new-salsa.html)
  * [Lamb Birria Ramen](http://www.comalcaliente.com/2016/02/lamb-birria-ramen.html)
  * [Leftover Fideo Soup](http://www.comalcaliente.com/2016/02/leftover-fideo-soup.html)

### Recetera

  I recently made a few more updates to Recetera.  The biggest change has been updating recipes to wither be "Published" or Unpublished".  That is to say a user can choose to have their recipe be part of the main recipe list, or only have it show underneath their profile under "Unpublished Recipes".  This is a little on how I accomplished this:
  
1. I began by generating a new migration to add a `published` boolean to my `Recipe` model. I set the default for this to `false` so that it will only get published when it is selected by the user.

{% highlight ruby %}
# recetera/db/migrate/20160118182312_add_published_to_recipes.rb

class AddPublishedToRecipes < ActiveRecord::Migration
  def change
    add_column :recipes, :published, :boolean, default: false
  end
end
{% endhighlight %}

2. I updated my `Recipe` form view in under to send a value of `true` or `false` to the `published` parameter.  This will get passed to the controller to update the current recipe.

{% highlight ruby %}
#recetera/app/views/recipes/_form.html.erb

<%= form_for @recipe do |f| %>
...
<%= f.button "Publish", :name => "published", :value => "true" %>
    <%= f.button "Save Draft", :name => "published", :value => "false" %>
  </div>
<% end %>
{% endhighlight %}

3. Finally, I updated the controllers `create` and `update` methods to switch the `published` value accordingly.

{% highlight ruby %}
# recetera/app/controllers/recipes_controller.rb

class RecipesController < ApplicationController
...
def create
    params[:recipe][:instructions] = params[:recipe][:instructions].to_json
    @recipe = current_user.recipes.build(recipe_params)
    @recipe.published = true if params[:published] == "true"

    if @recipe.save
      redirect_to @recipe
    else
      render "new"
    end
  end

  def update
    params[:recipe][:instructions] = params[:recipe][:instructions].to_json
    @recipe = Recipe.find(params[:id])
    @recipe.published = true if params[:published] == "true"
    @recipe.toggle(:published) if params[:published] == "false"

    if @recipe.update(recipe_params)
      redirect_to @recipe
    else
      render "edit"
    end
  end
end
{% endhighlight %}

With these updates, I can now have the user determine if a recipe should be published.  As a final step to simplify views when running a collection, I added the following to my model: 

{% highlight ruby %}
# recetera/app/models/recipe.rb

class Recipe < ActiveRecord::Base
...
def self.published_recipes
    Recipe.where("published = ?", true)
  end

  def self.unpublished_recipes
    Recipe.where("published =?", false)
  end
end
{% endhighlight %}

### Cryptopals

  This is something I keep going back to and I recently am on another kick.  I realized that I didn't actually save my response to challenge 6 of set 1, this required a lot of refactoring and basically doing the challenge all over again.  It was almost a good thing I didn't fully right it down, because it put me in a place where I had to play around with Ruby a lot to make sure I was getting the responses necessary to solve it.  Here are a few things I learned as a result:
