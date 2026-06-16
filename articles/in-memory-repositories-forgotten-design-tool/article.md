---
title: "In-Memory Repositories: A Forgotten Design Tool"
summary: How building in-memory repository implementations first — before touching SQL — accelerates design, drives out business rules through TDD, and simplifies the eventual database translation.
category: "C# & .NET"
publishedDate: 2022-01-26
tags:
  - csharp
  - testing
  - design-patterns
  - database
keyTakeaways:
  - Starting with SQL table design locks you into infrastructure decisions early and forces changes across multiple layers every time business rules evolve.
  - Building an in-memory repository first lets you defer storage technology decisions, surface preconditions through TDD, and deliver working logic to stakeholders immediately.
  - The same in-memory implementation that drives tests during development becomes a reusable real implementation in higher-level service tests, reducing the need for mocks.
  - Once the business rules are stable in the in-memory version, translating preconditions and queries to SQL is straightforward and produces a coherent database structure in one pass.
draft: false
---

![Photo by Luan Gjokaj on Unsplash](https://cdn-images-1.medium.com/max/800/1*GAVayU_GJFwkIuN6mRoihw.jpeg)
*Photo by [Luan Gjokaj](https://unsplash.com/@luangjokaj) on [Unsplash](https://unsplash.com/s/photos/ram)*

## Preface

Over the years, I've approached design and development in many different ways. When I first started developing, I would normally start with whatever aspect I thought was the most critical. In the majority of instances, that would be data modeling. But not class design, I would start with the SQL models. I would determine all the columns I would need, the size requirements, and joins right up front. Where the data was stored, seemed like it was the most logical starting point. Then I would work up the stack to the software data models, then the services I would need, and finally end up working the user-interface portion, whether that was API controllers (or WCF SOAP) or the actual views (ASP.NET).

Like anything in the development world, this isn't necessarily wrong, but I learned over time the difficulties in working this way. A few things would constantly happen when I used this flow:

1. Always seemed like the user only cared about the UI / Interface portion of the system over anything in the backend. So inherently all that work with the data didn't really make an impact early.
1. They seem to change what they need almost daily (when I was lucky it was weekly). This would cause the data elements to constantly change, including sizing, structure, and table design.
1. Every time a change was needed, I had to work through 3 or 4 layers to provide them any value, even when we weren't going live yet (just to show progress). This required a lot of DTO(s), interfaces, and objects to change through the whole stack.
1. Automated testing with the business logic always took longer, especially if I was trying to test the SQL designs along with the business logic.

It was very frustrating. It never felt "right" the way I was approaching this. The clients felt like everything took "forever" too.

Keep in mind, projects that were already established, weren't impacted as heavily as greenfield projects working this way. But when working on something brand new, the time to providing value felt elongated.

## My New Approach

Over time, I realized that requirements change a lot, especially in a new application (business rules and data element alterations being the most common). So I decided to move away from pure data storage up front and focus on the logic and data design before worrying about table structure at all. In fact, at this point in my career, I wait until the last possible moment to start putting storage together at all.

The way I accomplish this is to establish a solid interface early on in the project. Then, when using a Test-Driven Design / Development (TDD) approach, I start building up an In-Memory model of repositories and services.

There are a couple HUGE advantages to starting here first.

1. There is no infrastructure needed early on. This means no decisions on technology is inherently required at the beginning of a project (i.e. MongoDB, SQL Server, or raw files). You can wait to make this architectural decision later (I also understand that in many cases this may already be laid out for you, but still…).
1. You have a built-in testing implementation that will more appropriately represent your logic when doing automated testing, avoiding any heavy needs for Mocks or Fakes.
1. Establishes all business rules early on that can be easily translated to SQL, or any other language, keeping the cognitive load down by focusing all efforts at the same time. What I mean by this is that you can design the whole database structure at once, instead of piecemealing the design throughout the project for a more unified structure.
1. Avoids the expense of multiple steps to introduce a new feature, meaning no scripts have to be run first, or provisioning is required to run this locally in a development environment. Hitting play is trivial.

Since I've moved in this direction, I've never gone back to starting with the data infrastructure first. But I must admit, I find developers in the same place I was 10 years ago starting with SQL scripting first. And I get it. So, in hopes of showing an easier way, I will walk through an example next.

## Example Overview

In this example, I'm going to focus only on a single function of a repository interface. Specifically, it will be a product repository. Simple enough. But I will go through the TDD process (as I see it) as we work to show the benefit.

> Note: I'm not a TDD purist by any stretch. I take a pragmatic view by nature, but try to hold to many of the tenants.

Now although this is primarily done in C#, I firmly believe we can apply this in any OO language.

Let's jump in.

## Interface Definition

As I mentioned previously, I'm going to start with the interface. I rarely let TDD define my interface (although I do allow TDD to change it). Normally, this is because interfaces are contracts that will require adherence. Here's my current design:

<script src="https://gist.github.com/813dc2660b493389f3edf8cc5ee4b49a.js"></script>

This is a very simple CRUD interface. Basic I know, but common enough that it will still be useful. We're going to focus on the `AddAsync` method.

## Step 0: Preparing for TDD

I'm partial to [xUnit.net](https://xunit.net/) myself, so that's what I will use for the testing framework. In this specific situation, I will typically skip a few TDD steps and establish a baseline implementation like below:

<script src="https://gist.github.com/59693caa5eea914751c230882ec8539a.js"></script>

Once I have this available, I then go and create the xUnit class for my testing. Usually, it will start out looking like the following:

<script src="https://gist.github.com/0526a6a681416465da153a03bd10c908.js"></script>

At this point, it's time to start building out my implementation.

## Step 1: Building out the In-Memory Implementation

I will be presenting the approach with some back-to-back code examples, where I establish 1 test, then how it is implemented (a passing test). Just skip to the end, if you'd like to see the finished product.

> Given Valid Product Object, When Added, Then Non-Zero Int Returned

<script src="https://gist.github.com/f24e8972e17cac9ab150bd557fb7701d.js"></script>

> Given Two Unique Product Objects, When Adding Both, Returned Values are Unique and Not Equal to Zero

<script src="https://gist.github.com/762e822fd03023362ec585cf4550ade0.js"></script>

> Given Added Product, When Get By Id, Then Product with Same Details Returned

In this next one, it was time to start forcing the design a little bit (again, pragmatic TDD). So I need to do a basic implementation for the `GetAsync` call so I can start performing other types of tests.

You'll also notice, I started performing some refactors to make the tests smaller.

<script src="https://gist.github.com/1667a2ae755d3772ae7bc46df07619f6.js"></script>

> Given Invalid Product Name, When Added, Throws Exception

<script src="https://gist.github.com/e869e227c45ce29ef183f9d99c5f164f.js"></script>

> Given Added Product, When Add New Product with the Same Name, Then Exception Thrown

You'll see here that I continue to refactor the code a bit more to ease the testing burden. Refactoring tests should work the same as it does in production code (in my humble opinion).

<script src="https://gist.github.com/7a3587a76f54a702ec990f726bd9479b.js"></script>

> Given Null Product, When Added, Throws Exception

This is an obvious one…

<script src="https://gist.github.com/8640103232697c6fc579178b2cae9aba.js"></script>

> Given Null Description Added, When Retrieved, Then Description is Empty String

<script src="https://gist.github.com/ca9f182a0f9eca444ed669c5368a58aa.js"></script>

> Given Name Too Large, When Added, Throws Exception

> Given Description Too Large, When Added, Throws Exception

<script src="https://gist.github.com/307d1dfc27647eb66ef6c0c76ea6c675.js"></script>

## Code Summary

The final test and implementation code is below:

<script src="https://gist.github.com/37498a5cb14a6498448f530b51fdde3b.js"></script>

<script src="https://gist.github.com/570e10484491620bb80472881c3362e1.js"></script>

We focused almost entirely on `AddAsync` with a little help from `GetAsync`. This isn't too complicated to do, using an In-Memory implementation. You can immediately see the benefits though:

1. We have our pre-conditions figured out.
1. We can now use this implementation for future tests that may need an instance of `IProductRepository`.
1. These tests are fast to run!

But the purpose of this was to also provide a means of creating a database structure. Let's do that next.

## Step 2: SQL Implementation

I'm partial to Microsoft SQL Server (what I know best) and utilizing [Dapper (micro-ORM)](https://github.com/DapperLib/Dapper). So let's show a quick design of a table, and a partial SQL Server implementation with the rules we just established.

<script src="https://gist.github.com/e97884e1aa0d3619815b2a0a87652398.js"></script>

You can see in the designed table, that I used `64` for the max length for name and `256` for description, as determined by the testing. I also made them both `NOT NULL`. Naturally, the primary key is an `INT` to correlate with our model.

Now the implementation to add a product to the table.

<script src="https://gist.github.com/d7fcd00a80475ad2526bd0da0f9e7932.js"></script>

> If you haven't seen my usage of `ISqlExecutor` see the article [C# Tip: SQL Executor Service](https://justin-coulston.medium.com/c-problem-sql-executor-service-deb459132a50) for my approach here.

You'll notice a few things:

1. The pre-condition logic is the same. Although I didn't abstract it out, you can very easily have this logic separated in a way to be re-usable for these types of implementations.
1. I named the columns the exact same way as the object. With any ORM, this is just simpler to deal with. However, I admit that this will not always be the case.
1. We didn't set the product identifier on the object itself. In this way, the In-Memory implementation does NOT work the same as the SQL Server version. However, you can, at your discretion, perform this operation (probably best that you do).

This is a very simple implementation. However, I find the value is amplified more when we get to more complicated queries, like aggregate queries, counters, complex joins, and more.

## Conclusion

In this article, I walked through my whole process of developing an implementation (a single method) from testing to SQL Server implementation. In a real-world environment, the logic can become much more complicated, especially as you venture down services (and not simple CRUD repositories). But the same applies to these services as well.

The benefit with this approach is the ability to have your logic defined before putting too much work into the database structure. It also provides a huge benefit for future unit / integration testing. The ability to use a real implementation with the same core logic and conditions as the SQL version in higher level services allows for better tests to be derived. Although, I see value in mock services, you do lose significant business logic and checks (unless you explicitly make these for each test. Why not just use a real implementation?).

Lastly, I like this approach because it helps test your interface design early. In CRUD services, it isn't as big of an issue, but when designing business services or aggregate repositories, the advantages are more pronounced.

I know this is a very simple tip, but I have found it to be extremely impactful on my productivity as a developer. I hope you find it useful as well.

Til next time!
