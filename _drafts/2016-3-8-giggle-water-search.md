---
layout: post
title: Adding Search to Giggle Water
comments: true
---

  I wanted to break out this post as something separate from yesterday's update post since it involved a bit more work and a more significant change for the Giggle Water app I've been working on with my friend, Tomek.  The goal here was to include a search so that a user can easily find drinks by the name or ingredient that they like.  I had not done this type of addition to an app at this point and I looked forward to figuring it out.  I learned a lot in the process.

I began by adding updating routes so that search results will show under `/search`.
{% highlight ruby %}
# giggle_water/config/routes.rb
Rails.application.routes.draw do
  authenticate :user do
    get '/search', to: 'search#search'
...
end
{% endhighlight %}

Next came maybe what was the more difficult part of this update: the actual queries done to return results for our search parameters. Getting `drink_name` and `ingredient_name` were pretty straightforward as it was literally searching the name attribute by the search term being submitted.  It became pretty apparent that we were not getting all the results when a search for "whiskey" only produced 3 drink results.  The problem was that while drink names containing "whiskey" were being returned, drinks containing the ingredient "whiskey" were not. 

Figuring out how to do the proper query was one of the more interesting and fun parts of setting this up.  With Tomek's help we tried various ways to join tables in our search while still making it secure. What we eventually did was join `Drink` to `Ingredient` where they intersect at `DrinkItem`, that is to say `Drink.joins(drink_items: :ingredient).where("ingredients.name like ?", "%#{search}%" )`.  However, in order to actually get this to return, we needed to update our model association, which we looked at next.
{% highlight ruby %}
# giggle_water/app/controllers/search_controller.rb

class SearchController < ApplicationController
  def search
    @drink_results = drink_name_search(params[:search])
    @drinks_with_ingredient = joined_drink_search(params[:search])
    @ingredient_results = ingredient_search(params[:search])
  end

  def drink_name_search(search)
    Drink.where("name LIKE ?", "%#{search}%")
  end
  
  def ingredient_search(search)
    Ingredient.where("name LIKE ?", "%#{search}%")
  end

  def joined_drink_search(search)
    Drink.joins(drink_items: :ingredient).where("ingredients.name like ?", "%#{search}%" )
  end
end
{% endhighlight %}

The below association of `has_many :ingredients, through: :drink_items` is what allows the above query to work.  This states that `Drink` is now associated to `Ingredient` via the `DrinkItem` already associated to model.
{% highlight ruby %}
# giggle_water/app/models/drink.rb

class Drink < ActiveRecord::Base
  has_many :drink_items, dependent: :destroy
  has_many :ingredients, through: :drink_items
...
end
{% endhighlight %}

Next to the front end of this: we created a `search_field_tag` to create a text field which will pass `params[:search]` to the `SearchController`.  Once the user hits submit, the result will be our `/search` results page.
{% highlight erb %}
# giggle_water/app/views/layouts/_navigation.html.erb

<nav class="navbar navbar-inverse navbar-fixed-top">
...
        <%= form_tag(search_path, :method => "get", class: "navbar-form navbar-left") do %>
          <%= search_field_tag :search, params[:search], size: 30, placeholder: "Search drinks & ingredients", class: "form-control" %>
          <%= submit_tag "Search", class: "btn btn-primary" %>
        <% end %>
...
</nav>
{% endhighlight %}

I included the full code for this one mostly to demonstrate how crazy a bootstrap accordion layout can get.  I found myself getting lost in it at times.  There was some going back and forth to make sure the ids were being set up correctly and panels were collapsing as they should. The way the results are laid out below is that the user will get 3 panels: one for drink results, one for drinks with ingredients results, and one with just ingredients names.  When a user clicks on one panel, it collapses the others to save space and make navigation a little easier. Finally, we added a number next to each panel heading to indicate how many results in each category a user can expect.
{% highlight erb %}
# giggle_water/app/views/search/search.html.erb

<div class="container">
  <h2>Your Results for "<%= params[:search] %>"</h2>
  <div class="panel-group" id="accordion" role="tablist" aria-multiselectable="true">
    <div class="panel panel-default">
      <div class="panel-heading" role="tab" id="headingOne">
        <h4 class="panel-title">
          <a role="button" data-toggle="collapse" data-parent="#accordion" href="#collapseOne" aria-expanded="true" aria-controls="collapseOne">
            Drink Results (<%= @drink_results.count %>)
          </a>
        </h4>
      </div>
      <div id="collapseOne" class="panel-collapse collapse in" role="tabpanel" aria-labelledby="headingOne">
        <div class="panel-body">
          <table class="table table-bordered table-hover">
            <thead>
              <th>Name</th>
              <th>Ingredients</th>
            </thead>
            <tbody>
              <% @drink_results.each do |drink| %>
                <tr data-link="<%= drink_path(drink) %>">
                  <td><%= drink.name %></td>
                  <td><%= render 'drinks/minimal_drink', drink: drink %></td>
                </tr>
              <% end %>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading" role="tab" id="headingTwo">
        <h4 class="panel-title">
          <a class="collapsed" role="button" data-toggle="collapse" data-parent="#accordion" href="#collapseTwo" aria-expanded="false" aria-controls="collapseTwo">
            Drinks with Ingredient <%= params[:search] %> (<%= @drinks_with_ingredient.count %>)
          </a>
        </h4>
      </div>
      <div id="collapseTwo" class="panel-collapse collapse" role="tabpanel" aria-labelledby="headingTwo">
        <div class="panel-body">
          <table class="table table-bordered table-hover">
            <thead>
              <th>Name</th>
              <th>Ingredients</th>
            </thead>
            <tbody>
              <% @drinks_with_ingredient.each do |drink| %>
                <tr data-link="<%= drink_path(drink) %>">
                  <td><%= drink.name %></td>
                  <td><%= render 'drinks/minimal_drink', drink: drink %></td>
                </tr>
              <% end %>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading" role="tab" id="headingThree">
        <h4 class="panel-title">
          <a class="collapsed" role="button" data-toggle="collapse" data-parent="#accordion" href="#collapseThree" aria-expanded="false" aria-controls="collapseThree">
            Ingredients Results (<%= @ingredient_results.count %>)
          </a>
        </h4>
      </div>
      <div id="collapseThree" class="panel-collapse collapse" role="tabpanel" aria-labelledby="headingThree">
      <div class="panel-body">
        <table class="table-condensed table-hover table-bordered">
          <thead>
            <tr>
              <th></th>
              <th>Ingredient</th>
              <th># of Cocktails</th>
            </tr>
          </thead>
          <tbody>
            <% @ingredient_results.each do |ingredient| %>
              <tr>
                <td id="<%=ingredient.id%>">
                  <% if current_user %>
                    <%= render 'ingredients/add_or_remove', ingredient: ingredient %>
                  <% end %>
                </td>
                <td>
                  <%= link_to ingredient.name, ingredient_path(ingredient) %>
                </td>
                <td><%= ingredient.drink_items_count %></td>
              </tr>
            <% end %>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</div>
{% endhighlight %}

  I'm very pleased with how this came out.  This was something I would go back to and play around with over the course of a month, but I would learn something different every time.  When we eventually met up to make the final changes and really kick it out, we realized we spent almost all of our time figuring out how Active Record queries worked.  We laughed a bit at how there wasn't any real tangible change on the front end but the result was a more efficient search and a deeper understanding of how various aspects of Rails queries worked.
