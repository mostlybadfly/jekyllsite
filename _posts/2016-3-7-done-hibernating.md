---
layout: post
title: Out of Hibernation
comments: true
published: true
---


I took somewhat of a break from blogging over the winter. I slowed down a bit in terms of my learning process and seeking opportunities.  I was feeling a bit discouraged about my progress and if I'd ever move on to something new.  I took a somewhat conscious step of not worrying so much and just letting things come naturally.  I can say that even though I haven't been super active in terms of applying, I am a lot more comfortable with what I have been doing.  Looking back, I'm somewhat pleased witht he things I have done in spite of supposedly "laying low" all winter.  

Today is seems to be the first day in a series of warmer weather, I figure it is time to provide an update and look forward to the spring ahead. So what have I been doing? What is up next?

### Cooking

  I've been very proud of some of the dishes I made over the winter.  Understandably, there was a focus on soups and stews.  Here are some of the things I've made over the past few months:
  
  * [Puerco Guisado en Leche con Frijoles y Chochoyotes](http://www.comalcaliente.com/2015/12/puerco-guisado-en-leche-con-frijoles.html)
  * [Two Soups, One Broth](http://www.comalcaliente.com/2016/01/two-soups-one-broth.html)
  * [Random Casserole](http://www.comalcaliente.com/2016/01/random-casserole.html)
  * [Guisado de Puerco](http://www.comalcaliente.com/2016/01/guisado-de-puerco.html)
  * [New Salsa](http://www.comalcaliente.com/2016/02/new-salsa.html)
  * [Lamb Birria Ramen](http://www.comalcaliente.com/2016/02/lamb-birria-ramen.html)
  * [Leftover Fideo Soup](http://www.comalcaliente.com/2016/02/leftover-fideo-soup.html)
  
  Plans for future dishes include some favorites that my mother used to make during Lent.  While I am not religious at all, these dishes have had a huge impact on me and are some I remember the most fondly.  These include enchiladas, tortitas de camarón, taquitos de atún, and migas.  

### Recetera

  I recently made a few more updates to Recetera.  The biggest change has been updating recipes to wither be "Published" or Unpublished".  That is to say a user can choose to have their recipe be part of the main recipe list, or only have it show underneath their profile under "Unpublished Recipes".  This is a little on how I accomplished this:
  
  I began by generating a new migration to add a `published` boolean to my `Recipe` model. I set the default for this to `false` so that it will only get published when it is selected by the user.

{% highlight ruby %}
# recetera/db/migrate/20160118182312_add_published_to_recipes.rb

class AddPublishedToRecipes < ActiveRecord::Migration
  def change
    add_column :recipes, :published, :boolean, default: false
  end
end
{% endhighlight %}  

  I updated my `Recipe` form view in under to send a value of `true` or `false` to the `published` parameter.  This will get passed to the controller to update the current recipe.

{% highlight ruby %}
#recetera/app/views/recipes/_form.html.erb

<%= form_for @recipe do |f| %>
...
<%= f.button "Publish", :name => "published", :value => "true" %>
    <%= f.button "Save Draft", :name => "published", :value => "false" %>
  </div>
<% end %>
{% endhighlight %}

  Finally, I updated the controllers `create` and `update` methods to switch the `published` value accordingly.

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

  I plan to continue with Recetera by updating some of my forms and views visible to the user.  I also want to go back and refactor some controllers and models.  I have not gone back specifically to refactor so it would be interesting what sorts of changes I will make armed with additional Ruby knowledge.  Eventually in the coming months, I want to focus more on front end things, making the site more reactive, more "javascripty", this will be a great way to practice JS as well as design.

### Cryptopals

  This is something I keep going back to and recently I have been on another kick.  I realized that I didn't actually save my response to challenge 6 of set 1, this required a lot of refactoring and basically doing the challenge all over again.  It was almost a good thing I didn't fully write it down, because it put me in a place where I had to play around with Ruby a lot to make sure I was getting the responses necessary to solve it.  
  
  The latest challenge I worked on involved breaking repeating key xor cipher.  According to the group giving the challenge, it is one of the bigger hurdles in the challenges and solving it will arm me with the tools to solve the rest.  I definitely noticed myself separating out responsibilities more often than I did in the past: shorter methods that do only one thing versus multiple, using methods as returns vs returning a temporary variable, quickly being able to assess how an array or string can be manipulated.
  
  I hope to be done with the first set this week and proceed on to the next one.  This is something that fascinates me deeply.  While I don't have much experience with security, the idea of manipulating data and deciphering these puzzles really appeals to me.  It has been one of the more exciting and rewarding things I have gotten to do since learning to program.
  
### Well-Grounded Rubyist

  What a fantastic book and resource. I read this through a second time over the winter and I feel that I got a deeper understanding of the various elements I've been learning all this time.  It seemed that descriptions of String and Array manipulation clicked a lot more.  A huge benefit has been the way I view enumerables.  They seem to come more naturally to me know and really have become not only a way to iterate over objects, but also a way for me to understand what my code is doing.  It organizes what I'm doing in a neat and efficient manner, and I have been able to switch out syntax as needed to obtain the intended result, versus just getting frustrated at how these "loops" are supposed to work.  Ruby has been really fun to me, but now I see more and more how powerful it truly is.
  
  Books up next include a re-read of POODR, then Eloquent Ruby, and The Pragmatic Programmer.  I have created a list of books I plan to read on Goodreads that you can check out [here](https://www.goodreads.com/mostlybadfly).
