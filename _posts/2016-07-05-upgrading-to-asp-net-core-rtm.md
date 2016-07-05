---
layout: post
title: "Upgrading to ASP.NET Core RTM"
description: An overview of the not so obvious issues I faced when upgrading an application to .Net Core 1.0
headline:
modified: 2016-07-05 11:47:06 +0100
category: [csharp, webdevelopment]
tags: [c#, code]
imagefeature: asp_net_core.jpg
mathjax:
chart:
comments: true
featured: false
---

With the release of ASP.NET Core 1.0, I went about trying to upgrade a project I worked on to the latests bits.
It is a small Web API project that used ASP.NET 5 RC2 running on DNX in Ubuntu and is currently deployed in production using that environment.

The first step is getting the .NET Core installed, which is easy enough by following the instructions on the [.Net Core site](https://www.microsoft.com/net/core#ubuntu).

If you are already on RC2 with a dotnet based project, it's easier to migrate, there are not that many changes to make.
But my project is still DNX based, so there is quite a lot to change.

Fortunately, there is also some pretty good documentation about all the things that have changes from RC1 to RC2 to 1.0 RTM:

- [Migrating from ASP.NET 5 RC1 to ASP.NET Core 1.0](https://docs.asp.net/en/latest/migration/rc1-to-rtm.html)
- [Migrating from ASP.NET Core RC2 to ASP.NET Core 1.0](https://docs.asp.net/en/latest/migration/rc2-to-rtm.html)
- [Getting Ready for ASP.NET Core RC2](https://wildermuth.com/2016/05/13/Getting-Ready-for-ASP-NET-Core-RC2)
- [Converting an ASP.NET Core RC1 Project to RC2](https://wildermuth.com/2016/05/17/Converting-an-ASP-NET-Core-RC1-Project-to-RC2)
- [Converting ASP.NET Core 1.0 RC2 to RTM Bits](https://wildermuth.com/2016/06/27/Converting-ASP-NET-Core-1-0-RC2-to-RTM-Bits)

Most of the changes revolve around namespace changes and the project.json structure. Just a matter of going over them until you can build the project.

Some of the changes are not so obvious, because they don't generate compile time errors. Let me point out 3 of them I ran into.

### Using strongly-typed configuration settings

If you follow Microsoft examples on this one, it's pretty simple to inject your configurations wherever you need them.
Rick Strahl [has a post on it](https://weblog.west-wind.com/posts/2016/May/23/Strongly-Typed-Configuration-Settings-in-ASPNET-Core#PluggableConfiguration) which goes into detail on how to set it up.
The problem is it is injected as an instance of the generic IOptions<T> type and I don't like that.

Before, we we're using IConfiguration.Get() for getting and registering an instance of the configuration object itself in IoC services, like this:

```csharp
services.AddScoped<OdooConnectionInfo>(service =>
{
    return Configuration.GetSection("OdooConnection").Get<OdooConnectionInfo>();
});
```
That method is gone, so now, we need to use the extension method IConfiguration.Bind() to map a configuration section to our own created object instance.

```csharp
services.AddScoped<OdooConnectionInfo>(service =>
{
    var odooSettings = new OdooConnectionInfo();
    Configuration.GetSection("OdooConnection").Bind(odooSettings);
    return odooSettings;
});
```

There is an [open issue](https://github.com/aspnet/Configuration/issues/467) for this one to add support for a similar API as before, so you don't have to create an instance of the configuration object you want and then bind it.


### Entity Framework Sqlite provider connection string
I was using a relative path in the Sqlite connection string pointing to the App_Data/ folder, which was being resolved using the project source root folder where I started the application.

```json
"ConnectionString": "Data Source=./App_Data/AppDB.dev.db"
```

When I was finally able to start the application after the migration, I got this exception:

```sh
Microsoft.Data.Sqlite.SqliteException: SQLite Error 14: 'unable to open database file'.
```

Spent a lot of time trying to figure out what that meant, until I came across this [issue](https://github.com/aspnet/Microsoft.Data.Sqlite/issues/55) on github.
A lot of discussion about what a relative path should be relative to. It seems to be relative to the compiled executable location, which makes sense, because now we have a real executable compiled binary, whereas before if was just compiled in-memory.
So I ended up changing it to:

```json
"ConnectionString": "Data Source=../../../App_Data/AppDB.dev.db"
```

This will point to the original App_Data/ folder I have in the project like before, by going up from the bin/Debug/netcoreapp1.0/ folder dotnet compiles the application into.

### Json serialization settings

When everything seemed to be working fine again, I fired up the client application and started to get some weird errors. Upon closer inspection, it turned out the server responses weren't exactly equal to what we got before.
The default JSON property names serialization convention changed to camelCase and before it was not making any conversion, so it just used the .Net class property names.
Looks like finally .Net is aligned with the rest of the JSON world, which is good, but the existing client is already using the old names, so it doesn't exactly make sense to go and change it now.

We can get the old default behaviour back by [setting the JSON options on Startup](https://weblog.west-wind.com/posts/2016/Jun/27/Upgrading-to-ASPNET-Core-RTM-from-RC2#TurningoffCamelCasing):

```csharp
services.AddMvc()
        .AddJsonOptions(opt =>
    {
        var resolver  = opt.SerializerSettings.ContractResolver;
        if (resolver != null)
        {
            var res = resolver as DefaultContractResolver;
            res.NamingStrategy = null;  // <<!-- this removes the camelcasing
        }
    });
```

And with that, I got most of the application running on .Net Core 1.0.

Now I just have to handle the Postgres connection and the EMail library we were using before.