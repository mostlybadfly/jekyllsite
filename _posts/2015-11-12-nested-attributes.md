---
layout: post
title: Nested Attributes and a `has_many :through` Relationship.
comments: true
---

{% highlight ruby %}
class Recipe < ActiveRecord::Base
  belongs_to :user

  has_many :recipe_items
  has_many :ingredients, through: :recipe_items
  accepts_nested_attributes_for :recipe_items

  validates :title, presence: true
  validates :instructions, presence: true
end
{% endhighlight %}

{% highlight ruby %}
class Ingredient < ActiveRecord::Base
  has_many :recipe_items
  has_many :recipes, through: :recipe_items
end
{% endhighlight %}

{% highlight ruby %}
class RecipeItem < ActiveRecord::Base
  belongs_to :recipe
  belongs_to :ingredient
  accepts_nested_attributes_for :ingredient
end
{% endhighlight %}
