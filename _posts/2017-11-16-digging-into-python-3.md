---
layout: post
title: Digging into Python - ChiPy Mentorship Part 3
comments: true
---

<p align="center">
  <img src="https://i.imgur.com/q1XIBbl.jpg">
</p>

In my last update, I went through setting up the models for Recetario. After discussing with Steve, we decided to move on to making the views work the way we want them. This involved primarily creating forms to save recipes when a user logs in.  We took an iterative approach to this by first making individual forms for `Recipe`, `Ingredient`, and `Instruction`, then figuring out how to combine them into one big form.  I appreciated this approach because I got to see more how individual models are working together to achieve the end result. This step was a bit challenging and required a lot of scouring documentation to get it just right.  I am really proud of being able to work through this and get to the core of what my app does which is saving a recipe. Below is the end result.

### Forms

Let's look at some code. In my last post I shared the models that I created, below I am creating form objects using Django's generic form views.  These are very helpful in that Django will generate fields based on the attributes for my objects:

{% highlight python %}
#recetario/recipes/forms.py

from django.forms import ModelForm, inlineformset_factory

from .models import Recipe, Ingredient, Instruction

class RecipeForm(ModelForm):

    class Meta:
        model = Recipe
        fields = ('title', 'cuisine', 'cooking_time', 'servings')

class IngredientForm(ModelForm):

    class Meta:
        model = Ingredient
        fields = ('quantity', 'measurement', 'name')

class InstructionForm(ModelForm):

    class Meta:
        model = Instruction
        fields = ('ordinal', 'instruction_text')

IngredientFormSet = inlineformset_factory(Recipe, Ingredient, form = IngredientForm, extra=1)
InstructionFormSet = inlineformset_factory(Recipe, Instruction, form = InstructionForm, extra=1)
{% endhighlight %}

In the above you can see that I'm importing the `ModelForm` module as well as `inlineformset_factory`. These will help me create the forms in my view and template. First I utilize `ModelForm` to create form objects, in this case `RecipeForm`, `IngredientForm`, `InstructionForm`. The model gets referenced along with the fields that I want to expose.  Later on in my template, these fields will show up as individual input boxes.  At the very end of my code I'm creating inline formsets which are like a form object but specifically use to associate models when there is a foreign key relationship. I created a formset for both `Ingredient` and `Instruction` in this case since they both have a relationship to `Recipe`.

### Views

The views were the more complex part of this. It took a few tries and unfortunately most of the examples I was able to find were for on foreign key relationship, but I'm working with two. Let's take a look at the `RecipeCreate` view which is used when a user wants to add a completely new recipe: 

{% highlight python %}
#recetario/recipe/views.py

from .forms import RecipeForm, IngredientForm, InstructionForm, IngredientFormSet, InstructionFormSet
from django.views.generic.edit import CreateView, UpdateView
from django.urls import reverse_lazy
from django.db import transaction

from .models import Recipe, Ingredient, Instruction

class RecipeCreate(CreateView):
    model = Recipe
    fields = ['title', 'cuisine', 'cooking_time', 'servings']

    def get_success_url(self):
        return reverse_lazy('detail', args = (self.object.id,))

    def get_context_data(self, **kwargs):
        data = super(RecipeCreate, self).get_context_data(**kwargs)
        if self.request.POST:
            data['ingredients'] = IngredientFormSet(self.request.POST, prefix='ingredients')
            data['instructions'] = InstructionFormSet(self.request.POST, prefix='instructions')
        else:
            data['ingredients'] = IngredientFormSet()
            data['instructions'] = InstructionFormSet()
        return data

    def form_valid(self, form):
        form.instance.created_by = self.request.user
        context = self.get_context_data()
        ingredients = context['ingredients']
        instructions = context['instructions']
        with transaction.atomic():
            if form.is_valid() and ingredients.is_valid() and instructions.is_valid():
                self.object = form.save()
                ingredients.instance = self.object
                instructions.instance = self.object
                ingredients.save()
                instructions.save()
        return super(RecipeCreate, self).form_valid(form)
{% endhighlight %}

