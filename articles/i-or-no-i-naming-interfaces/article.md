---
title: I or No I, and the Naming of Interfaces
summary: An opinion on why the "I" prefix convention for C# interfaces exists, where it causes harm, and what better naming discipline actually looks like in practice.
category: "C# & .NET"
publishedDate: 2025-08-26
tags:
  - csharp
  - interfaces
  - dotnet
  - code-quality
keyTakeaways:
  - The "I" prefix on C# interfaces is a legacy of COM-era Hungarian notation that persists mainly to distinguish the interface from a same-named concrete class in the .NET BCL itself.
  - Dropping the "I" without renaming implementations creates visual ambiguity between inherited classes and implemented interfaces—any solution must address both sides of the colon.
  - The real naming failure is the `IFoo` / `Foo` pattern where the implementation is just the interface name minus the prefix; this signals the interface has no meaningful abstraction and should probably not exist at all.
  - "Prefer naming the interface after its role and the implementation after its technology: `EmailManager` as the contract, `SendGridEmailManager` as the concrete, which forces developers to think about why multiple implementations might exist."
draft: false
---

(An opinion on the correct answer)

![Photo by Clarissa Watson on Unsplash](https://cdn-images-1.medium.com/max/800/0*03pxZdLXPunnsJz4)
*Photo by [Clarissa Watson](https://unsplash.com/@issaphotography?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

## The Debate

> Should interfaces have the letter "I" in front of them? Or should they not?

That's it.

Well, I guess that's not it. Not really. Why is this even a thing?

Really, it's only a thing because Microsoft started doing it this way, an almost Hungarian notion for abstractions. But is this really the reason to name interfaces prefixing with the letter "I"?

## Simple Example

Let's look at a basic example of an interface we may want to use in our software. (Remember, an interface is simply a code contract, not actual running code).

```csharp
public interface IWebhookRepository
{
  Task ReceiveWebhookAsync(EndpointId endpoint, WebhookData data);
}
```

Now this is a very easy to understand interface. And even if it wasn't, it doesn't matter. We care about the naming of it. We have it prefixed with the letter "I" as is convention.

So where would this interface be used?

We'd probably have an implementation like so,

```csharp
internal class BlobStorageWebhookRepository : 
  BlobStorageBase, IWebhookRepository
{
  public Task ReceiveWebhookAsync(EndpointId endpoint, WebhookData data)
  {
    // ...
  }
}
```

When we implemented this we inherit from a base class, and then implement the interface.

But what if we left off the "I" in the interface. What would it look like?

```csharp
internal class BlobStorageWebhookRepository :
  WebhookRepository, BlobStorageBase { }
```

Uh huh… yep… that's what it would look like… Maybe some other examples might help,

```csharp
public class Stack<T> :
  Enumerable<T>, Collection, ReadOnlyCollection<T> { }
```

```csharp
public class SortedDictionary<TKey, TValue> : 
  Dictionary<TKey, TValue>, Dictionary, ReadOnlyDictionary<TKey, TValue> 
  where TKey : notnull { }
```

```csharp
public class StreamWriter : TextWriter { }
```

Of course, these are real examples from the .NET runtime itself where I removed the "I" from the interfaces. Now tell me, which of these inherited-, or implemented-, aspects are interfaces and which are classes?

If you have any experience with .NET, you'll probably know right away that TextWriter is the only class being inherited by another. But there really is no way to tell, not without prior knowledge (And the knowledge of only being allowed to inherit one class at a time).

## So Many Questions

Let's ask ourselves a few questions then.

1. Do we really need to know that any of these are interfaces or classes?

> Well. No. Not really. Why does it matter exactly? The reality is that our class is required to follow a contract when an interface is added to the list. This means it will be one or more members on the class. If we're inheriting from a class, its members are also a part of the targeting class and so, from the outside, it looks the same from the caller's perspective.

2. Does this happen often (inheriting and interfacing at the same time)?

> Well, not a lot in the .NET runtime. It seems they are very careful about their inheritance structure and if there is a class being inherited, they don't typically add an interface too. At least not at the lowest levels. But if it doesn't inherit, they will tend to use ONLY interfaces. So, if this happens, it'll be in your own code base (or some open source code base) you'll see this type of structure. It's not usually recommend due to this very confusion. This is why my webhook example really doesn't work well because it is ambiguous.

3. What other frameworks or languages have this convention?

> None (that I'm aware of). This seems to be entirely a .NET thing. Even Microsoft's Typescript doesn't use the "I" prefix. Instead, they go the route of interfaces being named like classes in Pascal Casing.

4. Where would having the "I" be useful then?

> Well, that's the question. I could imagine a world where no IDE(s) exist anymore and when we add an interface to the class, without knowing it is an interface, you may struggle to know that you MUST implement any functions. Notepad doesn't really help in this area. But that's not really an issue, right?

5. Is there maybe a reason for a large framework to structure it this way?

> Now that's a better question. Why would they do this? Why would the .NET team introduce this prefix of "I" in the names of interfaces? They don't do that for enumerations (`enum`) or structures (`struct`). Why do it for interfaces?

> Well, I would say it is because they need to do this for easier access from a programmers perspective. I mean look at `IDictionary<TKey, TValue>` and `Dictionary<TKey, TValue>`. The class is named the same but without the "I". This is because they needed a way to describe the contract separately (avoid the name clashing). And unfortunately, they both represent the same thing, a dictionary. We use `Dictionary<TKey, TValue>` as the default in many situations but occasionally need to use `ConcurrentDictionary<TKey, TValue>` or even create our own.

> I guess they could have named it `DefaultDictionary` or `InMemoryDictionary` or any other type thing, but for us, it is simpler to remember "Dictionary" and I believe this was a trade-off for usability of the framework. But I suspect that a more likely answer is that this convention stems back to the COM era of Windows and hungarian notation was a bit more popular back in the day and it kind of just stuck.

## Where does this leave us?

Honestly. I write my interface with an "I". But it's not because I like it. It's because it is convention in .NET languages. I think, semantically, interfaces without the "I" hold more power. For instance, when I see something like this,

```csharp
public interface IEmailManager { }
public class EmailManager : IEmailManager { }
```

it really gets under my skin. Why? What purpose then is there to having the interface at all? Ok, maybe you're injecting a `SendGridClient` into this `EmailManager` and it is sending things out the door. You'll want to mock it in some testing framework and will need to replace it somehow. Thus the interface. Well, that's fine. Just rename it to `SendGridEmailManager`. I'm not against having the interface, but when I see someone name an implementation after the interface, it "feels" like it the only one around. Rather, I would like to see developers be more specific about an implementation's usage. For this reason, I would prefer the interface to be `EmailManager` and the implementation to be `SendGridEmailManager`. In that way, when you create the interface, you're forced to think about implementations and expressing them appropriately.

But alas, that is not the world we live in. Instead, developers will create an interface called `IRandomizerService` and turn around to create a `RandomizerService`. Then one file down in the same folder have `IGuidGenerator` and `GuidGenerator`. And there is never another implementation. Just get rid of the interface. There would literally be no reason to keep it around.

I digress. I've always felt the right convention is to drop the "I" and go the way of all the other conventions out there, forcing contracts to be named appropriately towards their purpose. Like you would when you call your `Cat` your `Pet`. `Pet` is a contract (so-to-speak) that the cat holds to. A `PolarBear` isn't typically a `Pet` though. He's a `WildAnimal` with a `SurviveWithoutWaterBowl()` function defined on it. A wild animal is an abstraction to a many types of animals out there and it would be better to have an interface. I mean a polar bear will probably end up surviving very differently from that miscreant vole digging through my yard (but hopefully not for long).

## Conclusion

It is a pretty trivial thing to be honest. And this was kind of a shallow article (yes, I'm aware). But naming things ARE important. It isn't easy to name things, but it is crucial. So, even if we keep the "I" in place for the foreseeable future (and you're not a framework developer), could you at the very least make the implementation name not follow this algorithm please:

`string className = interfaceName.TrimStart('I');`

Thank you. Amen. And good night.
