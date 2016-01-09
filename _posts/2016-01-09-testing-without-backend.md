---
layout: post
title: "Testing without a backend"
description:
headline:
modified: 2016-01-09 19:57:24 +0000
category: [angular, js, webdevelopment]
tags: [angularjs, js]
imagefeature: fakeit.png
mathjax:
chart:
comments: true
featured: false
---

[AngularJS](https://angularjs.org) makes it very easy to mock out a server API, giving you control over the returned data without the need for a network connection.

Thanks to the $httpBackend service in [ngMockE2E](https://docs.angularjs.org/api/ngMockE2E/service/$httpBackend) you can setup your own rules for intercepting the requests made with $http and choosing what to do with them. This is really usefull in unit testing and end-to-end testing, but today I used it as a development helper because I didn't have access to an API server and I didn't want to run it locally.

It is really easy to setup, you just install it with ```bower install angular-mocks```, add the ```ngMockE2E``` module dependency to your module and then configure $httpBackend.

```js
(function () {
    'use strict';

    angular
        .module('goOntimeClient')
        .run(setupFakeHttpBackend);

    function setupFakeHttpBackend($httpBackend, endpoints) {
        if (!endpoints.isLiveApi) {

            var webapi = endpoints.webapi;

            $httpBackend.whenPOST(webapi + "/account/login")
                .respond({
                    User: {
                        Name: 'Roger Rabbit',
                        UserName: 'roger',
                        Role: 'user'
                    }
                }, 1000);

            $httpBackend.whenPOST(webapi + "/account/logoff")
                .respond({}, 500);

            // For everything else, don't mock
            $httpBackend.whenGET(/^\w+.*/).passThrough();
            $httpBackend.whenPOST(/^\w+.*/).passThrough();
        }
    }

})();
```

So in this sample, I first setup two responses for faking the user login API, and then configure everything else to pass-through because all the partial HTML requests are loaded with $http, so they also get intercepted. You can use a string or a regular expression to define your URL matches.

But what is that second parameter you see on the ```.respond()``` call?

That is courtesy of [angular-mocke2e-maydelay](https://github.com/popduke/angular-mocke2e-maydelay), a very useful module to provide a simple way to specify a delay time for each individual response. You install it, add it to the module dependencies and then just use that second parameter of the ```.respond()``` call to specify how many milliseconds you want to delay the response. Couldn't be any simpler.
