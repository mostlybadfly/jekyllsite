---
layout: post
title: Lita and creating a recipe handler 
comments: true
---

For the past week, I've been playing around with Lita, a chat bot written in Ruby.  Lita works within different chat protocols, irc being one of them. It does this with the help of a plugin which helps to connect to the chat protocol of choice. There are also handler plugins that will do various tasks such as welcome a user, image search in chat, and a karma ranking for popular words.  I decided to get to work on writing my own handler.

It is actually very easy to begin using Lita. The author has very extensive documentation on this which can be found [here](http://docs.lita.io/). After installing and creating the new bot, I installed gems and configured them to work.  What the configuration file does for Lita is let it know how to utilize the plugins.  Here is an example of botfly, my current Lita bot.

{% highlight ruby %}
Lita.configure do |config|
  config.robot.name = "botfly"
  config.robot.adapter = :irc
  config.adapters.irc.server = "irc.freenode.net"
  config.adapters.irc.channels = ["#mostlybotfly"]
  config.adapters.irc.user = "botfly"
  config.adapters.irc.realname = "botfly"
  config.adapters.irc.password = ENV['BOTFLY_PASS']
  config.adapters.irc.cinch = lambda do |cinch_config|
    cinch_config.max_reconnect_delay = 123
  end
  config.redis[:host] = "127.0.0.1"
  config.redis[:port] = 6379
  config.handlers.karma.term_pattern = /[\[\]\p{Word}\._|\{\}]{2,}|[<:][^>:]+[>:]/
  config.handlers.karma.term_normalizer = lambda do |term|
    term.to_s.downcase.strip.sub(/[<:]([^>:]+)[>:]/, '\1')
  end
  config.handlers.recipe.api_key = ENV['F2F_KEY']
end
{% endhighlight %}

What the above shows is that my bot will be connecting to irc channel #mostlybotfly, login as "botfly" and authenticated with a password. Below this is the connection to the local redis host, special term_pattern for the karma handler, as well as setting the api for my recipe handler. Documentation for various handlers could be find online, but in general they involve sending a command to Lita and getting a response in return.  Below is an example of a handler for Lita that I had fun writing.

The lita-recipe handler will return a random top recipe from Food2Fork.  The idea is that if you are in a chat and looking for a random recipe idea, you can have it on demand.  Lita makes it easy to create handlers and can be quickly created using `lita handler NAME_OF_YOUR_HANDLER'.  For mine, I did `lita handler lita-recipe`. From there, Lita creates a handler ruby file to create the actions to be carried out.  I have provided an example of the lita-recipe code:

{% highlight ruby %}
require 'lita'
require 'net/http'
require 'uri'
require 'multi_json'

module Lita
  module Handlers
    class Recipe < Handler
      config :api_key, required: true
      route(/^recipe/, :recipe, command: true, help: { "recipe" => "random recipe" })

      def recipe(response)
        res = Net::HTTP.get_response(URI("http://food2fork.com/api/search?key=#{api_key}&sort=t"))
        top_list = MultiJson.load(res.body)
        response.reply(top_list["recipes"].shuffle.first["f2f_url"])
      end
      private

      def api_key
        Lita.config.handlers.recipe.api_key
      end

      Lita.register_handler(Recipe)
    end
  end
end
{% endhighlight %}

In the above, the handler is getting a response from Food2Fork's search, using an API key which would have to be provided. After running some rspec tests to make sure the handlers was replying as needed, I completed the gemspec and deployed my first gem to rubygems! It was a little too exciting to be able to include `gem "lita-recipe"` in my gemfile.  

Working on Lita so far has helped to reinforce my interest and workflow as a developer.  I hope to create another handler which will evaluate ruby code, much like irb for a chat room.
