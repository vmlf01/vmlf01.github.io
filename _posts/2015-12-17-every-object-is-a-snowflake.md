---
layout: post
title: "Every object is a snowflake"
description:
headline:
modified: 2015-12-13 15:13:04 +0000
category: [js, code]
tags: [js, code, OOP]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

Messing around with a very simple game project where I have a set of lights that can be toggled and the team that gets them all on or off wins.

So I started implementing a ```GameLights``` manager class and a ```Lights``` class to model each individual light. I added a ```getLights()``` method to ```GameLights``` so I could retrieve the current game state and then it hit me! If I return the collection of lights that I'm using inside the ```GameLights``` object, anyone can change the state of the lights and cheat.

No problem, lets make the ```Lights``` class properties only changeable from the ```GameLights``` class. Easier said than done!

Google search turned me to [Implementing Private and Protected Members in JavaScript](http://philipwalton.com/articles/implementing-private-and-protected-members-in-javascript/), but that only raised additional problems: basically if you want a real private variable, you can't access it from prototype methods. Ok, so don't use prototypes, but that causes another issue: memory usage!

This one guy shows the results of a simple test on his article [Four ways to deal with private members in JavaScript](http://eclipsesource.com/blogs/2013/07/05/private-members-in-javascript/) and the memory difference between having the methods declared in the prototype and having them declared in the object is 8 fold!!

Then I turned to Crockford's article on [Private Members in JavaScript](http://javascript.crockford.com/private.html). If anyone know how to do it, it's this guy. No luck there either. There seems to be no simple way to have real private properties with access from prototype.

The first article does show two libraries that use a static Hashmap to keep track of every object intance property values, but that is not simple.

If Uncle Bob thinks [encapsulation is broken in OOP](https://youtu.be/QHnLmvDxGTY?t=38m5s), what would he say about this in JavaScript?
