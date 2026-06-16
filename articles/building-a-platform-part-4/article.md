---
title: "Building a Platform: Part 4"
summary: How to implement and test platform contracts using test-first development, in-memory implementations, and a single shared test suite run against multiple concrete implementations.
category: "Software Architecture"
publishedDate: 2022-02-20
tags:
  - architecture
  - testing
  - interfaces
  - design-patterns
keyTakeaways:
  - Test contracts and complex sub-systems, not every internal class — running the same test suite against each implementation proves the contract holds.
  - Start with the simplest possible implementation (in-memory) to flush out design issues before committing to a database or external dependency.
  - Avoid mocking frameworks; implementing your own fakes reveals interface bloat faster than any linter.
  - Tests are just another view of your system and deserve the same refactoring discipline as production code.
draft: false
relatedArticles:
  - building-a-platform-part-0
  - building-a-platform-part-1
  - building-a-platform-part-2
  - building-a-platform-part-3
---

*Implement and Test Contracts*

![Photo by Ildefonso Polo on Unsplash](https://cdn-images-1.medium.com/max/800/0*rkKjNyDRryDAg5ko)
*Photo by [Ildefonso Polo](https://unsplash.com/@i_m_polo?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

*This post may contain affiliate links and I may earn a small commission when you click on the links at no additional cost to you. As an Amazon Affiliate I earn from qualifying purchases.*

## Series Table of Contents

[Part 0: Standard Abstraction Layers and Defining the Platform](https://blog.devgenius.io/building-a-platform-part-0-e2a8a5af62bb)
[Part 1: Types and Process of Generalization](https://blog.devgenius.io/building-a-platform-part-1-cf543658bfe3)
[Part 2: The Architecture of Your Platform](https://blog.devgenius.io/building-a-platform-part-2-cc8998716246)
[Part 3: Designing Great Contracts First](https://blog.devgenius.io/building-a-platform-part-3-7d63d2a3d9d9)
**Part 4: Implement and Test Contracts**
Part 5: Continuous Integration Early Steps
Part 6: Evolving the Platform
Part 7: Dreaded Documentation Details
Part 8: Distribution of the Platform

## Introduction

In Part 3, we walked through designing out our contracts and the type of questions we should be asking. We talked about common contract types and tools to help you work out the details. The design is focused on return on investment (ROI) and the total cost of building functionality.

In Part 4, we're going to walk through the idea of implementing and testing our contracts. We won't walk through testing all types, but hopefully, we can provide a solid framework to making a stable, quality platform with the approaches in this article.

Remember,

> Code **Contracts** are formal, or informal, agreements between two entities in a software system, or external actors.

We will be showing all examples in C#.NET 6, as this is my preferred language. We will be using xUnit.NET for our testing framework. However, conceptually, your language and testing framework of choice will work just the same.

Let's get started.

## Test-First Development

TDD, or Test-Driven Development is the practice of writing tests first with the following cycle,

![TDD cycle](https://cdn-images-1.medium.com/max/800/0*_t46o5xrxCeJxr7t.jpg)
*[blog.cleancoder.com](https://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html)*

1. **Red**: Create a unit test that fails
2. **Green**: Write production code that makes that test pass.
3. **Refactor**: Clean up the mess you just made.

And do not forget the 3 laws of TDD ([Clean Coder](https://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html)),

1. You must write a failing test before you write any production code.
2. You must not write more of a test than is sufficient to fail (or fail to compile).
3. You must not write more production code than is sufficient to make the currently failing test pass.

In general, this approach works very well. But it only works well when you're focused on solidified requirements. If the requirements are not well defined, this approach can be frustrating. However, I think this to be true for any development when requirements are not organized well. It isn't necessarily TDD-specific.

This approach is typically used from a unit testing standpoint, where all code is written should have an associated test. This is the portion I disagree with more than anything. Instead, we should focus on the behavior of our application (like BDD) when we perform testing. And it should apply to these aspects of the platform:

1. **Externally Defined Contracts**: Web Service API, Well-Defined Exposed Interfaces within libraries, etc.
2. **Complex Sub-Systems**: Any system in the platform that may be overly complex with logic or interconnections that can be brittle due to technical limitations.

We should NOT write tests for every class that we have, nor every implementation of a contract we have. Instead, it should be only against the contract itself. We should run the same tests against each implementation to ensure they hold to the contract appropriately. This is the whole point.

> We're trying to focus on the contract itself and not on how it is implemented. This is a key distinction to covering your platform with appropriate tests.

It is important to understand that contract testing is a higher-level set of tests. For this reason, the depth of your test cases will determine how well your tests will cover you. There are cases when some contract tests will be considered "unit tests" and some cases where they will be considered "integration tests." It does not actually matter from the perspective of what we should be testing, just the potential complexity of setting up the tests.

> **Note**: Any tests that are brittle, in that they have to change often, should be avoided. These are typically contracts like UI or dynamic system apis. I'm not saying that you shouldn't write any. But the more you need to alter your tests, the less valuable they are. Minimize these types of tests from your automation, otherwise you'll find yourself maintaining tests more often than you should.

#### Test Contract Behavior

When writing these contracts, we are testing behavior of the contracts themselves. Where do we get the behaviors?

From the requirements of course.

But what if you don't have any requirements? Or requirements that are not well-defined. You have a couple options:

1. Go back to the source (client, BA, etc) and solidify them.
2. Make assumptions and label them clearly.

We all make assumptions when writing code. For instance, when we're putting a SQL table together, how often have you put a size on a `char` column arbitrarily?

"Hmm, I think the first name won't ever be bigger than 64 characters."

These are assumptions that we make. However, we don't always test these assumptions. Instead, when the business doesn't know the answer, and you are forced to make a decision, I recommend all of these going into a test and label it as an assumption. This will help you in the future, if issues arise, to locate any assumptions you may have made that may have been incorrect.

So, let's say we know our requirements. What exactly should we be testing? Each type of contract has different categories you'll care about but here are some of the contract behaviors you should take into account,

- **Does it work?** Are all the core functional (happy path) requirements met?
- **Are all the use cases covered?**
- **What types of errors (or exceptions) are expected?** How will they be returned to the caller?
- **What are the pre-conditions** of the data going into the request?
- **What are the post-conditions** of the data coming from the response?
- **How is the state of the system expected to change?**
- **Are the extremes of the parameters appropriately checked?**
- **Are there any workarounds that a user / caller can perform to break your functionality?** Are there tests to detect this?

These are somewhat generalized, but asking these questions are important when defining the contract agreements and assumptions you are making within the software. For each answer you provide, there should be an individual test.

#### Recommendation: Make the First Implementation the Simplest

Unfortunately, you cannot really test a contract. By definition, it is only an agreement about an implementation, like a standards document. Instead, you will always test an implementation of the contract. For this reason, I recommend that you always make your first implementation the simplest it can be.

For instance,

Let's say you're building a library for your platform. You have designed an interface for use by other developers. You know what they will expect from an API standpoint, but you haven't implemented the interface yet.

What should be the first implementation?

Well, this depends on the use case of the service,

- **Data Repository Service**: Create an In-Memory implementation that uses an internal collection to store records.
  *See [In-Memory Repositories: A Forgotten Design Tool](https://medium.com/dev-genius/in-memory-repositories-a-forgotten-design-tool-1613151b8491) for deeper examples*
- **Hardware Service**: Create an In-Memory emulator of the hardware device that can be setup during testing to "mock" up a real hardware device.
- **Business Logic Service**: Create a "Default" service that implements the contract appropriately. This one may end up as the final implementation.

These are just a few, but go for something that can be run locally, without actually having to have any services running, external dependencies, or hardware attached. Emulate the behavior if you must. You will find this to be highly valuable for quick testing, as well as providing a thorough understanding of what will be required in other implementations.

#### Side Note: Mock Frameworks

These types of implementations can completely replace mocking frameworks like [Moq](https://github.com/moq/moq).

In fact, I actually never use mocking frameworks. I think it can allow you to get lazy in your thinking. You mostly have to re-implement your logic in your tests, which to me, defeats the purpose. I will always recommend avoiding them.

Instead, I recommend creating fakes yourself. Try implementing your own interfaces again. You may actually get frustrated with how many methods you have to implement to make your fake work. This is a clear sign that your design is not optimized.

For instance, let's say you have an interface like the following,

<script src="https://gist.github.com/311441697f28369abf82a531a42e3462.js"></script>

I've asked you to re-implement this contract with a separate in-memory implementation. However, you really don't want to have to write all this code. This is where frameworks like Moq help. They make it so you only have to implement the one you care about in the test. But this is a lazy approach to testing this contract.

The fact that you don't want to do it because it will take longer means that the interface is probably too large. What if the contract looks more like the following?

<script src="https://gist.github.com/f8a20c51a3ffaa09b06a37a02a59d506.js"></script>

In this case, we only have to implement 4 methods instead of 7. This interface is also much more flexible. However, the testing of this contract doesn't actually change. For example, the tests for `FindByName` and `FindBySku` are now combined within the `Find` method. So, the tests will be the same, but calling a different interface method.

The value of re-implementing your interfaces is that when you get frustrated with the design, you feel it, and thus know that something needs to change.

#### Refactor Everything

Tests offer a number of valuable advantages over not having any tests at all,

- **Documentation** for requirements and assumptions locally to the code
- Provides automated **validation** of business requirements
- Builds **confidence** during architectural or code refactoring

Tests in essence provide documentation, validation and confidence. And with these, we will feel comfortable refactoring our codebase.

Now when I say refactor, I do not mean changing the contracts. No, when we refactor, we alter architecture and code *behind* the contract. This could be implementations that are publicly available or services that are hidden from the clients. We can definitely change interfaces, just not external contracts, as this will break our clients. Instead, changing interfaces should be focused around internal, or private, contracts.

But we really shouldn't limit our refactors to only production code. We should be refactoring our tests as well. I'm not suggesting changing the test cases, themselves, but writing clean code and creating any abstractions within the test libraries, as needed.

> Remember tests are just another view on your system, just like controllers, UI elements, CLI Arguments or calling services.

Refactoring your tests are just as important as refactoring your primary codebase.

## Example Testing of Implementations

I think it is time that we start walking through an example. Due to the nature of TDD, and the step-by-step iterative process it requires, I will not go through the stages of Red-Green-Refactor here, but instead, show you the finished code.

#### Model Contract Example — Parsing Phone Numbers

Let's say that we have a public model that we're exposing that will parse phone numbers. The "contract" here is that we want to parse all possible phone numbers into a standard format. Here are some of the requirements,

- Must optionally accept country code with 1 to 3 digits
- If a country code exists, the number must be prefixed with a `+` sign
- Must optionally accept area code with exactly 3 digits
- If country code exists, area code is required
- Must require locale code with exactly 3 digits
- Must require number with exactly 4 digits
- The parse function should accept a single `string` value as input and return a segmented phone number into `CountryCode`, `AreaCode`, `Locale`, and `Number`.

So here are our tests for this contract,

<script src="https://gist.github.com/8caa57978002dbaf50a954b27b8868ad.js"></script>

Now we could separate out the use-cases into buckets as well. However, for this case, it would be duplicated code that I don't find necessary. Always try to be pragmatic.

Now our implementation,

<script src="https://gist.github.com/84929637dd4a2b4020cf16448d67403d.js"></script>

Not the most elegant solution, but the tests pass. If you wanted to, you could easily change this method and hit play each time to verify the contract is still satisfied.

We didn't make any assumptions with the requirements here. But you could create some if you wished. For instance, what happens if a `null` is passed in? We don't have that case covered. This would be an assumption for us as we were not presented with that use case in the contract.

#### Repository Contract Example

This example was taken from the article [In-Memory Repositories: A Forgotten Design Tool](https://medium.com/dev-genius/in-memory-repositories-a-forgotten-design-tool-1613151b8491). Let's look at the contract,

<script src="https://gist.github.com/Jcouls29/813dc2660b493389f3edf8cc5ee4b49a#file-iproductrepository-cs.js"></script>

The contract is fairly simple. A standard repository has CRUD functions. Nothing more. Now we have a few requirements in addition to this interface contract,

- Every product must have a name (cannot be blank)
- Product names can be duplicated
- The maximum name of a product is 64 characters
- Products can optionally have a description up to 256 characters

Now here is our contract testing for the `AddAsync` method only,

<script src="https://gist.github.com/Jcouls29/37498a5cb14a6498448f530b51fdde3b#file-inmemoryproductrepository_tests-cs.js"></script>

And the implementation,

<script src="https://gist.github.com/Jcouls29/570e10484491620bb80472881c3362e1#file-inmemoryproductrepository-cs.js"></script>

Here you can see that we implement the `AddAsync` function completely with all preconditions to ensure compatibility with our contract.

I would typically create this in-memory version first to ensure that I completely work out all of the kinks in the logic. Once this is complete, I can then go onto my "real" implementation.

> Keep in mind that my personal approach is to build the whole codebase with In-Memory first then go back and do a database implementation afterwards. I can ensure that the system has been fully vetted before investing in the design of the database structures.

Let's say my real implementation is a SQL Server database that looks like the following,

<script src="https://gist.github.com/Jcouls29/e97884e1aa0d3619815b2a0a87652398#file-dbo-products-sql.js"></script>

<script src="https://gist.github.com/Jcouls29/d7fcd00a80475ad2526bd0da0f9e7932#file-sqlserverproductrepository-cs.js"></script>

This implementation is pretty straight-forward. Just insert the record after performing a few checks on the code.

How can I refactor my test class to accommodate the new implementation? The first thing I do is create a base class for my tests. It will contain an abstract method that will be used to create the specific implementation of the contract to test.

<script src="https://gist.github.com/15c7b3c6e05382afab64d9f3e0712c24.js"></script>

Then I simply create two inherited classes that properly setup the services to be tested.

<script src="https://gist.github.com/4862a6436e699183bf5d9076e78d6ddf.js"></script>

When you hit play using xUnit.NET, both sets of tests will be run: 1 with in-memory and the other with SQL Server.

The first set of tests act like "unit tests" in many ways, while the second set of tests act like "integration tests." Yet, we're using the same set of test cases.

This is important to understand the difference between these "contract tests" versus "unit tests" and "integration tests." Remember it doesn't really matter about the kind of tests we're running. What matters is that we're testing out contracts.

## Conclusion and Sign-Off

The summary of this article is simple,

1. Test your external contracts and any complex sub-systems
2. Start with the simplest implementation that meets the contracts
3. Utilize the same tests between implementations

That's it.

I didn't go through every type of contract, but the premise is the same. For a web service or UI, you'll want to first perform in-memory connections to your controller or application, preferably using in-memory repositories as appropriate. Once that contract is good, you can then test again with your "integrated" services for a closer end-to-end set of tests. Here you don't even have to change your tests themselves, just how the setup occurs for your contracts.

The reason this type of testing is beneficial is due to unnecessary changes to your tests to go from "unit" to "end-to-end" testing. With only setup changes, you can target different environments. You can use in-memory for local testing and SQL Server implementations with DEV / QA deployments.

Either way, the hardest part is determining your contracts. We discussed this in detail within [Part 3](https://medium.com/dev-genius/building-a-platform-part-3-7d63d2a3d9d9). I recommend going through that article again if you haven't fully understood the concept of contracts.

I hope you found this article helpful for testing your platform. Until next time!
