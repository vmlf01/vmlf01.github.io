---
layout: post
title: "Third time's a charm"
description:
headline:
modified: 2016-01-02 19:30:00 +0000
category: [csharp, webdevelopment]
tags: [c#, code]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

There is some truth in popular wisdom. After having to install VS Code on my desktop and re-install it on my laptop, I finally got it working great.

All the problems I had before were because I thought only CoreCLR was needed. Unfortunately, it seems that, for now, some of the .Net tooling is not working on CoreCLR yet, so we need Mono and the DNX Mono runtime installed, even if we are only going to create projects that use the DNX CoreCLR.

So this time around, I followed all of the steps on the [.Net Core DNX SDK for Linux](https://github.com/dotnet/coreclr/blob/master/Documentation/install/get-dotnetcore-dnx-linux.md) guide and I finally got IntelliSense working.

Also, when opening a new project, don't forget to select a project on the bottom right of the Code window after opening a .cs file.

![](/images/vscode-select-project.png)

If you have any other problems, be sure to check out the Developer Tools, which you can toggle in the Help menu.
