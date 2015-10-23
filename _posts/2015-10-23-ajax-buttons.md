---
layout: post
title: Ajax Buttons
comments: true
---

Over the past few months I've been helping my friend with a cocktail recipe app called [Giggle Water](https://github.com/tomekr/giggle_water). It has been great contributing and learning new things about Rails and getting first hand experience contributing to the development of an application.  One of the more complicated things I've had to do required creating 'add' or 'remove' buttons depending on whether or not a drink ingredient was already in a user's bar.  This had to be done without a page reload and also provide a flash notification when a drink was added or removed from the bar.  This would involve an ajax request and the use of JavaScript to change elements in the view.

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

In my partial there are some new things to account for.  For one are the actions of each button.  I knew I had to create both a `add_to_bar` and `remove_from_bar` in my Ingredients controller. Next, I'm using the `ingredient.id` to create a unique html id attribute for use in my JavaScript.  The `:class` is for use with Bootstrap, basically it is making a green or red button for Add or Remove. Finally at the end `remote: true` makes it possible for the button to be submitted via ajax.  This information(apart fromt he bootstrap tags) can be seen in the Ingredients controller: 

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
      format.js { render action: "add_or_remove" , locals: { message: "Removed from bar!"} }
    end
  end
  ...
end
{% endhighlight %}

For the `add_to_bar` method, we are creating a `bar_item` by using the current `ingredient` and adding to to the user's bar items.  After this saves, we use a `format.js` response to render an action related to a `add_or_remove.js.erb` file, along with a message we are passing that will help us render the flash notice.  Inversely, the `remove_from_bar` method is destroying the same `bar_item` that we find by using the current `@ingredient`, then destroying it.  The message passed is now "Removed from bar" and we are rednering the same action as a result.

{% highlight javascript %}
$('#<%= @ingredient.id %>').html("<%= j (render 'add_or_remove', ingredient: @ingredient) %>");
UnobtrusiveFlash.flashOptions['timeout'] = 2000;
UnobtrusiveFlash.showFlashMessage('<%= message %>', {type: 'success'});
{% endhighlight %}

First, I used the `ingredient.id` id attribute that was reference in the `_add_or_remove.html.erb` partial to target the correct button.  From here, that same partial is being rendered and will set change the button depending on whether or not the user added or removed the ingredient. 
From here, things got a little more interesting.  I spent a good deal of time figuring out how to get a flash notice to pop up as a result of an ajax response.  Apparently this doesn't just happen like it does with the standard html response and refresh that was occuring before. I played around with this for a bit, but it seemed to involved complicated custom method calls in `application.rb`.  I thankfully came across a the great [unobtrusive_flash](https://github.com/leonid-shevtsov/unobtrusive_flash).  It included JavaScript helper methods that allow for the timing and and display of a flash message.  Notice that my message variable from the controller is now used as the argument for `showFlashMessage`.

Yay! I got the buttons working and a flash notice was now popping up to inform the user.  The problem now was that the ingredients index view is longer and when a user scrolls down, they won't necessarily see the flash notice that appears right below the navbar. To correct this, I targeted the container holding the noticed in css and gave it the property `position:fixed`.  Unfortunately this was breaking views that weren't as long, causing the flash notice to overlap with the elements in the main container.  It took various tries: placing the flash container in different parts of the html, attemtpting to change the class attribute in the JavaScript itself, just forgetting about flash notices altogether.  This is what I wound up with

{% highlight erb %}
# app/views/layouts/application.html.erb

<html>
  ...
  <body>
    ...
    <main role="main"> 
      <%= render 'layouts/messages' %> 
      <%= yield %> 
    </main> 
  </body> 
  ...
</html>

# app/views/layouts/_messages.html.erb

<% unless (params[:controller] == 'ingredients' and params[:action] == 'index') %>
  <div  class="unobtrusive-flash-container"></div>
<% end %>

# app/views/ingredients/index.html.erb

<div  class="unobtrusive-flash-container ingredient-index-flash"></div>

{% highlight css %}
// app/assets/stylesheets/ingredients.scss

.ingredient-index-flash {
  position: fixed;
  width: 100%;
}
{% endhighlight %}
