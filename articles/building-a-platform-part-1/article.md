---
title: "Building a Platform: Part 1"
summary: Covers five types of generalization in software — storage, model, service, contract, and process — and when to apply a bottom-up versus top-down approach to building abstractions.
category: "Software Architecture"
publishedDate: 2022-02-03
tags:
  - architecture
  - design-patterns
  - interfaces
  - csharp
keyTakeaways:
  - There are five distinct types of generalization — storage, model, service, contract, and process — each with its own scope and appropriate use case.
  - The top-down approach (concrete first, generalize second) is preferred in roughly 80% of cases because it avoids premature optimization and exposes unknown requirements early.
  - Bottom-up generalization works best when rules and domain constraints are well-understood up front, such as in regulated industries like finance or banking.
  - "A hybrid approach is valid: generalize early the parts you fully understand, and work concretely on the rest before abstracting later."
draft: false
relatedArticles:
  - building-a-platform-part-0
  - building-a-platform-part-2
  - building-a-platform-part-3
  - building-a-platform-part-4
---

## Series Table of Contents

[Part 0: Standard Abstraction Layers and Defining the Platform](https://blog.devgenius.io/building-a-platform-part-0-e2a8a5af62bb)
**Part 1: Types and Process of Generalization**
[Part 2: The Architecture of Your Platform](https://justin-coulston.medium.com/building-a-platform-part-2-cc8998716246)
[Part 3: Designing Great Contracts First](https://justin-coulston.medium.com/building-a-platform-part-3-7d63d2a3d9d9)
[Part 4: Implement and Test Contracts](https://justin-coulston.medium.com/building-a-platform-part-4-91fa2173c1b7)
Part 5: Continuous Integration Early Steps
Part 6: Evolving the Platform
Part 7: Dreaded Documentation Details
Part 8: Distribution of the Platform

## Introduction

In Part 0 of this series, "Building a Platform" we defined a platform and the layers of abstraction. We discussed the different layers and the importance of creating abstractions to ease the work required by application and system developers.

Part 1, "Process of Generalization" will introduce the process of creating generalizations, and thus, abstractions for your software. All examples will be developed in the .NET Core Framework with C# 10. By the end of this article, it is my hope that you will understand an easier approach to generalizing your code and forming abstractions.

## Types of Generalization

The first thing we should understand, are the various categories of generalizations. By understanding what each of these covers, we can more appropriately apply abstractions. I have shown my top 5 types of generalizations, but there are certainly more.

1. **Storage Generalization**: The underlying storage structure is generalized to work with multiple entities, types, or elements in an attempt to avoid major changes to the structure.
1. **Model Generalization**: The business models are generalized to allow for common, cross-cutting concerns to be reused regardless of data in a way to simplify service usage.
1. **Service Generalization**: Creating a common setup of services that do not require alteration when multiple model types are utilized. Often worked in conjunction with model generalization.
1. **Contract Generalization**: The generalization of contracts between external systems and internal mechanisms. Most commonly used to simplify discovery and/or schema changes.
1. **Process Generalization**: An algorithm or workflow generalization that is common across services, or domain boundaries. Often worked in conjunction with model generalization.

These types are important to understanding what you can / cannot do with generalization. Each type forms scope of changes that should not impact other aspects of the system. However, there are times that some generalizations naturally lend themselves to multiple types.

Let's walk through each type. We will show two code examples in each type. One will be the non-generalized design and the other will be a more general approach. Detailed articles, explaining these types, will be written later in the series.

**Storage Generalization**

This type is about storing data. It could be in files, SQL databases, No-SQL databases, or any other form of storage. With any architecture, there are trade-offs to any design that is used. In some cases, it may be performed. In others, it may be complex.

In the following example, we're showing an example of multiple entities separated, then generalized for future growth.

*Basic Implementation*

<script src="https://gist.github.com/c1fa4444d4a8432b139f00acaa2c1942.js"></script>

*Generalized Implementation*

<script src="https://gist.github.com/6950e33097f8e245fb902fea26db2b6d.js"></script>

> We'll discuss more about the advantages and disadvantages to this example of generalization in a later entry in the series.

**Model Generalization**

When thinking of business models, we tend to err on the side of separation. And in most cases, this is perfectly valid. However, as systems become more and more complex, it is important to also ensure efficiency in the system. Model generalization is a means of sharing data and methods for common types of entities in a system.

In the following we're showing business models in the system and their generalized form.

*Basic Implementation*

<script src="https://gist.github.com/913135a160e6620a75bd676f03cc180d.js"></script>

*Generalized Implementation*

<script src="https://gist.github.com/a3bf52783a081678b4703e156975d835.js"></script>

Although no methods were included in this example, we can just as easily abstract functionality as well.

> We'll discuss more about the advantages and disadvantages to this example of generalization in a later entry in the series.

**Service Generalization**

When we start having common, cross-cutting concerns or simply common patterns in our system, it is advantageous to create generalizations of these topics. In the case of Service Generalization, it is most noticeable when we have similar model structures. We utilize this type when we want to operate on models, requests, or queries in the same manner across the system.

In the following we're showing a business service in the system and its generalized form. However, the details of the implemented methods will be omitted for brevity.

*Basic Implementation*

<script src="https://gist.github.com/10e8c58d001de0a09b0f9df9b0224682.js"></script>

*Generalized Implementation*

<script src="https://gist.github.com/5d7e333fcf839ddd51dd808f6a6701e1.js"></script>

> We'll discuss more about the advantages and disadvantages to this example of generalization in a later entry in the series.

**Contract Generalization**

Contracts are the basis of interoperability between systems. In the case of contract generalization, we're not really talking about internal contracts (like interfaces, abstract classes, etc.). Instead, we're referring to external interactions (like web services, data streams, file formats, etc.). When we form a contract, the complexity of interacting with the implementation of the contract increases with each different aspect. The fact that it affects external parties, makes this type unique from the general service type. This is probably better shown through an example.

In this example, we're going to show the HTTP contracts formed for a sample service and a generalized form of it that might be found easier to access.

*Basic Implementation*

<script src="https://gist.github.com/6721fa0af973324ba44b39d656899f53.js"></script>

*Generalized Implementation*

<script src="https://gist.github.com/db95331ab44a7bb66736c6191314a67e.js"></script>

In this example, we can see that we make it so the caller only needs to care about a single endpoint to create an entity in the system. However, this doesn't necessarily make it a better implementation, as we had to add a `type` field to make this work. The caller doesn't gain a lot of value here. However, the point of the example is to show the generalization of the contract.

> We'll discuss more about the advantages and disadvantages to this example of generalization in a later entry in the series.

**Process Generalization**

Last on the list is process generalization. This is more around the generalization of common sets of functionality as related to algorithms or workflows. Of the five types, this is the most open-ended.

For example, you may have created a number of background services for a particular system. However, you find that all of them have a `Start` method and a `Stop` method. More than that, you realize that each time you add a background service, you rebuild the application.

One way to generalize this approach is to create an interface for these background services.

<script src="https://gist.github.com/8884686c73eca97bcfd08fc8b5e540b5.js"></script>

Each background service would implement this interface. However, we would rather not build the core application each time. To fix this issue, we would need to create a plugin system.

A plugin system would allow for the developer to drop a set of dependencies into a directory and force the application to load these plugins in automatically. After a restart of the application, the service's `Start()` the method would be called.

Another example may be a system to generalize complex workflow logic that changes from customer to customer. We may want to send customers to different screens based on their configuration setup. By utilizing a builder-style pattern and the plugin-based architecture, we can create a highly generalized approach to altering page redirects.

<script src="https://gist.github.com/50283c39d9a5ee1aa1c0dd18fb5c5288.js"></script>

At first glance, this may or may not seem like a win to you. However, if you have 10 customers and all this logic lives in your controllers or business services, you run the risk of trying to manage very complex rules. Try to imagine the layers of `if-else` or `switch` statements needed here. Instead, separating out these differences into a workflow makes the management simpler at the customer delivery code site. However, the complexity of building this system can cancel out gains if the management of the underlying generalization is heavy, meaning you find yourself changing the sub-system often.

**Type Summary**

We will use these types extensively in this series to walk through ways to appropriately generalize software and create a platform layer that is re-usable and efficient.

Let's move on to the general approaches.

## Two Approaches to Generalizing

In my view, there are two primary approaches that can be used to build an abstraction:

1. **[Bottom-Up]** Create the generalization up front
1. **[Top-Down]** Create the models first, generalize second

The Bottom-Up approach is akin to creating a data model first, then data transfer objects, then the classes that will use them, and finally the UI layer for the user (or web api).

The Top-Down approach is the opposite. It is where we create the UI layer for the user, build service classes and business models, then the data models.

Both of these approaches can be used successfully, however, they are convenient in different scenarios.

**Bottom-Up**

We will typically start from the bottom when we have a very strong understanding of the domain, its limitations, and what the future holds for us. Or we would use this approach when the end result is fluid and can vary widely (letting the design form naturally). We see this commonly in software that is under more strict regulation and is better defined: Finance / Banking, Government Services, or Real Estate Systems. Or in systems that are "bleeding-edge" and the domain is flexible.

These systems may already have created some form of abstraction that we can directly utilize. For example, let's talk about a simple banking system:

<script src="https://gist.github.com/13c36b47efce6bbb685668e5fc01bdcd.js"></script>

Of course, this is a very simple example. A real banking system is much more complex, in terms of its data model. But this should convey the point. Most services within a banking system works off a sort of `BankAccount`. Even derived services like Certificate of Deposits can be considered `SavingsAccount` but with special interest rates and rules.

The point of this example is to say that in some cases, rules are well understood, and the abstractions are fairly obvious. We are not necessarily attempting to be innovative here, just efficient. We create the generalization based on the understood business rules early on. We do this early, in order to not waste time later on going backward, as is the case with the Top-Down approach.

**Top-Down**

The Top-Down approach is more geared towards unknowns. If you've developed with a team for any length of time, you find this to be the most common problem with requirements, the unknown-unknowns. We use the Top-Down approach to first create naïve software classes and methods and once we have worked out the requirements, we generalize afterwards. This is most useful in greenfield projects of unknown scope but where requirements are driven by external customers or stakeholders.

The downside of this approach is the lack of efficiency. It requires fully building out your methods and classes before working towards generalization. However, the biggest advantage is not prematurely optimizing your architecture, just to find that you missed a key element causing a full rework (or lingering technical debt).

Let's look at the example again in the **Model Generalization** section. Let's show this in two stages,

*Basic Implementation*

<script src="https://gist.github.com/913135a160e6620a75bd676f03cc180d.js"></script>

*Generalized Implementation*

<script src="https://gist.github.com/a3bf52783a081678b4703e156975d835.js"></script>

At first glance, this may seem like it couples these classes together. And it does. But this is purposeful. We can create a more efficient system of working all the way down the stack by allowing this abstraction. Without going into too much detail here, we can simplify our SQL code, processing code, and model mappings by creating a common base class.

> See article [Advanced C# — Common SQL Table Structures](https://blog.devgenius.io/advanced-c-common-sql-table-structures-bb7f7149b887) for an example.

But the essence of the example is that we started with concrete classes first and then created an abstraction from them. This is the Top-Down approach at its core.

> **Hybrid Approach**

> I'm not one to work in absolutes. Of course, there are hybrid models where you can abstract early the portions you fully understand (but please be definitive). Then work out the rest of the details in a concrete manner and work backwards later.

## Pragmatic Approach

The ability to abstract items, boils down to one key idea: **Understanding the Requirements**. In order to properly abstract, or generalize, any concept we need to see the end state first. The reason for the discussion on Top-Down and Bottom-Up was to throw this into focus.

Top-Down forces us to think about the end result first, working out the requirements and concepts immediately. Because we are building the "concrete" at the beginning, many unknowns are found early. But it feels inefficient to work this way, and in a certain sense it is (at least upfront).

Bottom-Up forces us to imagine our end result, using our mind's eye, working out the requirements conceptually. This has some severe limitations to creating strong abstractions. It is very easy to miss key aspects of the end result at the end. And can subsequently force us to rework abstractions late in the game, negatively impacting our architecture.

The real value in this approach is when we desire for our design to not be influenced by the implementation. The scenario usually works best when the end result is fluid (like a RESTful contract that is yet to be determined). We can build from the bottom starting with strict requirements pertaining to performance, security, and data structure, then let the end result morph accordingly.

Neither approach is right or wrong. However, I do believe that in most scenarios, a Top-Down approach is preferred (~80% of the time). Using the Bottom-Up is too limited and risky to be used in the majority of cases, especially since we are typically working with clients and stakeholders with very specific needs.

It's hard to say the cliché, "it depends" but that's the reality. In this series, we will be utilizing both approaches, and hopefully guiding folks into understanding when and why to pick one approach over the other.

## Conclusion and Sign Off

In this article we focused on different Types of Generalizations and the Process of creating them. Conceptually, the idea isn't difficult. But it can be difficult to put into practice correctly. In the moment, we're focused on solving a problem, usually for a client or stakeholder. However, it is the responsibility of the developer and architect to augment the original need with additional context on including concepts like reuse and maintainability. We must ensure that with each requirement, we see the abstractions and generalizations that can be applied as we go. Without active management, reworking a system can be difficult. I hope to help provide guidance in this area.

If you have any thoughts or suggestions, please comment below. And be sure to **follow me** on medium, as I release new topics all the time!

Until next time!
