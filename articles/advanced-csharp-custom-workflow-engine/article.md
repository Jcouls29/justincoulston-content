---
title: "Advanced C# — Custom Workflow Engine"
summary: A step-by-step walkthrough of building a custom workflow engine in C# using activity interfaces, a shared context object, a collection builder, and support for conditionals, loops, and exception handling.
category: "C# & .NET"
publishedDate: 2022-02-06
tags:
  - csharp
  - workflow
  - design-patterns
  - interfaces
  - dependency-injection
keyTakeaways:
  - A workflow engine built around a shared context object is the most flexible approach for code-based pipelines, allowing any activity to read and mutate shared state without tight coupling between steps.
  - Wrapping the activity collection in a typed builder interface — rather than raw extension methods on IActivityCollection — enables partial generic inference and a cleaner fluent API when a context type is required.
  - Conditional logic, loops, and exception handling can all be expressed as aggregate activities that hold sub-collections, keeping the runner simple and making the workflow declarative and readable.
  - Lambda-based simple tasks (via Then() extensions) reduce boilerplate for small inline mutations, but anything longer than a few lines should be extracted into a dedicated activity class for readability.
draft: false
---

There are many reasons to create a workflow system. It could be that you have users that want custom flows that they can manage. It could be that you want to create a processing pipeline for data. Or it may be as simple as wanting to write code that is more encapsulated and easier to manage, skipping some of the boilerplate needed in common applications.

All of these are valid use-cases (and there are many, many more). The reality is that everything in life works around "activities." We all deal with processes and procedures at work, at home, and even in our thoughts. Each step, or activity, in these processes, may work the same every time you activate them. So why rewrite the code, or do them manually, when you can create a workflow engine to do it for you.