To begin, I am using Django generic view, `CreateView` which is used specifically for making a form to create an object. In this case it is for creating a `Recipe` object, the rest of the logic is so that I can associate `Ingredient` and `Instruction` data to it. I referenced the model (`Recipe`) as well as the fields I want to expose below it. `CreateView` has methods that can be overridden. `get_success_url` is used so that I can give the view somewhere to go once the object is created. I am using `reverse_lazy` here to get the recipe detail URL after the object is created. It is my understanding this is used when the url details aren't immediately known, as with a new object. Within `get_context_data` this is where I am getting information from the two formsets I created. The `Ingredient` and `Instruction` objects along with their attributes are assigned to data here for use in saving the form. With `form_valid`, I am utilizing this data in order to get my list of ingredients and instructions. Here I use `transaction.atomic()` to set up the objects to be saved, in this case multiple ingredients and instructions, so that they can be committed to the database in a single transaction.

To utilize this view, I needed to reference it in `urls.py` by utilizing `as_view` on the view class and passing the template that I want to use.:

{% highlight python %}
#recetario/recipes/urls.py

from django.conf.urls import url
from .views import RecipeCreate

urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^add/$', RecipeCreate.as_view(template_name="recipes/recipe_edit.html"), name='add'),
    url(r'^(?P<recipe_id>[0-9]+)/$', views.detail, name='detail'),
]
{% endhighlight %}

Finally, in the template, I loop through the forms so that it will render the fields for the models. I have condensed the below to include only the fields for ingredients:

{% highlight html %}
#recipes/templates/edit_recipe.html
{{ "{% this " }}%}
{{ "{% extends 'recipes/base.html' " }}%}
{{ "{% load static " }}%}

{{ "{% block content " }}%}
<div class="col-md-4">
  <form action="" method="post">{{ "{% csrf_token " }}%}
    {{ form.as_p }}

    <table class="table">
        {{ ingredients.management_form }}

        {{ "{% for form in ingredients.forms " }}%}
          <tr class="ingredient_formset_row">
            {{ "{% for field in form.visible_fields " }}%}
              <td>
                {{ "{% if forloop.first" }}%}
                  {{ "{% for hidden in form.hidden_fields" }}%}
                    {{ hidden }}
                  {{ "{% endfor" }}%}
                {{ "{% endif" }}%}
                {{ field.errors.as_ul }}
                {{ field }}
              </td>
            {{ "{% endfor " }}%}
          </tr>
        {{ "{% endfor" }}%}
    </table>
    ...
        <input type="submit" value="Save"/> <a href="{{ "{% url 'index'" }}%}">back to the list</a>
  </form>
</div>
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
<script src="{{ "{% static 'formset/jquery.formset.js'" }}%}"></script>
<script type="text/javascript">
  $('.ingredient_formset_row').formset({
      addText: 'add ingredient',
      deleteText: 'remove',
      prefix: '{{ ingredients.prefix }}',
      formCssClass: 'ingredients-formset'
  });
</script>
{{ "{% endblock" }}%}
{{ "{% endhighlight" }}%}

There are various things going on here. The main part of this is that `form.as_p` will render the main `Recipe` form. Further down I am looping through `for form in ingredients.forms` in order to get my fields for an `Ingredient` to render. At the very bottom, you can see some JavaScript. I utilized the [django-dynamic-formset](https://github.com/elo80ka/django-dynamic-formset) JQuery plugin in order to dynamically add more fields for adding additional ingredients. Most important to note about this is the `ingredient_formset_row` classname I used on each `tr` that will allow the rows to be grouped up and passed to the `RecipeCreate` view for saving the transaction.

Now I can serve up this heaping bowl of code soup in order to create tons of recipes for my app! It took some work and head scratching but I'm really happy with how this is turning out.  Taking a break from all this back end work to move on to styling my templates so that they are a bit more user friendly. Stay tuned for the next update.
