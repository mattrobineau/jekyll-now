---
layout: post
title: Lessons learned - Updating .Net Core 1.1 to 2.0
---

.Net Core 2.0 has been [released](http://msft.social/uStZa6). To celebrate the release, I decided to update the framework of my .Net Core 1.1 project over to 2.0. I'm going to go over some of the issues I faced with the update, but first

# The .Net Core Highlights #

## Targeting has become a little easier ##

You can target linux by using the `linux-x64` and `linux-arm` runtimes rather than specifying the distrobution (eg: `debian8-x64`)

## .Net Core 2.0 Implements .Net Standard 2.0 ##

There will now be more of the API available to end users. Finally [the return of](https://stackoverflow.com/questions/42732207/where-is-the-equivalent-of-httputility-javascriptstringencode-in-net-core-1-1) the `JavaScriptStringEncoder` (now `JavaScriptEncoder`)
.Net Core 2.0 does also provider more APIs than is available in the .Net Standard 2.0. The dotnet team have compiled and provided a nice [diff](https://github.com/dotnet/standard/blob/master/docs/comparisons/netstandard2.0_vs_netcoreapp2.0/README.md).

## DbContext Pooling with Entity Framework Core 2.0 ##

EF will now allow you to create a pool of connections that your `dbContext` will use instead of recreating a new connection each time.

## Like query operator ##

`EF.Functions.Life(c.Name, "a%")` will use the SQL `LIKE` operator in queries.

# Lessons learned from updating to Core 2.0

## Branch! ##

I made a mistake of not creating a branch before starting to update packages and projects to .Net Core 2.0. I ended up with issues updating the packages and decided I needed a branch.

## Not all of the nuget packages you use will support .Net Core 2.0 ##

I found out (and regretted not creating a branch) that Npgsql does not currently support .Net Core 2.0. Their preview nuget package will install, however there is a runtime error due to missing implementations. Watch [this github thread](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL/issues/171) if you require Npgsql in your application. Hopefully it will be reopened with current comments.

## Set your target framework ##

In all of the projects, set your target framework to `<TargetFramework>netcoreapp2.0</TargetFramework>` and remove or update the `RuntimeFrameworkVersion`. The .Net Core 2.0 nuget packages will not update if these are not changed.

## Breaking changes ##

.Net Core 2.0 has deprecated the use of `IApplicationBuilder.UseCookieAuthentication` which will give you a build error. It has been replaced with `Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication`. You can find the relevant information in (this github ticket](https://github.com/aspnet/Security/issues/1310). 

I haven't run into other .Net Core related breaking changes, but I would not be surprised that there are more of them. Luckily it is easy to identify as it will show up in your build's error list.

## Updating nuget packages ##

This was probably the most painful part since many nuget packages reference one another, updating the packages might fail when package A references package B. It can be a nightmare if you have a lot of nuget packages. Most of the updates to the nuget packages consisted of failing to update a group of packages until 1 package worked (usually the base, but not always) and then updating the rest. On the main web project, I manually removed the packages and re-add each one using the higher version. This seemed the easiest and simplest solution. Especially when you don't know that package X supports .Net Core 2.0 (eg: NLog doesn't support 2.0 yet).

# In the end... #

Updating to 2.0 wasn't very painful or long. The longest part will be waiting to for the nuget packages my project depends on to support .Net Core 2.0. Until then, the best course of action is to continue implementing features/improvements on .Net Core 1.1 and watching out for when the packages are supported.

Sources:
[Announcing .NET Core 2.0](https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-core-2-0/)
[Announcing ASP.NET Core 2.0](https://blogs.msdn.microsoft.com/webdev/2017/08/14/announcing-asp-net-core-2-0/)
[.NET Standard 2.0 vs .NET Core 2.0](https://github.com/dotnet/standard/blob/master/docs/comparisons/netstandard2.0_vs_netcoreapp2.0/README.md)
[Announcing Entity Framework Core 2.0](https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-entity-framework-core-2-0/)
[[Draft] Auth 2.0 Migration announcement](https://github.com/aspnet/Security/issues/1310)