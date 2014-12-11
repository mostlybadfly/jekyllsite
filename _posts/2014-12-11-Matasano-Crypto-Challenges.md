---
layout: post
title: Matasano Crypto Challenges
comments: true
---

One of the most interesting (and definitely challenging) methods I have used to learn Ruby has been to work through Matasano's set of cryptography challenges (http://cryptopals.com). To me it is a practical way at looking how bits of information get pushed around and manipulated.  I have done various programming type challenges such as Project Euler, and CodeWars but this seems to be the most applicable to real situations, maybe because it quite literally is. I have so far done the first 4 challenges of the 1st set.

What I really enjoyed so far was creating a way to rank letter frequency in a string output.  The idea is to find the string that is most representative of English.  This is done via a regex to single out those strings that contain the highest frequency of commonly used english letters. This is from [Set 1: Challenge 3](http://cryptopals.com/sets/1/challenges/3/):

{% highlight ruby %}
def self.letter_freq(string)
    count = {}
    common = /[etaoins]/

    string.each_char do |ltr|
      if ltr.match common
        count[ltr] = 0 unless count.include?(ltr)
        count[ltr] += 1
      else
        count[ltr] = 0
      end
    end
    count.values.inject(:+)
  end
{% endhighlight %}

I share this bit because it was most fun for me to write. I just liked the flow of how it is searching through each character to make sure it falls within the regex I set. I don't think it is the most complex code but I think it is a really good representation of how Ruby flows in an elegant way to ge to the information you need.  I'll update with more tidbits of fun or interesting ideas I come across as I continue to work through these.
