---
layout: post
title: "Collections are fun, even when testing"
description:
headline:
modified: 2015-12-13 15:13:04 +0000
category: [tdd, nodejs]
tags: [tdd, node.js, chai]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

I'm doing a little hobby project to test out the [ESP8266](http://espressif.com/en/products/esp8266/) chip and [Espruino](https://github.com/espruino/Espruino) and decided to use TDD to get the hang of it. In one of the tests, I needed to check if all values of an array where equal to a value. Before starting to implement some loop logic, why not Google a bit!

Came across this [**chai**](http://chaijs.com/) plug-in named [**chai-things**](http://chaijs.com/plugins/chai-things) that does exactly what I wanted and much more.

To start using it I just needed to do:

```js
var chai = require("chai");
chai.use(require('chai-things'));
```

and then use it on an array like:

```js
status.should.all.equal(false);
```

Just a small note, I first tried using it like this:

```js
status.should.all.be.false;
```

but it didn't work.

There are a lot more things you can do with it, so if you are using chai, check out the docs and give it a try!
