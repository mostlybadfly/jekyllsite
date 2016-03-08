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

{% highlight ruby %}
# giggle_water/app/models/drink.rb

class Drink < ActiveRecord::Base
  has_many :drink_items, dependent: :destroy
  has_many :ingredients, through: :drink_items
...
end
{% endhighlight %}

{% highlight ruby %}
# giggle_water/config/routes.rb
Rails.application.routes.draw do
  authenticate :user do
    get '/search', to: 'search#search'
...
end
{% endhighlight %}

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
