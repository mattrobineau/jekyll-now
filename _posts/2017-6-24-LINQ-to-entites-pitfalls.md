---
layout: post
title: LINQ to Entites Pitfalls
published: true
---

I've been using LINQ to Entites with Entity Framework for a long time. I wanted to share some of the pitfalls or costly operations which you should avoid. 

As developers, we always strive (at least I hope) to give our users the fastest web experience possible. We sometimes forget that the experience we get while testing new features we implemented does not reflect the reality of the production environment. I'm specifically talking about simultaneous connection and multiple users.

A LINQ to Entites query could take miliseconds on a developers computer while testing and it might mean minutes for the concurrent users of your application or feature.

Some of these pitfalls include: multiple database trips and getting more data than you need.

Let's say we have a simple store app and we want to tally up a user's cart.
{% highlight c# %}
public class Product
{
  public int Id { get; set; }
  public string Description { get; set; }
  public decimal Price { get; set;}
  public decime SalePrice { get; set; }
}

public class CartItem {
  public int Id { get; set; }
  public int ProductId { get; set; }
}
{% endhighlight %}

<h2> Multiple database trips </h2>

{% highlight c# %}
foreach(var item in CartItems) {
  var product = await dbContext.Products.Where(p => p.Id == item.ProductId).FirstOrDefaultAsync();
  
  // Add price to total; add to order items; .. etc.
}
{% endhighlight %}

In this example, we're getting the `Product` for the items in the cart so we can do "things" with it like get a total or add them to a list of orders. Every time we get a product, Entity Framework will make a call to the database. If we have 100 products, thats 100 calls to the database.

We can reduce the amount of calls to the database and therefor the cost of the operation down by modifying how we get the products we need to loop over.

{% highlight c# %}
var products = await dbContext.Products.Where(p => CartItems.Any(i => i.ProductId == p.Id)).ToListAsync();

foreach(var product in products) 
{
  // Do the thing
}
{% endhighlight %}

_There is usually never a reason to materialize entities inside of loops._

It is better to get all the objects you need to work on before hand and looping over those entites after. EF will get all the `Product`s in a single call rather than once every loop. Understandably, 1 database call is better than say, 100 database calls.

<h2> Excess data </h2>

We'll use the same app example, with a few changes.

{% highlight c# %}
public class Product
{
  public int Id { get; set; }
  public int BrandId { get; set; }
  public string Description { get; set; }
  public string Sku { get; set; }
  public decimal Price { get; set;}
  public decimal SalePrice { get; set; }
  public bool IsOnSale { get; set; }
  
  public virtual Brand { get; set; }
}

public class Brand
{
  public int Id { get; set; }
  public string Name { get; set; }
  
  public virtual List<Product> Products { get; set; }
}
{% endhighlight %}

There are a couple of way this can manifest itself. It is important to recognize when data is being loaded that is not necessary to the overall operation you are trying to accomplish. Getting more data than required may be a little harder to spot because it is highly contextual.

An example I see often is getting the `Id` of a navigation property

{% highlight c# %}
var productId = 1;
// Ugly
var product = await dbContext.Products.FirstOrDefaultAsync(p => p.Id == productId);
var brandId = product.Brand.Id;

// Bad
var product = await dbContext.Products.Include(p => p.Brand).FirstOrDefaultAsync(p => p.Id == productId);
var brandId = product.Brand.Id;

// Good
var product = await dbContext.Products.FirstOrDefaultAsync(p => p.Id == productId);
var brandId = product.BrandId;
{% endhighlight %}

I see this kind of behaviour often enough that I thought it was worth showing. Lets look at each example.

__Ugly__
It is rarely advantageous to lazy load anything! Don't do it.

__Bad__
We're no longer lazy loading and we have the `Brand`. Now we can get the list of products associated to this brand of something. The thing is that we don't need the `Brand` to get the Id. The product already has the `Id`!

__Good__
Like I explained in __Bad__, the product already has the Id. We don't need to fetch the `Brand` to get it!

Let's look at another example. Imagine we want to list all the Post title on the front page of our blog.
{% highlight c# %}
public class Post
{
  public int Id { get; set; }
  public string Content { get; set; }
  public string Title { get; set; }
  public DateTime CreatedDate { get; set; }
}
{% endhighlight %}

{% highlight c# %}
var posts = await dbContext.Posts.OrderBy(p => p.CreatedDate).ToListAsync();

foreach(var post in posts)
{
  // Do the thing
}
{% endhighlight %}

In this situation, we only need the title for each posts. The problem is that we get everything else from the post. This might some seem like a big deal at first, but the data for the list of posts will be sent over the _wire_ to our application. This no only includes the `Title` but also some (probably) non-trivial bytes of `Content` which isn't needed at all. This could be a difference getting 1kb of data per post vs 100 bytes of data per post.

{% highlight c# %}
var posts = await dbContext.Posts.OrderBy(p => p.CreatedDate).Select(p => new { p.Id, p.Title }).ToListAsync();

foreach(var post in posts)
{
  // Do the thing
}
{% endhighlight %}

Now we are only getting the `Id` and `Title` for each post. 

<h2>Review the generated SQL</h2>

Discovering the issues with your LINQ to entities queries can only come from examing how you would write the SQL to get the data you want against the generated SQL of entity framework. If you do this regularly, you will start to see what patterns are best for retrieving the data you want in your application.

Application Insights in VS 2017 will show the generated SQL if you are using the community edition of Visual Studio. For those of you who use SQL Server, you can get SQL Server Profiler and profile your requests to the database.

I challenge you all to take a look for yourself how LINQ to Entities and EF are getting your data. Entity Framework is a great ORM, however it will not stop you from making bad decisions.
