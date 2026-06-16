---
title: "C# Tip: SQL Executor Service — Extending Functionality"
summary: How to use the decorator pattern with a simple ISqlExecutor interface to add cross-cutting concerns like performance logging, exception handling, and multi-tenancy without changing existing code.
category: "C# & .NET"
publishedDate: 2022-02-15
tags:
  - csharp
  - dotnet
  - dependency-injection
  - design-patterns
  - database
keyTakeaways:
  - The decorator pattern lets you wrap an ISqlExecutor with additional behavior — performance metrics, exception handling, tenant routing — without modifying the original implementation.
  - Stacking decorators composes complexity at the registration layer, keeping individual implementations focused and small.
  - A thin interface like ISqlExecutor enables multi-tenant connection switching transparently, reducing cognitive load in repository code.
  - The true value of an abstraction like this appears only when you extend it; the boilerplate removal is merely the entry point.
draft: false
relatedArticles:
  - csharp-problem-sql-executor-service
  - csharp-tip-sql-executor-service-dependency-injection
---

*How to add functionality without changing your original class*

![Sample Dependency Injection](https://cdn-images-1.medium.com/max/800/0*0szHlufaajgPZntB.png)
*Sample Dependency Injection*

## Series Table of Contents

[Part 1: SQL Executor Service](https://blog.devgenius.io/c-problem-sql-executor-service-deb459132a50)
[Part 2: Dependency Injection](https://medium.com/dev-genius/c-tip-sql-executor-service-dependency-injection-4455efb453b0)
**Part 3: Extending Functionality**

In part 1, I described a simple interface that could be used to execute SQL Commands. Its most apparent utility is to remove some boilerplate from your code avoiding large amounts of `using` statements, opening connections, etc. But this is only the tip of the iceberg of its utility.

<script src="https://gist.github.com/Jcouls29/ed8d5df4d85ba7981cab84c89ba2de84#file-isqlexecutor-cs.js"></script>

In part 2, I went through injecting the service through dependency injection and adding a scoped version to allow for a shared transaction. This proves useful when you want all SQL statements within a single ASP.NET request to be rolled back on error, especially across repositories.

In part 3, I'm going to walk through general approaches to extending functionality for different use-cases with this design.

## Introduction

The beauty in an interface like this is in its simplicity. A comment was made about a week ago on this specific interface that went a little something like this:

> I'm confused what the benefits of this approach are. So we have a wrapper for Dapper? Which is just an extension over IDbConnection?

And another comment,

> I think there is no exceptional value here, but a way to avoid writing boilerplate. Just inject the service + it looks cool :)

Nothing wrong with these comments at all. In fact, I appreciate that they were made. What it means to me, is that I didn't really communicate the full scope of what creating this interface can accomplish and how it saves you effort in the future.

I remember when I first saw an interface similar to this one. My first reaction was, "Wow… Why didn't I think of that?" I was actually frustrated with myself not simplifying my life earlier on. I mean, I've been writing SQL for years and I was always tired of the boilerplate, but what could I do? Then I saw something along these lines and a lightbulb went off.

To be fair, this could have been a bit more generic of an interface. It probably should have looked more like this,

<script src="https://gist.github.com/c128b8bc4b1cb38969a6f425d4b53104.js"></script>

Here I am using the interfaces `IDbConnection` and `IDbTransaction` instead. But honestly, at the time, it really didn't add any value to me. I wasn't writing a public library.

Anyways, what I saw in this interface is the removal of the boilerplate and the ability to easily switch out SQL implementations (like `ScopedTransactionSqlExecutor` or `NonTransactionSqlServerExecutor`). But this isn't really the full story of the value and savings it can provide.

Let's talk about decoration and building up an instance.

## Extending Functionality

I'm a huge fan of the decorator pattern. I like it because you can dynamically alter the behavior of a class through the use of a wrapper around your primary class. This is especially useful in scenarios where cross-cutting concerns may be prominent.

In a SQL case, what if we started having performance issues? How can we measure the performance of the queries from the perspective of the code?

With decoration, we simply create a wrapper class,

<script src="https://gist.github.com/c28ae89f51b39260c4efb11fd524018b.js"></script>

We inject another `ISqlExecutor` into this class (the decorated class) and a logger that will perform the writing of the metrics. From there, each method will start a stopwatch, run the method, and log the result.

A downside to this interface is the inability to see the SQL Statement being run. And without re-creating [Dapper](https://github.com/DapperLib/Dapper), with an interface-driven design, we have no way of intercepting these queries easily. We would have to create a custom `IDbConnection` and `IDbTransaction` and use these to proxy the calls. We could then grab the details of the last call and parameters at this point. But this complexity isn't really necessary right now. Instead, we simply include the stack trace. This will allow us to locate the code, which should inform the SQL being executed.

> If anyone would like to see this interceptor implemented, just let me know in the comments :)

Once we have the class created, we simply have to inject it within the DI container. We can make our lives easier by using the NuGet package [Scrutor](https://github.com/khellang/Scrutor),

<script src="https://gist.github.com/922295022597abdcc9357a2d7c580093.js"></script>

And finally, to use it in `Program.cs`,

<script src="https://gist.github.com/1089f4b56ffcbfb198c401a25f16e8ee.js"></script>

And that's it. Performance metrics will now be logged to your log file. I'll leave it up to you to make this configurable.

## Other Use Cases

#### Exception Handling

Another common use case that I come across is exception handling. Although I do not particularly agree to catch exceptions in a repository or service class, unless there is an absolutely wonderful reason to do so, it is a use case for some developers.

Here's a possible implementation we could use,

<script src="https://gist.github.com/838abd03ca9b31707d1b4c1e80018863.js"></script>

We did something similar to the performance metrics and wrapped an executor that we want to catch exceptions.

We could then add an extension method,

<script src="https://gist.github.com/4f85cf583d24900d661711396588e8ea.js"></script>

What's nice about these small implementations is that we can really build them up in our system. We aren't limited to one. We can have many different ones all wrapped around each other building a more complex setup. Let's just show what this might look like in `Program.cs`,

<script src="https://gist.github.com/de8fd62499037807ade17a239224c4c2.js"></script>

This setup is basically creating the following,

```
new HandleExceptionsSqlExecutor(
   new LogPerformanceSqlExecutor(
      new ScopedTransactionSqlExecutor(...)
   )
)
```

Depending on the order, we can handle the exception first, then log the performance. You could add another performance one on the outside and have two of them, if you were so inclined.

The point is, we are working at a higher level. We're not having to write the raw code. Instead, we are composing our classes together to form functionality, as if it was a higher-level language.

#### Tenant Selection

What if you have a more complicated scenario, like a multi-tenant application, where each database contains a different tenant's data? Well, the beauty of the interface, is that we can create another implementation that can switch tenant connections implicitly.

<script src="https://gist.github.com/80bc4f304f16a1908436527cf7620382.js"></script>

This is as simple as it really needs to be. The `ITenantOptions` is a scoped service that will determine which tenant is generating the request through the `Authorization` header or some other HTTP Header. It will then look up the options and inject them into the executor.

We would need to create a `ISqlExecutorFactory` but this is fairly trivial to create,

<script src="https://gist.github.com/c49791ff8dee948e629371ef88bb4cc9.js"></script>

Now in our repositories, we don't have to figure out how to inject the appropriate connection strings (assuming your `ISqlExecutor` is also injected scoped). This will be handled with a service that resolves it for us. For me, this makes thinking about specific repository code easier. I don't need to worry about whether I'm in the right tenant or not. Instead, let me make an assumption here and just work on the best SQL Code I can without thinking about that level of detail.

## Conclusion and Sign-Off

I wrote this article in response to the comments only because I feel the full extent of the benefits were not adequately explained. If Dapper had been a more interface-based design, this wouldn't actually be necessary. Unfortunately, it wasn't. So, I created this interface to provide me a level of flexibility in my code that I wouldn't normally be able to achieve using traditional approaches (using `SqlConnection` directly)

If we're careful in writing SQL, not to use proprietary keywords, you can also use this methodology to allow connecting to different database engines by tenant, or even allowing the transition to a different engine easier (in the case of converting from something like MySQL to SQL Server). I have only had to do this once, but it does save a lot of time.

There are a lot more use cases than I presented here, but I hope that this helps to clarify the richness that can be brought to a design like this. We can use it in so many ways, extend it in so many ways, and address real cross-cutting concerns.

As a final note, this interface may not be perfect. There might be better ways to approach it than what I have presented here. Ultimately, I'm hoping to provide a tool for you to utilize as you grow as a developer to create more efficient coding structures, allowing for easier modification in the future.

Until next time!
