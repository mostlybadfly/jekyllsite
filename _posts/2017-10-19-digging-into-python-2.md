---
layout: post
title: Digging into Python - ChiPy Mentorship Part 2
comments: true
---
<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9b/CasBombu-recetario.jpg/382px-CasBombu-recetario.jpg">
</p>

A few weeks in and things are going great with the mentorship.  I have met with Steve and had a couple of informative hangouts to set up and go through how my Django application will work. I have settled on a name: Recetario. Directly translated it can mean “cookbook”, but specifically a collection of recipes for a specific family or household. In other usages it can mean medical handbook or list of prescriptions.  I felt like this was fitting for the intention of my app: a place for avid home cooks to jot down and store recipes they create or modify.  A collection of their own personal recipes, procedures, and bits of inspiration for their creation.  Apart from a listing of ingredients and instructions, my hope is that a cook will have the option to include a blog-like post with the recipe describing their motivation and thought process behind what they made. They will also be able to tag the type of cuisine if one (or multiple) apply. I’m looking forward to thinking of more features to add. As someone who plans to use this heavily, I’m pretty excited!

### Mentor Meetings

Steve has been a great help these past several weeks.  It has been a great experience getting all these bits of knowledge and guidance from someone who has been working in the industry for a while.  There are various aspects of developing a web application that I know to do but don’t know necessarily WHY I’m doing them. It has been an eye opening experience to get a breakdown of how data structures work or how to break down the development steps into smaller more consumable pieces. I have noticed feeling less overwhelmed by the different things needed to create a web application and see know that it is totally manageable thanks to Steve’s insight.

### Project Progress

At this point, I have been utilizing the [Django Tutorial]( https://docs.djangoproject.com/en/1.11/intro/tutorial01/) as a guide for the beginnings of my application. We found that the first few sections provide a good outline for some of the views and models I needed to implement for Recetario. So far, I have models and URLs set up for my recipes.  What I like about Django is how easy it is to read the models and see exactly what they are doing:

{% highlight python %}
#recetario/recipes/models.py
from django.db import models
from django.contrib.auth.models import User

class Recipe(models.Model):
    title = models.CharField(max_length=200)
    cuisine = models.CharField(max_length=50)
    cooking_time = models.IntegerField(blank=True, null=True)
    servings = models.IntegerField(blank=True, null=True)
    created_by = models.ForeignKey(User, related_name='recipes')

class Ingredient(models.Model):
    quantity = models.IntegerField(blank=True, null=True)
    measurement = models.CharField(max_length=50)
    name = models.CharField(max_length=100)
    recipe = models.ForeignKey(Recipe, related_name='ingredients')

class Instruction(models.Model):
    ordinal = models.IntegerField(blank=True, null=True)
    instruction_text = models.TextField(null=True)
    recipe = models.ForeignKey(Recipe, related_name='instructions')
{% endhighlight %}

In the above you can see my three main {% highlight python %}Recipe{% endhighlight %} related models and the attributes for each. Setting up the relation between `Ingredient` or `Instruction` and `Recipe` felt pretty straightforward. I was somewhat surprised I was able to save a recipe with ingredients and instructions in the console right away. Getting to this point felt good as I began to see how all the other pieces will fall into place.

I am up to chapter 4 of the tutorial and currently working on user signup, login, and logout functionality. I’m really happy to say that as of today, a user is able to sign up and log into Recetario. They can’t create a great recipe yet, but that is my next step: creating recipe forms and having a saved recipe associate to the logged in user.  Creating the sign up was probably my biggest challenge so far, but an enjoyable one nevertheless.  I used a guide from [Simple Is Better Than Complex]( https://simpleisbetterthancomplex.com/) to help with this process. I found that they have a lot of useful tidbits for working with Django on their site, even though the Django docs are my primary source since they do a great job of including everything.  For my Login and Logout views, I used the Django built in authentication views. Conveniently, Django provides user authentication views, from there all I needed to do was create templates.  More info can be found [here]( https://docs.djangoproject.com/en/1.11/topics/auth/default/#module-django.contrib.auth.views)

### Next Steps
After I finish user authentication, I will be taking a trip to template town. I will be creating templates for recipe creation, recipe edit, recipe detail, recipes listing, and better user templates along with profile pages.  For styling I will be using bootstrap so apart from Python there will be a fair deal of HTML and CSS coming up. 
Eventually, when I have a good deal of the basic functionality set up, I want to into setting up tests and familiarize myself with TDD. Hopefully this will lay down the groundwork for added additional features such as the text posts on recipes that I mentioned previously. I've got a lot cooking right now and can’t wait to see how everything comes out!
