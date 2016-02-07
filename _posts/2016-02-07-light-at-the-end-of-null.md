---
layout: post
title: "Light at the end of null"
description:
headline:
modified: 2016-02-07 18:35:30 +0000
category: [csharp, webdevelopment]
tags: [c#, code]
imagefeature:
mathjax:
chart:
comments: true
featured: false
---

One of the coolest things in the new C# 6.0 is the **Null-Conditional** operator. It provides a way to largely simplify and avoid one of the most common things in c#: protecting against null.

So, instead of doing this:

```csharp
var a = null;
if (b != null)
{
    a = b.SomeValue;
}
```

you can now just do:

```csharp
var a = b?.someValue;
```

You can even chain several calls and it works using a short-circuit logic, same as with logical operators.
It will evalute the expression from left to right. If any part of it returns null, the whole expression will return null.

Suppose you need to parse some XML:

```csharp
XDocument doc = XDocument.Load("test.xml");
var field = doc.Descendants("field")
               .Where(x => (string) x.Attribute("name") == "my_cool_id")
               .FirstOrDefault();

if (field != null)
{
    string value = (string) field.Element("value");
    // Use value here
}
```

Using the new **Null-Conditional** operator, it would look like:

```csharp
XDocument doc = XDocument.Load("test.xml");
string value = doc.Descendants("field")
               .Where(x => (string) x.Attribute("name") == "my_cool_id")
               .FirstOrDefault()?.Element("value")?.ToString();

// Use value here
```

Pretty cool, huh?
