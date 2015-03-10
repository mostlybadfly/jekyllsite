---
layout: post
title: Random Rubyism
comments: true
---

  While working o the [Matasano Crypto Challenges](http://cryptopals.com/), I came across a fun little bit of Ruby.  This was completely random and I didn't even know I could do this, so I wanted to share.  I am always amazed by really elegant and just neat ways you can express things in Ruby, and I was a little excited that this was the key I needed to complete one of the challenges.

  The challenge was to encrypt a some text by utilizing a repeating-key XOR. The text to encrypt was as follows:
  {% highlight text %}
  **_Burning 'em, if you ain't quick and nimble_**
   
  **_I go crazy when I hear a cymbal_**
  {% endhighlight %}
  The key to encrypt this is 'ICE'.

  The instructions (which can be found [here](http://cryptopals.com/sets/1/challenges/5/)) specify that the first byte of the key will be applied to the first byte of text, the second byte to the next, and the third byte to the next byte of text after that until the key repeats.  In previous challenges when I had to implement XOR, I split both the key and the text and zipped prior to XOR'ing each pair of corresponding bytes.  In this case I have less bytes in the key, which would result in nils after using `zip`:

  {% highlight ruby %}
  text = "burning".split('')
  key = "ice".split('')
  text.zip(key)
  # => [["b", "i"], ["u", "c"], ["r", "e"], ["n", nil], ["i", nil], ["n", nil], ["g", nil]]
  {% endhighlight %}

  I wracked my brain trying to figure out a good solution to this.  I tried passing blocks or somehow nesting two `map` methods, but nothing was working.  While I was reviewing the Ruby documentation in search of a solution I came across `cycle` which is described like this:
  ```
  *Calls the given block for each element n times or forever if nil is given.*
  
  *If no block is given, an Enumerator is returned instead.*
  ```
  I found this interesting and decided to see if I can call `cycle` from within `zip` under the assumption that is would stop when the first array stops:

  {% highlight ruby %}
  text = "burning".split('')
  key = "ice".split('')
  text.zip(key.cycle)
  # => [["b", "i"], ["u", "c"], ["r", "e"], ["n", "i"], ["i", "c"], ["n", "e"], ["g", "i"]]
  {% endhighlight %}

  Success! The `key` array continues to `cycle` during the `zip` to match up the remaining characters. With this accomplished, I was able to finish implementing repeating-key XOR.
 
  I am continued to be amazed at how much fun finding and working with bits of Ruby like this can be.  The fact that an hour or so of trying various things ended in a nice little method call is funny but also a wonderful thing.
