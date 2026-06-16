---
title: "C# Problem: Sql Executor Service"
summary: How to eliminate SQL connection boilerplate in C# repositories by wrapping Dapper in a reusable ISqlExecutor service.
category: "C# & .NET"
publishedDate: 2022-01-22
tags:
  - csharp
  - sql
  - dependency-injection
  - design-patterns
keyTakeaways:
  - Injecting a raw connection string into every repository class creates repetitive boilerplate and tightly couples each class to connection lifecycle management.
  - An ISqlExecutor service abstracts away connection creation, transaction management, and cleanup, letting repositories focus purely on query logic.
  - Providing both a non-transactional and a transactional implementation gives callers flexible control over commit semantics at composition time.
  - A cross-repository transactional executor enables multiple repositories to participate in a single atomic operation when registered with the correct DI lifetime.
draft: false
relatedArticles:
  - csharp-tip-sql-executor-service-dependency-injection
  - csharp-tip-sql-executor-service-extending
---

I make no claims about being the best developer around, but sometimes I have these "aha" moments that I can enjoy, even if it is only for myself.

![Sample Dependency Injection](https://cdn-images-1.medium.com/max/800/1*EUboWyiiXIeu0Z6KDyKLEQ.png)
*Sample Dependency Injection*

## Series Table of Contents

**Part 1: SQL Executor Service**
[Part 2: Dependency Injection](https://medium.com/dev-genius/c-tip-sql-executor-service-dependency-injection-4455efb453b0)
[Part 3: Extending Functionality](https://justin-coulston.medium.com/c-tip-sql-executor-service-extending-functionality-edad1c554f)

## Problem

I write a lot of software that interacts with SQL Server. We can, of course, go the raw ADO.NET route, but who wants to do that amount of work anymore? And yes, Entity Framework (EF) has its place but doing it right with the performance you'd expect takes deeper knowledge of the inner workings. So I have gravitated over the years to utilizing [Dapper](https://github.com/DapperLib/Dapper), a wonderful little micro-ORM.

My issue comes from having to inject primitive options into service classes (like `string connectionString`) and having to write boilerplate code around creating connections, transactions, and ensuring they're closed.

> In these lower level classes, I don't like utilizing .NET Core's Options pattern because it creates a strong dependency. If I'm going to have a strong dependency I'd rather be my own code.

For example, let's say I have a query service:

<script src="https://gist.github.com/e2d05c29d1ae16a79b721b91ce84e134.js"></script>

And of course, the implementation itself:

<script src="https://gist.github.com/036b1e6dcf6f0cf98e733f32d2f08e96.js"></script>

I struggle writing this over and over and over again… It's time-consuming and when there are close to 100 classes and methods with this type of boilerplate, it does start to be mundane…

> Let's also not forget that we have limited our ability to perform cross-repository transactions as well, which may be desirable.

## Solution 1: Simple Code Reuse

The simplest solution is to start by just re-using some code and creating some helper methods. A natural approach is to just add the following helpers to each class or add the method to some common area for sharing (could be static class, extensions, or base class).

<script src="https://gist.github.com/f9806cc79c7d0b31acbfdad250032ba3.js"></script>

But this still doesn't feel right for me. I want some more flexibility…

## Solution 2: SQL Executor Service

How about instead, I create a separate service to do this for me? I could then control the lifetime of service when it gets committed or refuse to use transactions at all. That kind of flexibility sounds nice to me!

Let's take a look at the SQL Executor Service:

<script src="https://gist.github.com/ed8d5df4d85ba7981cab84c89ba2de84.js"></script>

The naming could probably be a bit better, but I kept them the same for now.

What this allows us to do is inject a service into all repositories with different features set up. For instance, here is a simple non-transactional implementation:

<script src="https://gist.github.com/c3367df26f3d19770038d2b83eb69aee.js"></script>

In this case, a transaction is not created for each call and passes null to the function. This works just fine when working with Dapper. However, any other users may require checking for the transaction first.

Next up, adding in a transactional version is trivial:

<script src="https://gist.github.com/33dd30cb315c029da5465e7fad90895f.js"></script>

Now that I have some implementations I can work with (choose which you'd prefer to use) we need to use it.

<script src="https://gist.github.com/35fc2fe583be224654e5371e60d5d2bc.js"></script>

Nothing complicated or magical. Just simple. And saves a bunch of lines of code.

## Conclusion

What I like about this approach is that we can easily inject this `ISqlExecutor` anywhere. We can re-use it and ignore everything related to creating connections and transactions. This has cut down on the amount of code in my repositories substantially allowing me to strictly focus on Transact-SQL optimization.

Hopefully, you find this small trick useful in your own code! It's not meant to be creative or crazy. I have found many developers I work with will keep doing the same thing over and over again (myself included) and forget that we can create small abstractions to create efficiencies in our code.

Until next time!

## Bonus: Cross-Repository Transactions

Almost forgot about this last implementation. The following can provide a means of cross repository transaction management. This is especially useful in more complex web API calls where you need multiple repositories to work as one within a business service.

<script src="https://gist.github.com/fe824285d63e0fdbe13e04de41b2d02e.js"></script>

For this to work though, you have to be able to inject this service `Scoped` appropriately in your DI container. This can be pretty tricky if not done right. So I'll have to save that implementation discussion for another day. But if interested, let me know!
