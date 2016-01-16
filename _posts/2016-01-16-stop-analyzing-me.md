---
layout: post
title: "Stop Analyzing Me!"
description:
headline:
modified: 2016-01-16 16:53:11 +0000
category: [csharp, webdevelopment]
tags: [c#, code]
imagefeature: analyze-this.jpg
mathjax:
chart:
comments: true
featured: false
---

I installed [Microsoft Visual Studio 2015 Community](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx) a few weeks ago to debug a Web API services project created with the [CoreCLR runtime](https://github.com/dotnet/coreclr). As expected, the debugger worked great, but whenever I start the project, the new diagnostics view kicks in automatically and it slows down the debugging session.

I was checking the network requests with [Fiddler](http://www.telerik.com/fiddler) and there were also a large amount of other packets because of the tracing.

I spent some time looking around the solution and project menus and properties for an option to turn it off to no avail. Finally I found an article written by someone who shares the same pain with the steps to disable it: [Turn off the Diagnostics Tools window in Visual Studio 2015](http://blog.accentient.com/vs2015-diagnostics-tools-window/).

Turns out you need to go into the Visual Studio options and search for "Enable diagnostic tools while debugging" and turn it off. Why did someone though we needed this enabled by default? Life is good again!
