---
layout: post
title: Binding DbContext in Simple Injector
---

It is possible to find all the information to successfully cross-wire your dbcontext into simple injector in the [documentation](http://simpleinjector.readthedocs.io/en/latest/aspnetintegration.html). The documentation doesn't explicitly state the how to cross-wire/bind a dbcontext. I will attempt to clear this up in this post.

## Cross-wire the DbContext ##
``` c#
public void ConfigureServices(IServiceCollection services)
{
    // Things

    services.AddDbContext<Context>(options => options.UseNpgsql(Configuration.GetConnectionString("NpgsqlConnection"), b => b.MigrationsAssembly("Stories")), ServiceLifetime.Scoped);

    // More things
    services.UseSimpleInjectorAspNetRequestScoping(container);
}
```

The `ServiceLifetime.Scoped` will insure that the context will have a scoped lifetime instead of a transient life time.

``` c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    //things 
    container.Options.DefaultScopedLifestyle = new AsyncScopedLifestyle();
    // things
}
```

`AsyncScopedLifestyle` will scope the container to the `async` call and all subsequent/child `Task`s will run using this container.

This means that when you use a controller with `async Task<IActionResult>` the container will be scoped to this call. If you make a call to 2 different controllers, a new container will be created for each.

## Binding the DbContext ##

When it comes to a DbContext, we don't necessarily want to have the DbContext instance created for every `async` call. If it is bound this way, you will run into deadlock issues. For example, if you have an action that updated the number of items in a cart (int32), and double click the "Do It" button, it will create 2 different dbContexts and attempt to update the same column and row. If done quickly enough, the row will be locked by the database as it attempts to update, and one of the two calls will not be able to update.

``` c#
container.Register<IDbContext>(() => app.GetRequiredRequestService<DbContext>(), Lifestyle.Scoped);
```

By using `GetRequiredRequestService`, Simple Injector will bind the DbContext within the current `request`, solving the deadlock issues.