---
title: A Response to "Stop Using If-Else Statements"
summary: An expansion on the popular advice to avoid if-else statements, covering state machines, early returns, object typing via DI, and the cases where if-else is still the right tool.
category: "Developer Practices"
publishedDate: 2022-12-31
tags:
  - design-patterns
  - csharp
  - code-quality
  - dependency-injection
keyTakeaways:
  - State-based if-else chains are best replaced with the State Pattern, which isolates transition logic into dedicated classes and makes adding new states a matter of implementing an interface.
  - Early returns (guard clauses) eliminate else blocks by inverting condition checks and returning immediately, flattening nested logic without changing its meaning.
  - Object typing—replacing string/enum dispatching with interface implementations injected via DI—removes the need to hunt down every call site when adding a new variant.
  - Some algorithms genuinely resist restructuring; forcing if-else elimination in tight loops with interdependent branches can hurt readability more than it helps.
draft: false
---

![By The Author](https://cdn-images-1.medium.com/max/800/1*70AIET1ybLBzNDdu29ZozQ.png)
*By The Author*

This article is in response to [Nicklas Millard's](https://medium.com/u/7c7a43b3d9de) post "[Stop Using If-Else Statements](https://medium.com/swlh/stop-using-if-else-statements-f4d2323e6e4)". However, as I went through the comments, you can clearly see that folks look at the article as clickbait. And I can see why. Even Nicklas himself says he hasn't written an If-Else statement in years (in the comments).

![Comment from Stop Using If-Else Statements](https://cdn-images-1.medium.com/max/800/1*8nH7sm9UjlOsE1FHYJ1SyQ.png)
*Comment from Stop Using If-Else Statements*

The thing is, I actually agree with him in principle. `if-else` statements are harder to maintain in most scenarios, especially as the system evolves. However, in quick and dirty utility apps, I expect the overhead of using State objects, and such, is overkill.

But I think what I want to do is clarify the other scenarios `if-else` statements can be removed. Nicklas only covered a single use-case in his article, state-based `if-else` statements. There are other scenarios to keep in mind.

## Case #1: State-Based

I would recommend looking at his article [here](https://medium.com/swlh/stop-using-if-else-statements-f4d2323e6e4) for his example. He does a good job explaining it. However, as a quick overview, here is a state-based `if-else` setup we might want to convert using the State Pattern,

```csharp
public class Order
{
  public string State { get; set; } = "new";

  // All other properties, methods, etc.
}

public void AdvanceOrder(Order order)
{
  if (order.State == "new")
  {
    // TODO: Validate New Order
    order.State = "received";
  } else if (order.State == "received")
  {
    // TODO: Send to Warehouse Team
    order.State = "processing";
  } else if (order.State == "processing")
  {
    // TODO: Send Shipped Email to Customer
    order.State = "shipped";
  } else if (order.State == "complete")
  {
    // TODO: Update Order Information and Send Received Email to Customer
    order.State = "complete";
  } else if (order.State == "cancelled")
  {
    throw new OrderCancelledException("...");
  }

  SaveOrder(order);
}

public void CancelOrder(Order order)
{
  order.state = "cancel";
  SaveOrder(order);
}
```

This contrived example, without any major branching logic, could be re-organized into a state machine fairly quickly. The interface could look like this,

```csharp
public interface IOrderState
{
  // Used for database storage and showing the state to an API / User
  string DisplayName { get; }

  IOrderState AdvanceOrder(Order order);
  IOrderState CancelOrder(Order order);
}
```

From here, we implement the states,

```csharp
public sealed class NewState : IOrderState
{
  public string DisplayName => "new";
  public IOrderState AdvanceOrder(Order order) => new ReceivedState();
  public IOrderState CancelOrder(Order order) => new CancelledState();
}

public sealed class ReceivedState : IOrderState
{
  public string DisplayName => "received";
  public IOrderState AdvanceOrder(Order order) => new ProcessingState();
  public IOrderState CancelOrder(Order order) => new CancelledState();
}

public sealed class ProcessingState : IOrderState
{
  public string DisplayName => "processing";
  public IOrderState AdvanceOrder(Order order) => new ShippedState();
  public IOrderState CancelOrder(Order order) => new CancelledState();   
}

public sealed class ShippedState : IOrderState
{
  public string DisplayName => "shipped";
  public IOrderState AdvanceOrder(Order order) => new CompletedState();
  public IOrderState CancelOrder(Order order)
    => new AlreadyShippedException("...");
}

public sealed class CompletedState : IOrderState
{
  public string DisplayName => "shipped";
  public IOrderState AdvanceOrder(Order order) 
    => throw new AlreadyCompletedException("...");
  public IOrderState CancelOrder(Order order)
    => throw new AlreadyCompletedException("...");
}

public sealed class CancelledState : IOrderState
{
  public string DisplayName => "cancelled";
  public IOrderState AdvanceOrder(Order order) 
    => throw new OrderCancelledException("...");
  public IOrderState CancelOrder(Order order)
    => throw new OrderCancelledException("...");
}
```

Not complicated at all to do this, and it can be very quick. The last piece is the order object itself,

```csharp
public class Order
{
  public IOrderState State { get; private set; } = new NewState();
  // All other properties of the order

  public void AdvanceOrder()
    => State = State.AdvanceOrder(this);
  public void CancelOrder()
    => State = State.CancelOrder(this);
}
```

And TA-DA! We have a simple state machine.

These types of systems can be useful, although I rarely find myself implementing these in the way shown above. Instead, I prefer to build workflow systems. I wrote an article called [Advanced C# — Custom Workflow Engine](https://medium.com/dev-genius/advanced-c-custom-workflow-engine-a52ba44bb2c4) about this exact thing.

## Case #2: Pre/Post-Conditions and Early Returns

The second case where I find we can remove if-else statements is in the basic condition checks within a function. Let's take this for example,

```csharp
protected void ValidateRequest(Model request, ICollection<string> validationErrors)
{
  if (request != null)
  {
    if (string.IsNullOrWhiteSpace(request.Name))
      validationErrors.PropertyRequired(nameof(request.Name));
    if (string.IsNullOrWhiteSpace(request.Description))
      validationErrors.PropertyRequired(nameof(request.Description));
  } else {
    validationErrors.ArgumentRequired(nameof(Model));
  }
}
```

In this case, I have separated out my validation before a primary function is called. However, I use the negated form to check the request (not equals `!=`). If we invert this, and early return, we can make it look a bit cleaner,

```csharp
protected void ValidateRequest(Model request, ICollection<string> validationErrors)
{
  if (request == null)
  {
    validationErrors.ArgumentRequired(nameof(Model));
    return;
  }

  if (string.IsNullOrWhiteSpace(request.Name))
    validationErrors.PropertyRequired(nameof(request.Name));
  if (string.IsNullOrWhiteSpace(request.Description))
    validationErrors.PropertyRequired(nameof(request.Description));
}
```

Here's the thing about this code. We aren't removing the `else` logic at all. We're simply syntactically changing it. The `return;` is how we're replacing the `else` keyword. If we remove the `return`, we would have to add the `else` back into the code. This is called "early returning."

Some folks have a rule that conditionals should always be the positive or the negative form. For instance, some people feel that `if (!string.IsNullOrEmpty(value)) { }` is less readable. However, for early returns, you really cannot be hampered by this philosophy. It WILL get in the way.

Now the above code shows an example of Pre-Conditions. However, this certainly applies to Post-Conditions or any other internal state of the function. The moment you can return from the function, DO IT! In this way, you avoid `else` statements.

## Case #3: Object Typing

The third major use-case I see is the idea of object typing, where a `string` or enumeration is used to select logic of some kind.

Let's say we want to seed a tournament algorithm. We could have code like this,

```csharp
public class Tournament {
  // ... All other Tournament Code ...

  public async Task<IEnumerable<SeedResult>> SeedAsync()
  {
    if (tournament.SeedAlgorithm == "ordered")
    {
      // TODO: Run Ordered Algorithm
      return results;
    } else if (tournament.SeedAlgorithm == "random")
    {
      // TODO: Run Random Algorithm
      return results;
    } else if (tournament.SeedAlgorithm == "total-score")
    {
      // TODO: Run Total Score Algorithm
      return results;
    } else {
      throw new InvalidAlgorithmException("...");
    }
  }
}
```

Every time you add another algorithm, you will have to update this method. But honestly, probably not a big deal. The real problem with this methodology is when you use the `Tournament.SeedAlgorithm` in multiple places within your code. Maybe there is a spot you check the algorithm type to perform some logic. But maybe there are restrictions on the UI depending on the algorithm. Now you have logic for "seeding" in multiple places. When you add another algorithm, you'd have to be absolutely positive you caught all instances within your code.

This is the worst.

Yes, IDEs allow us to quickly find all references. But that doesn't help for non-loaded assemblies. So you always run a risk.

Instead, we should create a service type to handle these scenarios. Let's look at an example here,

```csharp
public interface ISeedAlgorithm
{
  string Name { get; }
  bool IsVisibleToUser { get; }

  Task<IEnumerable<SeedResult>> RunAsync(Tournament tournament);
}
```

From here, we implement it a few times,

```csharp
public class OrderedSeedAlgorithm : ISeedAlgorithm
{
  public string Name => "ordered";
  public bool IsVisibleToUser => false;
  public async Task<IEnumerable<SeedResult>> RunAsync(Tournament tournament)
  {
    // TODO: Run Algorithm
    return results;
  }
}

public class RandomSeedAlgorithm : ISeedAlgorithm
{
  public string Name => "random";
  public bool IsVisibleToUser => true;
  public async Task<IEnumerable<SeedResult>> RunAsync(Tournament tournament)
  {
    // TODO: Run Algorithm
    return results;
  }
}

public class TotalScoreSeedAlgorithm : ISeedAlgorithm
{
  public string Name => "total-score";
  public bool IsVisibleToUser => false;
  public async Task<IEnumerable<SeedResult>> RunAsync(Tournament tournament)
  {
    // TODO: Run Algorithm
    return results;
  }
}
```

And now you have a separate set of algorithms.

Now, by itself, this doesn't present a whole lot of benefit in the near term. However, from a maintenance standpoint, long-term, we can find a number of benefits,

1. We can use Dependency Injection (DI) Containers to supply these algorithms to other classes that need them.
2. We can inject any number of dependencies into these that the algorithm may need without polluting a single class that uses `if-else` logic.
3. We can quickly add a new algorithm by implementing the interface and adding it to your DI container registry.

Overall, this is a better design for most use-cases.

## Case #4: Algorithms

The last use-case to talk about is algorithms, specifically algorithms that cannot allow for early exits or object separation. For instance, I had a case recently where I was setting up rounds for a tournament where "bye" rounds were possible. It looks something like this,

```csharp
for(int iMatch = 0; iMatch < len; iMatch++)
{
    var p1 = top[iMatch];
    var p2 = bottom[iMatch];

    // ... other code ...

    // This is a bye for a participant
    if (p1 == -1)
    {
      isBye = true;
      matchName = $"{participants[p2].Name} Bye";
      assignments = new[] { new MatchStationAssignment { Participant = new Reference<Participant> { Id = participants[p2].Id, DisplayName = participants[p2].Name }, Seat = 2 } };
      results = new[] { new IndividualMatchResult { Assignment = assignments[0], Result = MatchResult.DefaultWin } };
    } else if (p2 == -1)
    {
      isBye = true;
      matchName = $"{participants[p1].Name} Bye";
      assignments = new[] { new MatchStationAssignment { Participant = new Reference<Participant> { Id = participants[p1].Id, DisplayName = participants[p1].Name }, Seat = 1 } };
      results = new[] { new IndividualMatchResult { Assignment = assignments[0], Result = MatchResult.DefaultWin } };
    } else
    {
      matchName = $"{participants[p1].Name} vs. {participants[p2].Name}";
      assignments = new[]
      {
          new MatchStationAssignment { Participant = new Reference<Participant> { Id = participants[p1].Id, DisplayName = participants[p1].Name }, Seat = 1 },
          new MatchStationAssignment { Participant = new Reference<Participant> { Id = participants[p2].Id, DisplayName = participants[p2].Name }, Seat = 2 }
      };
    }

    // ... other code ...
}
```

Without going into too much depth about what this code does, it would be fairly difficult to break this out without else statements. This is mainly due to the "other code" running around this `if-else` setup. I'm sure I could find some clever way to do it if I spent the time, but honestly, the readability would be atrocious.

I'm open to any suggestions :)

## Conclusion

The reality of `if-else` logic and reconfiguring it like we did above, is that the logic still exists. Open up Visio, or Diagrams.IO, or whatever tool you use, and start to map out your logic. It will ALWAYS be there. We really are breaking these out for two reasons,

1. **Readability**: My opinion, exiting early from a routine and being certain after that point that my objects are good, makes programming so much easier. Plus having all your code in one place, breaking them into classes, makes finding details simpler.
2. **Maintainability**: Having to search a code base for all use-cases of an enumeration is painful. And on top of that, having to change tested code, always makes me nervous. Instead, adding new classes and leaving the old alone, always gives me the warm and fuzzies.

But the point is, the logic exists. We're just trying to make it just a bit better from those two perspectives.

Overall, I agree with the sentiment of Nicklas. We should do everything in our power to avoid `if-else` setups. However, there will absolutely be cases where you shouldn't do it (like case #4). It would only make things worse (unless someone shows me something better). We cannot ever be dogmatic about any programming principles. You'll eventually be proven wrong.

Hope this article helps clear some things up! I did appreciate Nicklas' article overall and believe he has solid tactics in his coding practices. Just figured I'd help those out looking for a bit more depth on the topic.

Until next time!
