---
layout: post
title: "Starting ASP.NET5 on Linux, Part 2"
description:
headline:
modified: 2015-12-21 10:00:00 +0000
category: [csharp, webdevelopment]
tags: [c#, code]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

Starting where we left off in [Part 1]({% post_url 2015-12-20-starting-asp-net-5-part1 %}), to get [OmniSharp](http://www.omnisharp.net/) integration (tasks, IntelliSense and more) in Visual Studio Code, I need to install Mono first.

Following the [official Mono instructions](http://www.mono-project.com/docs/getting-started/install/linux/#debian-ubuntu-and-derivatives), I ran:

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list
sudo apt-get update
sudo apt-get install mono-complete ca-certificates-mono
```

Looks like running dnx/dnu inside Visual Studio Code is tied to Gnome Terminal, so I had to install that as well:

```
sudo apt-get install gnome-terminal
```

Finally, I was able to run press *Ctrl + Shift + P* and then *dnx Restore Packages*. That opened a terminal window to execute the ```dnu restore``` command. Took a while, but came back ok.
If you get a message telling OmniSharp server is not running, you can do *Ctrl + Shift + P* and select *Restart OmniSharp* first.

After restoring the dotnet dependency packages, I was able to start the application by running ```dnx web``` on the command line. That starts up the Kestrel web server on http://localhost:5000 so if we open that in the browser, we get the default ASP.NET 5 web application.

Just to get everything else, I also did on the command line inside the project folder:

```
npm install

npm install bower --save-dev

./node_modules/.bin/bower install
```

Looks like the project relies on Bower being installed globally, but I don't like that, so I added it to the package.json under the dev dependencies. Anyway, it looks like that is only necessary if running the project in Development mode, otherwise it will load the js libs from ajax.aspnetcdn.com.

I tried to run the project with the *development* configuration using ```dnx web --ASPNET_ENV development``` but it doesn't seem to work out of the box, because the referenced bower components return a 404. It seems there is a task missing to copy all the necessary files into the *lib* folder under *wwwroot*. Let's consider that one a WIP.

Also, I seem to get IntelliSense in the C# code, but it doesn't auto-trigger when I type a '.', I have to explicitly press *Ctrl + Space*. Other than that, it seems to work ok.

Setting up ASP.NET development on Linux is still not a smooth experience, but at least now it is possible and it will surely get better.
