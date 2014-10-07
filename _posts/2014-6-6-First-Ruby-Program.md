---
layout: post
title: One of My First Programs
comments: true
---

When first started this, I decided to just start learning ruby and to get really good at programming. I really didn't know what that meant, it was just what I thought you had to do. one of the first things I did was a simple game collection program.

{% highlight ruby %}
require 'game.rb'

class GameApp
  def initialize
    puts "Welcome to your game library!"
    puts "What would you like to do?"
    puts "Type 'new', 'update', 'delete', or 'show'"
    puts "Type 'bye' to exit the system"
    @task = gets.chomp
    main_menu
  end
  
  def start_over
    puts 'what would you like to do next?'
    puts "'new', 'show', 'update', 'delete', 'bye' :"
    @task = gets.chomp
    main_menu
  end
  
  def main_menu
    
    case @task
    when 'new'
      puts 'What is the title of the game you want to add?'
        name = gets.chomp
      puts 'What is the year it was made?'
        year = gets.chomp
      puts 'For which system was it made?'
        system = gets.chomp
      puts "------------------------"
      
      newgame = Game.new(name, year: year, system: system)
      newgame.save
      start_over
    
    when 'show'
      s = Game.new(s)
      s.display
      start_over
      
    when 'update'
      t = Game.new(t)
      t.display
      puts "Select the ID of the game you want to modify:"
      id = gets.chomp
      
      puts 'What is the title of the game?'
        name = gets.chomp
      puts 'What is the year it was made?'
        year = gets.chomp
      puts 'For which system was it made?'
        system = gets.chomp
      puts "------------------------"
      
      editgame = Game.new(name, year: year, system: system, id: id)
      editgame.update
      start_over
      
      
    when 'delete'
      t = Game.new(t)
      t.display
      puts "Select the ID of the game you want to delete:"
      id = gets.chomp
      
      gamebye = Game.new("name", id: id)
      gamebye.delete
      start_over
      
    when 'bye'
      puts 'goodbye :)'
      
    else
      puts "did not recognize input, please choose 'new', 'update', 'delete', 'show', 'bye'"
      main_menu
      
    end
  end
end
{% endhighlight %}

This was what I would say was a first success for me in that I got a basic set of actions covered, and I was able to create a handler program to talk with the database.  The above requires 'game.rb', here is the link to this, suggestions are welcome: [gamelist](https://github.com/mostlybadfly/gamelist).
