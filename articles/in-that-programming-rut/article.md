---
title: "In that Programming Rut…"
summary: A reflection on the comfortable patterns experienced developers fall into and how defining your own quality metrics is the first step to breaking out.
category: "Developer Practices"
publishedDate: 2022-12-07
tags:
  - career
  - design-patterns
  - interfaces
  - code-quality
keyTakeaways:
  - Repeating the same approach in every project may signal mastery of a technique, but without deliberate reflection it becomes a rut rather than a skill.
  - Define your own programmer identity by explicitly prioritizing software qualities — readability, maintainability, reusability, flexibility — and measure your code against them.
  - Patterns like minimal singleton interfaces, extension methods, and decorators have real trade-offs; acknowledging the downsides is part of owning your identity as an engineer.
  - Comparing yourself to who you were yesterday rather than to peers is a healthier and more actionable measure of growth.
draft: false
---

![Photo by Jamie Hagan on Unsplash](https://cdn-images-1.medium.com/max/800/0*q2-XaHr3TTjMt6KX)
*Photo by [Jamie Hagan](https://unsplash.com/@dearjamie?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Have you ever been in a programming rut, where you find yourself writing code the same way for each and every project; where you feel that you're not learning, or even pushing the boundaries of your capabilities as an engineer?

I sure have.

In fact, I feel like I'm in a rut right now. It's like my experimentation-esque personality has vanished and is visible in my code every day. I admit that after 20+ years of experience and experimentation that I have come to a fairly solid approach to software design and architecture (especially for business applications). However, it does start to make you wonder whether you have hit the ceiling or you have simply found a comfortable level of certainty in your abilities.

I wrote an article a while back called [The True Way to Software Mastery](https://medium.com/dev-genius/the-true-way-to-software-mastery-64dc0bf8a2f8) where I talk about the idea of becoming a master of software. Although, I feel I have earned the respect of my colleagues in the field of software design and architecture, you can never really be sure. And in reality, we shouldn't compare ourselves to other people today, but instead compare ourselves today to who we were yesterday. So when I compare myself today to myself yesterday, not much has changed… thus the rut.

## My Current Rut (.NET C#)

I find myself doing the same things over and over again for each project. Here are some of the items I seem to repeat over and over again.

#### Interface Design

I build the simplest interfaces I can possibly build and drive them to ALWAYS be scoped to singleton (from a DI perspective). I will even go out of my way to do this. For example,

```csharp
public interface IWorkflowContextAccessor
{
    IWorkflowContext Current { get; }
}
```

This interface is similar to the `IHttpContextAccessor` and will allow me to grab this context from any other class. However, for an ASP.NET application, I could just as easily inject the `IWorkflowContext` as a scoped dependency instead and determine it on-the-fly.

**My Reasoning**: Singleton-scoped dependencies can be injected into all DI dependencies scopes (transient, scoped, and singleton). But the reverse is not true. To me this lowers the cognitive load on remembering which services can be injected into what classes.

In consequence, everything I need for a function, MUST be passed around through method arguments and limits stateful class usage, which can be useful.

#### Extension Everything

I have a habit of creating extension methods for everything in C#. I use the minimal interface designs to make new implementations easier, while allowing more complicated logic to exist elsewhere.

Here is a sample repository interface that I might create:

```csharp
public interface IRepository<T> where T : Model
{
   Task UpsertAsync(T model);
   Task DeleteAsync(Identifier id, Identifier? scopeId = null);
   Task<T[]> FindAsync(ExactMatchQuery query);
}

public abstract class Model
{
   public Identifier Id { get; set; } = Identifier.Empty;
   public Identifier ScopeId { get; set; } = Identifier.Empty;
   public string Name { get; set; } = string.Empty;
   public string Description { get; set; } = string.Empty;
   public DateTime CreatedTimestamp { get; set; } = DateTime.UtcNow;

   public Dictionary<string, string> Properties { get; set; } = new Dictionary<string, string>();
}

public class ExactMatchQuery
{
    public Identifier? Id { get; set; }
    public Identifier? ScopeId { get; set; }
    public string? Name { get; set; }
    public string? Description { get; set; }

    public DateTime? MinimumCreatedTimestamp { get; set; }
    public DateTime? MaximumCreatedTimestamp { get; set; }
    
    public int Offset { get; set; } = 0;
    public int Limit { get; set; } = 10;
}
```

Then I would use extensions all over the place to do very specific work:

```csharp
public static class RepositoryFindExtensions
{
    Task<T?> FindByIdAsync<T>(this IRepository<T> repository, Identifier id, Identifier? scopeId = null) where T : Model
    {
        var found = await repository.FindAsync(new ExactMatchQuery
        {
            Id = id,
            ScopeId = scopeId
        };

        return found.FirstOrDefault();
    }
}
```

**My Reason**: You can do a lot with simple interfaces without having to embed them directly within the interface itself if you abstract appropriately. This allows for future implementations to have less requirements, thus allowing for these extensions to be reused for any implementation.

But now the logic is outside the main repository classes. Good luck trying to find it without a good IDE like Visual Studio or VSCode. This separation can be confusing for junior developers.

#### Deck the Halls Decorations

Then there is this design pattern called Decorators that I find myself using constantly. I like the ability to add functionality dynamically and so I create layers of interfaces to allow for adding new code, rather than changing old code.

```csharp
public interface ISqlExecutor
{
    Task<T> ExecuteAsync<T>(Func<ISqlConnection, ISqlTransaction, T> execute);
}
```

Then I would have at least two implementations. One for my main executor and one for performance monitoring. Of course, in a typical scenario, I usually have more than this.

```csharp
public sealed class SqlServerExecutor : ISqlExecutor
{
    public SqlServerExecutor(...) { }

    public async Task<T> ExecuteAsync<T>(Func<ISqlConnection, ISqlTransaction, T> execute)
    {
        using (var sqlConnection = new SqlConnection(...))
        {
            await sqlConnection.OpenAsync();
            using (var sqlTransaction = sqlConnection.BeginTransaction())
            {
                var results = await execute(sqlConnection, sqlTransaction);
                sqlTransaction.Commit();
                return results;
            }
        }
    }
}

public sealed class PerformanceMonitoringSqlExecutor : ISqlExecutor
{
    private readonly ISqlExecutor _innerExecutor;
    
    public PerformanceMonitoringSqlExecutor(ISqlExecutor inner)
    {
        _innerExecutor = inner ?? throw new ArgumentNullException(nameof(inner));
    }

    public async Task<T> ExecuteAsync<T>(Func<ISqlConnection, ISqlTransaction, T> execute)
    {
        var watch = StopWatch.StartNew();
        var results = await _innerExecutor(execute);
        watch.Stop();
        
        // TODO: Log these results somewhere
        
        return results;
    }
}
```

**My Reason**: This allows me to dynamically build up my needs depending on configuration or other methodology (i.e. A/B Testing). In this way, I can avoid direct feature flags and bloated code like log statements in every implementation. Moreover, I don't necessarily need to copy this code into every implementation when a new one arrives.

But now the code is more complex and can be hard to reason about, especially if all you see is the interface in the core business logic. How would you know anything is getting logged? There is a lot of boilerplate to keep this easy to read for younger developers.

## The Cure?

Nothing I am doing above, to my knowledge, is necessarily bad or poor practice. It really depends on the environment you're working in. However, I've been doing this for years; the same way for all new projects. I got here because of the experimentation over those years. I trialed techniques and cut the ones that made things worse. But then what does "worse" mean?

#### Breaking Out

To break out of a rut, you first need to know what your values are as a programmer. What is it that you value? And how do you judge your own code (what are your metrics; whether subjective or objective)?

I have my own set of metrics I use to evaluate these type of situations:

- **Readability**: Will the team responsible for working on this code now, and in the future, understand the code, where to find it, and how it works? If not, more work needs to be done.
- **Maintainability**: Can I minimize the number of times this code has to be touched? For instance, by using decorators, I can eliminate the need to change implementation code. Instead, I can create new implementations, in an additive way, for new functionality.
- **Reusability**: How much has this code been abstracted? Can I minimize my future boilerplate code with the code I have produced? Is it in a library that I can use for future projects, saving time in future development.
- **Flexibility**: When the requirements change, how much code has to be changed, if any? Can feature flags be used (or decorators)? Or can I create a simple implementation that replaces a current one? By creating extension points, I offer flexibility when features inevitably get added or changed.

Of course, there are more, but these are my primary goals. I believe in the recent months, I have stopped asking myself these questions and caused this rut to be formed.

The reality is that software will never be perfect in all of these aspects at once. Trade-offs are required when balancing these software qualities. So the measure of priority I put on each of these qualities affects the direction my code evolves. The question then arises as to what is the priority, or weight, I put into each of these qualities?

I don't really have an answer to that question, but I believe that for any developer to have an identity, they must first define the qualities of their software they are aiming to achieve. Then, they must prioritize those qualities to optimize to their standards. This is what I would call your **Programmer Identity**. And this identity is used to measure you against yourself, but also potential employers. This is when, I believe, programmers are rejected by employers. If the identity of the organization and the developer do not significantly overlap, then a programmer will be doomed to fail in his/her position. Not for any skillful reason, but because of his inherent personality towards software.

I'm going to keep searching to find my way out of this rut. Thanks for listening in on my rant. If you have any suggestions, by all means, suggest away.

Until next time!
