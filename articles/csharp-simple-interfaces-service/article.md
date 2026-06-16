---
title: "C# Simple Interfaces: Service"
summary: How to design minimal service interfaces in C# and use extension methods to layer friendly method signatures on top without bloating the core contract.
category: "C# & .NET"
publishedDate: 2022-01-23
tags:
  - csharp
  - interfaces
  - design-patterns
  - dependency-injection
keyTakeaways:
  - Keeping a service interface as small as possible — ideally two core methods rather than seven — makes each implementation easier to write and maintain.
  - Extension methods let you restore convenient method signatures on top of a minimal interface without changing the contract or duplicating logic across implementations.
  - Because extensions are tied to the interface rather than any single class, every future implementation (SQL, in-memory, cached decorator) gets them for free.
  - The main limitation of this pattern is that extensions cannot access non-exposed internal state; if significant logic requires injected private services, it belongs on the contract directly.
draft: false
relatedArticles:
  - csharp-simple-interfaces-containers
---

Software is an art form in so many ways. There are 100s of ways to solve the exact same problem, each with its own set of pros and cons. However, you learn over time different ways to simplify your life while opening your options up when the future comes knocking with changes. Simple interfaces, in my opinion, are an important concept when building an architecture for your applications.

## Service Interface

I'm specifically talking about service interfaces in this article. An interface that has the sole purpose of performing some task or set of tasks. In contrast, there are data interfaces that hold information (think `IServiceCollection`). We'll talk that through next time.

Because an interface is a contract, keeping the contract as small as possible, tends to provide the most benefit. This is especially true if the interface holds all the necessary functionality you may need. However, I have found that developers tend to focus on the business rules first and design these interfaces around those rules in a very literal fashion. There isn't anything inherently wrong with this, but you can block yourself into a corner the more an interface is extended (the addition of functions) over time.

Here is an example inventory service interface I have seen in the past.

<script src="https://gist.github.com/c0fbf5cab35d65d17266794f38910c42.js"></script>

Again, this is natural to create. But we all know that as you start to implement this service, you will find yourself adding a lot of duplicate logic and checks within each method. Sure, you can create private functions to ease this pain, but we can re-design this interface to provide better cohesion, without losing out on the friendliness of the method signatures themselves.

Let's alter the interface to look more like the following.

<script src="https://gist.github.com/fa504069914f3d9a5684fc0df502f0dc.js"></script>

> Now, I want to make a note here for any newer developers. Even if you do not change the `public interface IInventoryService { ... }` code itself, changing the `InventoryRecord` or `InventoryQueryModel` effectively alters the contract that has been made within the code. Although adding parameters is easier in the class-based parameter models, it is the same as altering the parameters directly (from a contract standpoint). Care should be taken when altering any portion of a contract.

Now using this new service is actually more difficult, in my opinion, at least directly. But we have cut down the number of methods to 2, instead of 7. How can we get back the nice method signatures of the original contract?

Well, this is the beauty of Extensions in C#.

> An extension effectively allows you to "add" methods to existing types without creating new derived types or otherwise modifying the original type.

Here are a set of extensions that bring back the niceties of the original interface.

<script src="https://gist.github.com/8d1c3c38423d4ae31e285f5e79a9f374.js"></script>

For me, this is much nicer. However, the value may seem minimal at first glance. In some regards, this code could be in the implementation itself, and it wouldn't matter all that much.

The real value comes in when you start working with multiple implementations of this contract. For instance, I like to first build up in-memory versions of services to work out the logic and utilize those instances for unit testing purposes. When I have all the logic settled, I translate those preconditions, postconditions, and queries into SQL statements appropriately.

You may also have a need to handle multiple connector types (i.e. SQL Server vs MySql) in a single codebase. Again, these extensions prevent the need to add a bunch of boilerplate for each implementation.

<script src="https://gist.github.com/67ee74a9908c5b51e3926561a7912a57.js"></script>

But let's just say that you don't care about having more than one core implementation or even an in-memory version. If, and when, you start having some performance issues, you may decide that you would like to start caching common queries. You can create a cached implementation that decorates your core service class.

<script src="https://gist.github.com/06483979620920a43e2c03e379209365.js"></script>

Even in this case, other than the 2 methods we have defined, we do not have to implement the other 5.

With the extensions tied to the interface, and not any single implementation, we are able to utilize the methods for any combination of decorators or implementations we create in the future, minimizing code changes over time.

## Conclusion

Extension methods are your friends. Although, not everything can, or should be extension methods, they can really be a time saver for all sorts of cross-cutting concerns and implementation details. These "helpers" effectively extend your ability to write clean code, without destroying your original contract. It makes developing clean software easy and intuitive.

It should be noted, that a major downside to extension methods is the ability to utilize services. If a specific method requires some internally injected service, that is not exposed, you won't be able to utilize this technique. You can try to force it by setting field scopes to `internal` but then we start to create code smells. If you find that you have significant logic that cannot be handled from the core interface methods, it's probably best to simply put it on the contract to begin with.

Code Long and Prosper!
