---
layout: post
title: "Starting ASP.NET5 on Linux, Part 1"
description:
headline:
modified: 2015-12-20 16:31:28 +0000
category: [c#, webdevelopment]
tags: [c#, code]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

Future is looking bright. Options, options and more options. After seeing a few presentations on the new ASP.NET platform, I have to say it looks a lot like Node.js - it certainly took a lot of inspiration from it!

First off, I went into [http://get.asp.net](http://get.asp.net), clicked *Install for Linux* and downloaded the DNX tar file. After extracting it, we get two executables:

- ```dnx```, a .net execution environment, which contains the code necessary to bootstrap and run an application
- ```dnu```, the .net development utitlity, which manages project dependency packages

But what about this ```dnvm``` I hear about? I already went throught some difficulties with running Node.js without using sudo all the time and solved it by using ```nvm```, so of course I wanted ```dnvm```. So I headed into the [detailed install instructions](https://docs.asp.net/en/latest/getting-started/installing-on-linux.html#install-the-net-version-manager-dnvm).

That's where problems start, but it is entirely my fault. I'm using [fish shell](http://fishshell.com/) instead of plain old boring bash, so once in a while, some shell scripts don't work. No worries, drop into bash shell for a while and re-run the command to install:

```
curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
```

and lo and behold, I got ```dnvm``` on the path to call upon wherever I want. Back into fish then. Or so I thought. This was going too easy. Simply sourcing the ```dnvm.sh``` script into my fish prompt doesn't work. Fortunately, took 2-3 minutes googling around to find this [repo](https://github.com/mgolisch/dnvm-fish) on github which has custom fish functions to make ```dnvm```, ```dnx``` and ```dnu``` work in the fish shell. Downloaded them and put them on my ```~/.config/fish/functions``` folder and everything is working again, without going into bash shell. Sweet!

Next up, let's install a runtime. So I did ```dnx --help``` and saw there was an *install* option, as expected. Second mistake. Why does dnvm install dnx for MONO by default??? What I really need to do to install the latest Core CLR is:

```
dnvm upgrade -r coreclr
```

Better read the instructions from now on. So next I installed libuv from source, as shown on the [install docs](https://docs.asp.net/en/latest/getting-started/installing-on-linux.html#install-libuv). Looks like we are all set for now. Let's move on to some coding.

I don't see any tutorial for Linux, so I jump into the [Your first ASP.NET 5 Application on a Mac](https://docs.asp.net/en/latest/tutorials/your-first-mac-aspnet.html) - I guess if we are not Windows, we must be OSX! Following along the tuturial, looks like I need to [install a yeoman generator for ASP.NET](https://docs.asp.net/en/latest/client-side/yeoman.html). I already have npm and yeoman, so no problem there, just business as usual.

Then I fire up Visual Studio Code (I already had that one installed) and try *dnx: Restore Packages* as instructed, just to find out I need something called OmniSharp server running and it is not. Going into *Help -> Toggle Developer Tools*, I get some error messages telling me I need Mono version >=4.0.1. There is no escaping it!

This is taking longer than I anticipated, so I'm going to split this into 2 parts. Don't miss out on [Part 2](starting-asp-net-5-part2), where I will be compiling and running the ASP.NET sample project.
