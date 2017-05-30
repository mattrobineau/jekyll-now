---
layout: post
title: Setting up appsettings.json for a console application on .Net Core 1.1.2
---

First post!

On my linux hosted ASP.Net Core app, I've realized that I would need a cronjob to run a database task on the server. Naturally, I decided to create a .Net Core console application and run it using `cron`. The first thing I wanted to do was mimic the ASP.Net project by taking advantage of the `ASPNETCORE_ENVIRONMENT` variable. I want to use an `appsettings.{Environment}.json` file so I don't accidentally commit production sensitive information.

I set up to find a way to do this in a console application. Microsoft's docs will tell you that you need two nuget packages. These are [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) and [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json). However, to use `IConfigurationRoot` you must have [Microsoft.Extensions.Configuration.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Abstractions). This is true for both a console application and an Asp.Net application.

The wiring for the console application is then simple.

```c#
string environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");

var builder = new ConfigurationBuilder()
                .SetBasePath(Path.Combine(AppContext.BaseDirectory))
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{environment}.json", optional: true);
```

And that's it.

I realize that the nuget packages were made by the aspnet team, but none of the dependencies require ASP.Net.
