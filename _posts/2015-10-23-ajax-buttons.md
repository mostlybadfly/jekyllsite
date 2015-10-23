---
layout: post
title: Ajax Buttons
comments: true
---

Over the past few months I've been helping my friend with a cocktail recipe app called [Giggle Water](https://github.com/tomekr/giggle_water). It has been great contributing and learning new things about Rails and getting first hand experience contributing to the development of an application.  One of the more complicated things I've had to do required creating 'add' or 'remove' buttons depending on whether or not a drink ingredient was already in a user's bar.  This had to be done without a page reload and also provide a flash notification when a drink was added or removed from the bar.

To begin, the button existed on the ingredients index page.  Where the button will render, I created a partial to render an 'Add' or 'Remove' button depending on context.

{% highlight erb %}
# app/views/ingredients/index.html.erb 
<% @ingredients.each do |ingredient| %>
  <tr>
    <td id="<%=ingredient.id%>">
      <% if current_user %>
        <%= render 'add_or_remove', ingredient: ingredient %>
      <% end %>
    </td>
    <td>
      <%= link_to ingredient.name, ingredient_path(ingredient) %>
    </td>
    <td><%= ingredient.drink_items_count %></td>

  </tr>
<% end %>

# /app/views/ingredients/_add_or_remove.html.erb
<% if current_bar %>
  <% if current_bar.bar_items.find_by_ingredient_id(ingredient.id) %>
    <%= button_to "Remove", {:controller => :ingredients, :action => 'remove_from_bar', :id => ingredient.id }, :class => 'btn btn-block btn-sm btn-danger', :method => :delete, remote: true %>
  <% else %>
    <%= button_to "Add", {:controller => :ingredients, :action => 'add_to_bar', :id => ingredient.id }, :class => 'btn btn-block btn-sm btn-success', :method => :post, remote: true %>
  <% end %>
<% end %>
{% endhighlight %}

In my partial there are some new things to account for.  For one are the actions of each button.  I knew I had to create both a `add_to_bar` and `remove_from_bar` in my Ingredients controller. Next, I'm using the `ingredient.id` for that table row to pass to the controller since that is the one I'm modifying.  The `:class` is for use with Bootstrap, basically it is making a green or red button for Add or Remove. Finally at the end `remote: true` makes it possible for the button to be submitted via ajax.  This information(apart fromt he bootstrap tags) can be seen in the Ingredients controller: 

{% highlight ruby %}

# app/controllers/ingredients_controller.rb 

class IngredientsController < ApplicationController
  ...
  def add_to_bar
    bar_item = current_user.current_bar.bar_items.build(ingredient: @ingredient)

    respond_to do |format|
      if bar_item.save
        format.js { render action: "add_or_remove", locals: { message: "Added to bar!" } }
      else
        format.json { render json: @ingredient.errors, status: :unprocessable_entity}
      end
    end
  end

  def remove_from_bar
    bar_item = current_user.current_bar.bar_items.find_by(ingredient: @ingredient)
    bar_item.destroy
    
    respond_to do |format|
      format.html { redirect_to @ingredient, notice: 'Ingredient was removed from bar' }
      format.js { render action: "add_or_remove" , locals: { message: "Removed from bar!"} }
    end
  end
  ...
end
{% endhighlight %}
