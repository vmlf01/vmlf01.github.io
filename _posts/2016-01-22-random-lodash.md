---
layout: post
title: "Random Lodash Pearls"
description:
headline:
modified: 2016-01-22 15:02:23 +0000
category: [js, code]
tags: [js, code]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

Collection operations never cease to amaze me! From SQL to .Net's Linq to now JavaScript's [Lodash](https://lodash.com/) provide some awesome tools to manipulate them.

Just came across these 2 simple examples from the Lodash docs related to random operations:

## _.random

Generates a random number.

```js
_.random(0, 5);
// an integer between 0 and 5

_.random(5);
// also an integer between 0 and 5

_.random(1.2, 5.2);
// a floating-point number between 1.2 and 5.2
```

## _.sample

Returns a random element from a collection.

```js
_.sample([1, 2, 3, 4]);
// returns a random element, for instance: 2
```

## _.sampleSize

Returns a random element from a collection.

```js
_.sampleSize([1, 2, 3, 4], 2);
// returns 2 random elements, for instance: [3, 1]
```

These examples are trivial, and yet they make life so much easier when we need random things.
