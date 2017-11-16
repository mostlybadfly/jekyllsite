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

The views were the more complex part of this. It took a few tries and unfortunately most of the examples I was able to find were for on foreign key relationship, but I'm working with two. Let's take a look a the `RecipeCreate` view which is used when a user wants to add a completely new recipe: 

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

{% endhighlight %}
