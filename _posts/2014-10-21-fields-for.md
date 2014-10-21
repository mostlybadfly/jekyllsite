---
layout: post
title: fields_for from FormHelper for a single attribute.
comments: true
---

  So I'm currently working on a recipe app and I'm starting to work on a simple form so that I can get a working idea of how user input might look.  I also wanted to see how this input gets interpreted and saved to the model.  Using a Rails form helper, I built a form for various fields of a recipe including :title, :author, and :cuisine.  A simple text_area to input these worked just fine, but then I came across an issue:  what to do about :ingredients.  The problem I was facing is that I want to give the option of selecting 3 different fields to complete the input for :ingredients which is set up in my model as text. After researching the Rails API, I found another helper called 'fields_for'.  This creates a set of fields tied to a single parameter. This is what it would look like within my form:
  
{% highlight erb %}
<%= form_for :recipe, url: recipes_path do |f| %>
  ...
  <%= f.fields_for :ingredients do |ff|%> 
    <%= ff.text_area :qty, :class => 'formfield' %> 
    <%= ff.text_area :measurement, :class => 'formfield' %> 
    <%= ff.text_area :ingredient, :class => 'formfield' %> 
  <% end %>
  ...
<% end %>
{% endhighlight %}

  I implemented this and the form loaded fine in the view, great! However, my input for :ingredients was not showing. This is what the parameters being passed look like when I use 'render plain: params[:recipe].inspect':

{% highlight ruby %}
{"title"=>"Sandwich",
 "author"=>"mostlybadfly",
 "cuisine"=>"Lunch"
 "servings"=>2
 "ingredients"=>{"qty"=>"2", "measurement"=>"slices", "ingredient"=>"bread"}}
{% endhighlight %}

  It looks like it is doing what I wanted. I got 3 bits of information to use.  The problem that I was running into with is was that :ingredients won't actually save because it isn't set as a single string which is what the model is expecting. I first assumed fields_for wasn't what I wanted so I kept going in circles trying to figure it out.  I was getting nowhere. As with most things, I think having a long time away helped me think about it a bit more.  I realized that if params[:recipe] is actually the hash listed above, I should just be able to play around with the :ingredients key and reassign it to a single string prior to saving. This is what I ended up with in the recipes controller:

{% highlight ruby %}
def create 
  params[:recipe][:ingredients] = params[:recipe][:ingredients].values.join(' ') 
  @recipe = Recipe.new(recipe_params) 
  @recipe.save 
  redirect_to @recipe 
end 
{% endhighlight %}

  What I did here was update the value related to the :ingredients key prior to passing the parameters to the Recipe.new(recipe_params) so that it could be interpreted as the string it requires.  After going to rails console and checking the database record I could see that the :ingredients is listed at "2 slices bread", success! The string saved to the model for later use.
  
  My next challenge is going to be to include multiple fields, because most recipes involve more than one recipe. The end goal for ingredients would be to save an array of ingredients strings that can be displayed in the view.  I will also be working on an ingredients model which will draw from params[:recipe][:ingredients][:ingredient] to create a relationship between Recipe and Ingredient.