Now, you cannot just apply this to everything in life (obviously). But you can take complex topics in code and translate them into a workflow engine. The benefit is we can avoid many nested, or configurable, `if-then` statements in our code. Can you imagine writing a dynamic workflow that requires each step to be checked directly in our core code, where there are nested if statements 5 deep? This would be a nightmare (of course, most developers wouldn't do this, to begin with).

But go back and look at code, or services, you've created in maybe a Web API. Could it have been implemented with a workflow of some kind? Does it have pretty complex processing? Are you using parts of that pipeline in multiple places, but it is hardcoded in a way that you'll have to change it in multiple spots?

Workflows also tend to be more readable, long-term. The underlying system itself is usually complex, but utilizing the frameworks are easy. This is where we save our time.

There are a number of great workflow systems already out there. My goal isn't to make these obsolete, or suggest, using this approach below over these frameworks. You should investigate these and determine which one fits your needs best. Each approach the idea of workflow very differently and have unique advantages and disadvantages

- [Workflow Core](https://github.com/danielgerlag/workflow-core): Light weight embeddable workflow engine targeting .NET Standard
- [Elsa Workflows](https://github.com/elsa-workflows/elsa-core): Library that enables workflow execution in any .NET Core application and can be defined in code or using a visual designer.
- [Automatonymous](https://github.com/MassTransit/Automatonymous): A state machine library for .NET that provides fluent syntax for events, states and activities

There are many more than this. These are just ones I've worked with and enjoyed. It'll be your choice to decide if these are overkill for your use-case or have the features you need. But if not, or in doubt, build your own that meets your needs!

Let's walk through it.

> Note: I made mention of workflows and provided a very similar example in [C# Simple Interfaces: Containers](https://blog.devgenius.io/c-simple-interfaces-containers-f2980e76b2bd). This article will be more thorough in explanation and implementations.

## Activities

For any workflow, activities are the key. We want to perform a task, or set of tasks, depending on certain conditions. The complexity isn't necessarily in the activity themselves, but when conditions come into play and how we want developers to interface with the engine's contract.

Let's start by defining some details about activities themselves,

<script src="https://gist.github.com/afa9270aa69f14b2272c9149cd4f3b48.js"></script>

This set of interfaces provides us a very simple framework for activities and will be the core of our workflow system (although it will change later in the article). In fact, if we want, we can add some extensions and then stop, if this is good enough for your needs.

<script src="https://gist.github.com/745991b96e4d7a912b3c4e4d16a703bb.js"></script>

We're only adding a single extension type of `Then` which will simply run 1 activity after another.

#### Processing the Activities

Now that we have our activities, we need to process them. There are a lot of ways to do this, but we'll just create another interface and an implementation for the system.

<script src="https://gist.github.com/5f64f4b30b0192c402065c796b25bc42.js"></script>

And then an implementation,

<script src="https://gist.github.com/75f35e4d82f32804f64d0514a7aa572a.js"></script>

This service will basically go through each activity in a collection and run them back-to-back in a loop, until completion, or until canceled by the activity itself. We will either create an activity based on type or run the provided instance directly. We are not performing any optimizations here, like caching activity creation, but this can be easily added.

> Note: Custom activities can have services injected into them. This is why I use `ActivatorUtilities`. It provides this service for us. However, it also means that the activity must be registered with DI.

Let's not forget we need a basic implementation of `IActivityCollection`.

<script src="https://gist.github.com/9ab5b4b01ece41212889a1a50b686c02.js"></script>

It doesn't have to be much more complex than this, although you may want to customize it a bit. Up to you really.

The last point to make is how we use this simple engine. Let's assume that you have the following defined activities:

- `ProcessDatabaseRecordsActivity`: Goes to the database and processes any available records.
- `DataWarehouseActivity`: Takes current data in the database and pushes to a data warehouse.
- `EmailActivity`: Emails you at the end of the process that it was completed successfully.

You can customize these activities through constructor injection, but no data is being passed between them. We'll address this later.

Let's show a simple usage example,

<script src="https://gist.github.com/aa2413f952b2fd0d5589569286fd9d20.js"></script>

And that's it! We have created a basic workflow engine that will actually work. However, I would be the first to admit this isn't EXACTLY a flexible implementation, yet. There are some issues with this approach:

1. **Data cannot be "shared" between steps**. What if we don't want to have a central source, like a database, that coordinates this information?
1. **We have to create a custom activity for even the simplest of tasks**. Can we find a way to use lambdas for very basic steps, like altering a property in the data?
1. **We have no conditional logic of any kind**. How can we implement `if-then` logic based on data in our workflow?

Let's solve some of these issues.

## Passing Data and The Context Object

I've seen three primary approaches to sharing data between steps in a workflow process:

1. **Shared Context Object**: A single object is created and any mutations to the object is seen by all steps.
1. **Paired Input / Output Matching**: Each step takes a specific input object type and outputs a specific return type. This approach requires steps to match input to output otherwise an impedance mismatch occurs.
1. **Global Step-Based Data**: As each step is completed, it can set output data that can then be accessed by subsequent steps by referencing a prior steps output property. Most useful in UI-based interfaces where steps are well known by the design.

Like anything else, there are trade-offs. **Shared Context Object** is the easiest to implement in a strongly typed language than the others but debugging can be tricky when trying to determine who is mutating the object when issues arise. **Paired Input / Output Matching** is appropriate for ETL applications where data is being transformed and you want to control step building (Think functional programming). It is much more limiting in terms of available use-cases and largely can be resistant to change. **Global Step-Based Data** is great in UI-based interfaces where the user wants to directly access data, knowing their process, but is less useful for dynamic workflows, or code-based workflows.

I am going to take the **Shared Context Object** approach due to its simplicity and varied use cases. I believe it will provide the most value in the majority of developer use-cases.

#### Changes to the Structure and Contracts

We'll need to make some adjustments to our activity definitions to make this work. Let's show the changed interfaces and structures with added context.

<script src="https://gist.github.com/8b901a80633e135902cc71c74032165f.js"></script>

Then the altered extensions,

<script src="https://gist.github.com/ddd20c927744861a97c69df365e5653b.js"></script>

We'll need another implementation for the collection itself,

<script src="https://gist.github.com/a0e67dae3a41e1051206b357ec2d362a.js"></script>

And without going too much further, a sample usage,

<script src="https://gist.github.com/deb76e34af1441f28a38ee5e6e602c3f.js"></script>

Now we can provide a specific context object to our activities, as long as the object implements `IActivityContext`. We can customize it, add functions. Anything. However, each set of steps can only have one context object as it runs (another limitation).

I want to pause here and talk about an issue with this. Before, we didn't have to specify the Context in the extension methods. Instead, they were pretty simple and straight forward. However, extension methods fall short here. It's not really the extension methods themselves, but the nature of method generics.

Without getting too deep in the weeds, when specifying generics, you either need to specify ALL the generic parameters, or none of them (when they can be implied). There isn't any in-between, or partial support. So, in the case where we want to specify the type, we run into a bit of a problem. I really don't like this from a usability standpoint.

How can we make this better?

#### Collection Builder

We're going to have to create a specific builder interface instead.

<script src="https://gist.github.com/ffeed104c5e8c8edb3c085e2abfde086.js"></script>

Ignoring the implementation of this interface for a minute, let's look at its usage now,

<script src="https://gist.github.com/2f4c6ccbed0d753e73bddd81afcafc12.js"></script>

Now, this feels more right to me. This also means that the extension methods we created for `Then` previously are no longer needed. Instead, we're going to have to use the builder to get the syntactic sugar we desire. Which means, that we'll also have to use it for any future extensions as well. So instead of extensions on `IActivityCollection` most will be on `IActivityCollectionBuilder` here on out.

#### Changes to the Runner

We still don't have a runner for this new structure, with the context object. So, let's create that for completeness.

First the interface needs to change,

<script src="https://gist.github.com/8f4415b995b5188214ce2b5fce659758.js"></script>

Then our runner implementation,

<script src="https://gist.github.com/203dbf20a2e4f8edfc51dc0116c1e9ff.js"></script>

Not much has changed, except that we now have to pass in a context object to run the activities. Maybe you want to pass in a creator lambda? Or maybe your context object is a DTO and you want it to be created automatically for you. We can create a couple of extension methods to make this a bit easier.

<script src="https://gist.github.com/a05a29a33ed58dbfef7804ba5493e645.js"></script>

> Again, these are quality-of-life extensions. They are not necessary.

For completeness, I'm going to also provide a base `ActivityContext` that can be inherited. For now, it will only make it so you don't have to reimplement the `Cancel` property. But in your case, there may be more than just a single property.

<script src="https://gist.github.com/f61424b3b985c50159a815e9e1d52f29.js"></script>

## Simple Tasks

Now that we have a core framework down for our workflow engine, maybe we should make some quality-of-life adjustments for simple tasks. For example, after a step, you might want to mutate your context based on some simple logic, or even just set a property. Let's create a couple of activity implementations, and associated extension methods, that will help us out here.

First the activities,

<script src="https://gist.github.com/1b88d955cb851e793b2281b3f41dc001.js"></script>

All these activities provide is a holder for the lambda that will be executed at runtime. There are cases, where you will have some synchronous code, so we added both an `async` version and a `void` version.

To ease their usage, we create associated extension methods,

<script src="https://gist.github.com/095d0ea90d5b60b6e45cea497ae24446.js"></script>

The usage of these methods is pretty straightforward,

<script src="https://gist.github.com/436d81ade82adc278a071df0ee3bd6e0.js"></script>

There will be times when the custom logic will make sense to be defined in this flow. However, anything bigger than a few lines will make this less readable. So be cautious on when you use it. Either way, it does help prevent the need to create a large number of custom activities for simple tasks.

## Conditional Logic, Loops, and Exception Handling

How about conditions in the workflow (sometimes known as decision points)? How about a scenario where we need to run a process multiple times in a row? Or how do we handle exceptions?

Although, these are all unique situations, we're going to use a similar technique for all of them, where we create an aggregate activity and use a sub-builder to extend them.

#### Default Builder

At this point, we haven't really defined our builder. Before it was just assumed. However, we need to define it now.

<script src="https://gist.github.com/955582e7eb40650982aa2ed2aa954e8a.js"></script>

The builder defines a private class for the collection. There are cases where this isn't desired, but we allow this to ensure the implementation is what we want. However, there are cases, where custom collection types might be desirable. In that case, there would be a need for an `IActivityCollectionFactory` of some sort.

Everything else, should be self-explanatory. We add the appropriate methods for `Then`. Then we return the collection during build. I make sure that we replace the backing field, so that the builder cannot alter it anymore after this point.

> Normally, I would make this builder an internal class. This protects its usage and also allows me to expose only the pieces required for the developer to properly use it. Here, it is public for ease of use in our examples. But always try to keep classes internal until they need to be public. EN-CAP-SU-LATION.

#### We're going to run into a little problem…

We're about to hit a road bump. And unfortunately, it will require some refactoring. See, we're going to want to use `IActivityCollection` as the input for our activities. However, it's a collection of `ActivityDescriptor`. This means that the creation of activities, and running them, is going to cause us some pain, especially at runtime. We MAY have access to an `IActivity` to run, but maybe not. We might have a `Type` that we have to create. To create these types, we need `IServiceProvider`.

It's getting complicated.

So maybe, let's create an activity that will do this creation instead, allowing us to only have an `IActivityCollection` that inherits `IList<IActivity>`.

> Sorry, if you feel like we're going back and forth on the implementation. This is the process of refactoring. I'm hoping it provides some level understanding as to my process.

First let's change the collection interface,

<script src="https://gist.github.com/d7dbf8e9805902cb0439a8fd0e139754.js"></script>

We simply change from `ActivityDescriptor<TContext>` to `IActivity<TContext>`. Not any more complicated than that.

We have to alter our builder to have the provider injected. We'll also need to change our collection implementation here (Glad we put it here. Hid the changes from the user, almost).

The builder interface changes,

<script src="https://gist.github.com/eab54bb0716c1fb9acc627e6d6c1f1cd.js"></script>

We added a `StartNew` method that will act as a factory for sub-systems that want to start over with a clean collection.

<script src="https://gist.github.com/d7310d4959072465b99ac553a9934ece.js"></script>

Probably good to show the `ServiceProviderActivity` itself,

<script src="https://gist.github.com/abb9c440d09f8f66ad2d9ae47e6af721.js"></script>

Our runner will also need to be changed,

<script src="https://gist.github.com/e8cbba7293be7b6048eb8e1261425a24.js"></script>

We were able to simplify the class a bit and remove the `GetActivity` portion.

Now when it comes to usage,

<script src="https://gist.github.com/1cc20060ca7874b5fc867bc0496b3625.js"></script>

The biggest change here is the use of the `DefaultActivityCollectionBuilder` and passing in the `IServiceProvider`. At this point, we can move forward with conditionals.

#### Condition Logic

We're going to create a very simple `If-Then-Else` conditional logic activity. We will optionally be able to provide just the `If-Then` logic or the `If-Then-Else` in the same activity.

<script src="https://gist.github.com/e72d408c983bca73da8290e550e7b65e.js"></script>

What are we doing? We're passing in two sets of activities and a condition. The condition is what will determine which collection to run. The collections are then run appropriately based on said condition.

Where are those quality-of-life improvements, you say? Those are next!

<script src="https://gist.github.com/4d9f34e36b04247628f87739b24371de.js"></script>

We're using the `Then` methods to create the conditional activities. The question becomes what this looks like to the programmer now? Let's look at usage,

<script src="https://gist.github.com/a315041a0cdb13d47f20036fa7038d18.js"></script>

After we have processed the data, we will check if it was successful with a flag that is set. If we are successful, then we will run a success email activity, otherwise, a failure.

#### Loops

I won't go through all possible implementations of various loops, but I will show one, a Repeat Loop.

The goal with the repeat activity is to run the same set of activities multiple times either with a static count or based on a property on the context.

<script src="https://gist.github.com/ac05aa3dd4fae842924bd003a3dcbaa0.js"></script>

Simple enough. We can pass in a function that will be calculated based on context, or we can pass in the count directly. Let's create those extensions,

<script src="https://gist.github.com/043c56dd7f9d7300f40b6678c9151fb6.js"></script>

And finally, it's usage,

<script src="https://gist.github.com/57bf1a2a47dc5af3163dc4bbcfcb26f3.js"></script>

I would encourage you to add whatever other extensions that may be useful. Maybe you want to just pass in a single activity. For instance, you can have an extension that takes `count` and `IActivity<TContext>` instead.

#### Exception Handling

The last common issue that can arise is the handling of exceptions. You can do this in the activities yourself. But what if you want to alter course based on a specific exception? Similar to the last two approaches, we'll create a special activity for this.

Let's show the activity first,

<script src="https://gist.github.com/69b873ffc7d98cc35e9272a63250604f.js"></script>

I kept this somewhat simple. You pass in a set of activities for the `try` clause and optionally the `finally` clause. For the `catch` clause we use a list of `Type` and activity collections. When catching the exception, we want to use similar logic as the true `try-catch`. So, we find the first exception in the collection that is assignable. If none, then we re-throw.

> Note, best practice would say that we should have an interface for the `CatchActivityCollection`. I left this out for brevity.

Let's look at the extensions,

<script src="https://gist.github.com/7b0214fa05a0ed82906bdc4ba7a7b0d5.js"></script>

And finally usage,

<script src="https://gist.github.com/707f651752ddfe030a4ca69e81c1a2e5.js"></script>

I will admit, from a readability standpoint, this could be better. I think we could find some ways to improve it, but hopefully you get the idea.

## Conclusion and Sign-Off

Well, that sure was a lot of code. We created a simple workflow engine that can be used for endless use-cases. However, it does have some downsides,

1. **Saving / Loading**: It would be difficult to save / load this structure dynamically from a file or database. Everything is created as instances and not descriptors.
1. **Configuration Limitations**: Configuration is limited to what can be injected into the constructor of activities. Maybe this is ok, but dynamically changing configuration could be a useful feature.
1. **Lack of State Machine**: There isn't really a state machine being used here. You could theoretically run a state machine with the context object, but what if you don't want everything to run at once? What if you want to continue a workflow later?

This is by no means a perfect workflow system. This version can be used in many ways, but not in every situation. At the very least I hope you learned some different techniques that you can implement when designing your next system.

If you have any thoughts or suggestions, please comment below. And be sure to **follow me** on medium, as I release new topics all the time!

Until Next Time!
