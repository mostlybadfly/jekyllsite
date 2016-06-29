---
layout: post
title: Calculator
comments: true
published: true
---

About a month ago, I joined a JavaScript study group.  While I have done some lessons and some simple JavaScript as well as JQuery for different projects, I felt I would benefit from additional practice.

Our first task as a group was to write a calculator in pure JavaScript.  This involved making various calls to the DOM and changing values in the textfield as number values changed.  I learned many things working on this and really made me excited to dig into more aspects of JavaScript.

There were some logical considerations for an app like a calculator.  First I had to make sure that when performing an operation that there is already a previous number, an operator and a new number to evaluate this against. Most of this logic can be found in my `performOp()` function which adds an operator to my `opStack` depending on how many numbers are already in `numStack`. Getting this logic down was challenging at first but I’m pleased with where I have taken it to this point.

```javascript
function performOp() {
  if (result.value!='' && numPressed) {
    if (numStack.length == 0) {
      numStack.push(result.value);
      opStack.push(this.innerHTML);
    } else {
      result.value = operation[opStack.pop()](Number(numStack.pop()), 
                                              Number(result.value));
      numStack.push(result.value);
      opStack.push(this.innerHTML);
    }
  }
  numPressed = false;
}
```

I was also happy with my solution to defining how the different operators will be carried out. I learned that you can make a hash of different functions that can be called individually.  So from the following:

```javascript
var operation = {
  '+': function(a,b) { return a + b},
  '-': function(a,b) { return a - b},
  '*': function(a,b) { return a * b},
  '/': function(a,b) { return a / b}
}
```

one would be able to call `operation[‘+’](1,2)` to get the result `3`.  This allows for the operator as a string type to be evaluated according to the definition in the hash.

The full repo for this can be found (here)[ https://github.com/mostlybadfly/calculo] . To play around with the calculator, visit (my calculator page)[http://www.mostlybadfly.com/calculator].

I had a lot a fun playing around with this and figuring out some more JavaScript. Some ways I’d like to improve this is to make the code a little more descriptive as to what is going on and perhaps having a function to update the textfield so it isn’t done as part of other functions.  Currently, I am looking at rebuilding this calculator anew using React as a framework.
