---
title: "Building a Platform: Part 3"
summary: How to design and prioritize the contracts that hold a software platform together, from stakeholder requirements to APIs, libraries, and data storage.
category: "Software Architecture"
publishedDate: 2022-02-13
tags:
  - architecture
  - interfaces
  - design-patterns
  - database
keyTakeaways:
  - Contracts are any formal or informal agreement between two entities — they span UI mockups, web APIs, SDKs, data schemas, and inter-team processes.
  - "Prioritize contract design by stability and fan-in: the contracts affecting the most clients demand the most upfront attention."
  - Document every contract — without documentation there is no agreement, no shared expectation, and no way to support the system.
  - Design APIs and libraries to their minimal footprint first, then layer additional data requirements and future-planning considerations on top.
draft: false
relatedArticles:
  - building-a-platform-part-0
  - building-a-platform-part-1
  - building-a-platform-part-2
  - building-a-platform-part-4
---

*Designing Great Contracts First*

![Photo by Leon Seibert on Unsplash](https://cdn-images-1.medium.com/max/800/0*sAGFZbAfpxyOZs5R)
*Photo by [Leon Seibert](https://unsplash.com/@yapics?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

*This post may contain affiliate links and I may earn a small commission when you click on the links at no additional cost to you. As an Amazon Affiliate I earn from qualifying purchases.*

## Series Table of Contents

[Part 0: Standard Abstraction Layers and Defining the Platform](https://blog.devgenius.io/building-a-platform-part-0-e2a8a5af62bb)
[Part 1: Types and Process of Generalization](https://blog.devgenius.io/building-a-platform-part-1-cf543658bfe3)
[Part 2: The Architecture of Your Platform](https://blog.devgenius.io/building-a-platform-part-2-cc8998716246)
**Part 3: Designing Great Contracts First**
[Part 4: Implement and Test Contracts](https://justin-coulston.medium.com/building-a-platform-part-4-91fa2173c1b7)
Part 5: Continuous Integration Early Steps
Part 6: Evolving the Platform
Part 7: Dreaded Documentation Details
Part 8: Distribution of the Platform

## Introduction

We're continuing our series on "Building a Platform." We have talked about the "Process of Generalization" and the "The Architecture of Your Platform." Now it's time to put some of this into practice.

In Part 3, we're going to be talking about contracts and the importance of these as we build out the platform.

> Code **Contracts** are formal, or informal, agreements between two entities in a software system, or external actors.

These contracts can be anything from API Design Schemas, Abstract Class Conditions, General Performance Agreements, or anything else. The word "contract" is a very general term. We will describe design principles around these contracts and which ones to prioritize.

## Common Contract Types in a Platform

As mentioned before, a contract is any agreement between two entities. Sometimes, the agreement is one-sided (where the developer / designer determines the agreement solely) or two-sided (where requirements are set from a customer and signed off). Both will happen in every project or platform. Let's discuss various types of contracts and their importance.

#### Stakeholder Requirements

These are the most common type of contracts within a system. These are typically features, or business logic, that the customer desires in a system. It is our job to ensure these requirements are met appropriately. Even if a platform isn't used, a professional developer will deal with requirements.

This type of contract is *implicit*, by nature. There isn't any real design put forth, although there may be discussion about the efficacy of a requirement. Typically, we cannot get out of this "contract." For any project, these are the most important. These also are the foundation of a platform, and subsequently, the design for the remainder of the contracts. Think about requirements as the bottom-most contract.

#### User-Interface

The user-interface (or UI) is the most visible type of contract in a system. This is where we design, get feedback, and agree to how the system may look or feel. Agreements are usually formed via mock-ups, prototypes, or general design documents.

![Balsamiq Sample Design](https://cdn-images-1.medium.com/max/800/1*ggrI01-FXOPwJkm8PTXIZQ.png)
*[Balsamiq](https://balsamiq.com/) Sample Design*

Once a design is agreed upon, it is important that an accurate representation is implemented within the final product.

#### Web API Schema

Not all users will interact solely from within a UI. Instead, they may interact with a service or endpoint, typically server-server. Sometimes it could be that the UI calls a web endpoint. Either way, a schema acts as a digital mechanism to transfer data between two interlocking (general) interfaces.

This type of contract has a high impact on interoperability and should be considered stable. Unless there is a high degree of control of the clients, and their implementation of the schema calls, changing the parameters of a schema could drastically affect many of your clients. Causing clients development time, lowers the value of the service.

> One approach to allowing more flexibility in changes in your schema is to provide integration packages that will interact with the services through a programming library or API. This can help avoid changes on the client, if they are utilizing these packages.

One benefit to building this contract early, before development has started, is that when API(s) are being used by the frontend team, you can have the frontend and backend teams working simultaneously. Since both know the contract, you don't necessarily have to wait on the backend to be finished to start development.

#### Library / SDK

If providing a platform is targeting a group of developers, much of the contracts will be around function calls and some abstractions. We, as developers, use these types of libraries every day. It could be frameworks [`Vue.js`](https://vuejs.org/) or libraries like [`System.Net.Http`](https://github.com/dotnet/runtime/tree/main/src/libraries/System.Net.Http/src/System/Net/Http). The authors target a very large audience, and consequently, must ensure backward compatibility to ensure the lowest impact when changes occur.

Libraries and SDK(s) should be considered a stable contract that should be appropriately managed.

> Internally utilized libraries, however, may not require the same level of stability, if the audience is limited.

#### Data Storage

Where, and how, we store the data for our system can have a profound impact on its usefulness. For instance, some systems will utilize the same data structures for OLTP Systems and Reporting Systems. Some systems will have a monolithic database that contains all data for all modules. While other systems may have several smaller data stores for a more microservice-esque architecture.

Regardless of the approach, the design of a data storage mechanism is a form of contract. This is especially true when more than one application accesses the same storage. When each application has its own storage, and only communicates between service calls, the stability of the design becomes less important.

#### Triggered Services

Have you ever had to create a background task that runs every day at 1:00AM? Or had to send emails out after a specific process has run? This is what we call a triggered service. We create a "contract" with the business or system, that guarantees that something will occur in response to some event.

This type of contract helps to guarantee the operation of a system. When some output goes missing, we must understand what we expect to occur. In most cases, this form of contract is not considered stable, or necessary to be so. Instead, it simply requires us to set an expectation of event-driven behavior.

#### Process Design

How do we interact with other teams? What are we going to pass to X department when we're ready to do Y? A well-designed process can also be considered a contract. It will typically be between people though (Although services and other systems is applicable as well). We will want to make sure that the design of our systems within our platform, including communication is thoughtful and efficient for all groups. And the hardest part of process design contracts? **Buy-In**.

A contract between groups may be as simple as the following:

1. Developers build a plugin as a feature addition.
2. Developers agree to produce feature installation instructions.
3. Developers agree to produce release notes on the plugin.
4. Developers pass plugin and documentation to delivery engineers.
5. Delivery engineers agree to post plugin and documentation on the platform's marketplace.

This is a very simple example, but without these agreements, the uncertainty of responsibility causes inefficiency. It is important that all groups are involved during the process to ensure there isn't an overburden on any single group. It is just as bad for developers to become more efficient but pass the inefficiency onto another group, as it is to keep it themselves.

## Contract Design Prioritization

There are many more categories of contracts that are possible, than the ones provided above. These, I believe, to be the most common we'll encounter in most projects.

The question now arises, "**Which contracts do we design?**"

In an ideal world, you would design all of them, from start to end, without fail. And in some scenarios, this does occur: Military Systems, Hospital Systems, or any system that is critical to the sustainment of life.

But in the majority of systems, we cannot be paralyzed by design and never get the application built. So, we really should prioritize which contracts to design. How then, do we pick which contracts to design? **Which ones are the most important?**

#### The Customer and The Platform

Let's sidetrack the conversation for a moment and talk about the customer needs and the platform needs.

We need to remember the purpose of our platform…

> A Platform is **NOT** built for a customer! It is about delivering a System to a Customer more efficiently.

This means, what we're calling a platform, is for our use, or the use of an internal client that will deliver a finished product to a customer. Our goal is efficiency and to reduce cost of building software.

> In the event that you're providing a "self-service platform" for an external customer to use, they will be using it to fulfill their requirements more efficiently. A great example of a self-service platform is [webflow.com](https://webflow.com/). This allows anyone to build an eCommerce site, with data storage, without ever knowing software development. Webflow is a platform for all intents and purposes.

#### Stability and Fan-In

So, how do we know which contracts are the most important to start with? This actually is a fairly easy answer,

> Prioritize contract designs with the largest stability needs, or put another way, the contracts that affect the most clients.

We should always prioritize the contracts that affect the most clients, and has the highest external fan-in. These designs will have the most impact, if they have to change.

For instance, if we're building a plugin system that will be utilized by clients to add their own functionality, we should focus on creating a highly dynamic contract that can be used by all clients. If done well, we may even be able to use this system to deliver plugins ourselves instead of altering the underlying system.

#### Return On Investment (ROI)

If we're building a large number of external contracts, we should next prioritize based on our return on investment (or ROI). We can define our ROI as,

> The total cost of building customer functionality without said contract minus the total cost of building customer functionality with said contract.

This is probably not the best definition. And it is a little abstract (and can be hard to measure). But let's use an example to illustrate,

Let's say we have to perform customer translations each time we deliver an online web product. We are utilizing a simple JSON file with key-value pairs of `key: "<translation text>"`. Each file is its own language. Let's suppose that each key is fairly inconsistent from application to application that uses these keys. When a new key is added, it is manually added to each file. Assuming we do not forget to add these keys appropriately, we end up taking approximately 2 weeks of time from a developer to ensure translations are correct on the screen (with manual testing).

The contract, in this case, is the JSON file formats and the manual process of adding the key to the files when a developer has the need. Instead, we should really make this a lot better.

What if we did the following,

All developers, on all applications, agree to a standard key format. But instead of adding the keys manually, the implementing developer creates a CLI tool that scans all code files for the keys. These keys are then populated into a standardized, computer-friendly format. This format can then be used to export to an appropriate Excel / CSV format that can be passed onto a translation team. Once completed, the team can then return the Excel / CSV file to be merged into the original. At this point, the developer may spend a couple hours coordinating between groups, but manually entering these into a JSON file, or locating appropriate keys disappears.

By coming to an agreement (a contract) on our process, we can see the return we can achieve by investing in a few small tools for the platform. At first glance, we may find it wasteful, as we are not implementing customer requirements immediately, but over time these costs add up. We should prioritize these types of improvements after experiencing the waste. When we focus on efficiency, we increase our ROI.

## Designing the Contracts

As you have seen, thus far, we have been talking about the idea of contracts and some simple ways to prioritize them. Ultimately, we're aiming for efficiency. But we should realize that not all contracts are code related. In the example above, a good portion of our ROI improvement is a process contract.

Let's walk through the tools of design for different contracts and some technical concerns required for each.

> **Note**: I'm will only cover UI, API, Libraries, and Data Storage contract types.

#### General Recommendations

Before jumping into the details of each type, I want to recommend a few guidelines to ease your pain in design. These are simply guidelines, and are not requirements, but if practiced, you'll find immeasurable value. These apply to all contract types.

**1. Collaboration**

Collaborate when doing design. I don't recommend bringing everyone and their mother to the design sessions. However, you should bring folks in that have dynamic viewpoints and can appropriately disagree with you. They will help you to question any preconceived notions you may have about your own design, only making it better.

**Recommended Tools**: [*Microsoft Teams*](https://www.microsoft.com/en-us/microsoft-teams/group-chat-software), [*Slack*](https://slack.com/), *Whiteboards*

**2. Work/Data/Process Flows**

Almost everything we do has some form of "flow" to it. It could be the user workflows in a UI to the flow of data in the system. When you start talking about concrete interfaces, the use of a good flow can make the process trivial.

I'm partial to [Microsoft Visio](https://www.microsoft.com/en-us/microsoft-365/visio/flowchart-software) myself (mainly because I know it best). However, I do know that it can be limiting. I've used diagramming software like [Lucidchart](https://www.lucidchart.com/pages/) in the past as well and found it to work well (if not better in some scenarios). And if neither of those work you can always produce the final documentation, after manually working it out (whiteboarding), in something like [PlantUML](https://plantuml.com/).

![Example Workflow](https://cdn-images-1.medium.com/max/800/1*ps54RPQChWSkfn3I7t4O9w.png)
*Example Workflow*

![Lucidchart Dataflow Diagram Example](https://cdn-images-1.medium.com/max/800/1*exDFdm86LsmqnrQ5jIuRFg.png)
*Lucidchart Dataflow Diagram [Example](https://www.lucidchart.com/pages/data-flow-diagram)*

These flows themselves offer a form of contract design. Getting these done will save a lot of communication when trying to get buy-in from other developers and designers.

**Recommended Tools**: [*Microsoft Visio*](https://www.microsoft.com/en-us/microsoft-365/visio/flowchart-software), [*Lucidchart*](https://www.lucidchart.com/pages/), [*PlantUML*](https://plantuml.com/)

**3. Design Sessions**

When you do perform design sessions on these contracts with other individuals, record them (with the permission of the team of course). A lot can be said and taking notes during a meeting (and doing it well) is hard. So, record them, review them, and ensure that everyone is in agreement by the end of the session.

Even if you're doing a design session by yourself, record it. Rubber duck the session and talk out loud. You'll find that even talking it out loud a lot of Aha! moments will jump out.

**A sample agenda for a design meeting**

1. (Start) Summarize Problem or Requirement
2. Draw out Flow(s) for the problem and talk through all use cases known (Allow others to propose other use cases)
3. Debate High-Level decisions (technology, architecture, etc)
4. Sketch Low-Level decisions (API Design, Abstract Classes, etc)
5. Debate Design and Repeat 4–6 until a decision is agreed
6. (End) Summarize decisions of the design

> Have questions and thoughts before ever starting a design decision. Be sure to defend your position. You will, and should, get pushback. However, be able to concede if a better design emerges.

**Recommended Tools**: [*Microsoft Teams*](https://www.microsoft.com/en-us/microsoft-teams/group-chat-software), [*OBS*](https://obsproject.com/)

#### User-Interface (UI)

With a user interface "contract" design the goal is NOT high-fidelity. Instead, it is about intent. Assuming we already have some workflows in place, we should utilize a tool that can be used to easily walk through the design in a storyboard fashion.

For a developer, I usually prefer [Balsamiq](https://balsamiq.com/). However, I have also used [Adobe XD](https://www.adobe.com/products/xd.html) for rapid prototyping. Another tool that I have seen used is [Figma](https://www.figma.com/), although I do not think this is as tailored to developers as it is UI/UX expertise.

The goal with any design here is to understand the underlying mechanisms you'll need. I would break this up into several stages:

1. **Raw Design**: Design the UI in a very literal way to what you think the user will need.
2. **Componentize**: Find common elements and generalize components for re-use throughout the UI.
3. **Document**: Document the design

![Adobe XD (WindowsUI Kit)](https://cdn-images-1.medium.com/max/800/1*kDID9CIJqHwTBK_w44Earw.png)
*Adobe XD (WindowsUI Kit)*

Stage 1 is just about the design. Again, this can be within Adobe XD or Balsamiq and isn't meant to be a high-fidelity design. It is just intended to understand what the UI will require.

Stage 2 is where we start developing out the contracts we care about. We will componentize the design. This is where we define the components that can be re-used throughout the design, or even better, throughout multiple applications.

Stage 3 is about documenting the components. This can best be described through simple documentation guidelines. In most cases, a formatted markdown file should be enough.

Some considerations for the component design are the following:

1. **What is the input data**? Is there a specific format required to make this work interchangeably?
2. **What will the component display** versus what will it have to maintain in the background?
3. **Is the component interactive**? What are the events you expect for it to operate fully?
4. **How will the component change between different use cases**? Can the changes be quantified? Is it just style differences? Or are their different visual layouts? What about enabling or disabling different aspects of the component? Answering this question will directly affect the options of the component.

By understanding and documenting the answers to these questions, you will have a clear set of requirements.

When it comes to documentation of UI elements, it is recommended that you follow a common guideline to defining the interface of a component in your system. Here is an example below,

```
# Component: Internationalization (i18n) Label

## Status
Proposed

## Description
A simple label that will automatically translate text based on a key and language offering. The element will be an `inline-block` that utilizing the `span` HTML element. An implicit class of `i18n` will be on all elements.

### Input
| Attribute | Type | Description |
| --------- | ---- | ----------- |
| data-key | string | The i18n key to be used |
| lang | string | i18n standardized language value |
| default | string | The default value to show while the strings are loading |

### Events
N/A (Non-Interactive)

### Functionality
A service is required to perform the replacement of these keys appropriately. The component itself should not perform the replacement but rather it acts as a holder of the keys.

### Usage
`vue.js` Example
<i18n data-key="page1.title" :lang="options.language" default="The title of the page" />

**NPM Package**: `internal\general-components\vue`
**TypeScript Interface**: `VueI18n`
```

Of course, change this according to your needs. There are lots of ways to design a decision record. The real goal is to provide a clear understanding of the component to those developing and utilizing it.

I recommend that these documents live either:

1. In an agreed upon centralized global location with appropriate access controls
2. With the code library itself, centralized within the repository.

#### Web API Schema

When creating web API schemas, it is increasingly important that you have appropriate documentation. However, the process of describing a schema is itself a big topic. But before we describe this process, let's talk about the stages of designing this contract.

Remember, the number of clients accessing your API will determine the level of generalization you may require. Although, there are no hard and fast rules, this does require a level of creativity and future planning. The stages of design can be summarized by the following:

1. **Minimal Request / Response Footprint**: What are the minimum fields required in your system to operate the core functionality?
2. **Additional Data Requirements**: What are the fields that are needed by the clients? This could be identifiers they may require to be returned.
3. **Future Planning**: Do we expect this API to change a lot? What are common changes that we can foresee?
4. **Design Testing**: Always test the designs against use cases, to ensure compatibility
5. **Document Design**: Document the design for usage by clients and development.

I recommend that at each stage, really write out the sample requests / responses. It can just be samples of the requests. As you start to add to the design, you'll come to a conclusion as to what you should use.

As a part of the contract, here are some considerations,

- **Should we version the API?** This will allow for us to more easily alter it without affecting clients. This is the safest way to provide changes in a backwards compatible way. (i.e. `/api/v1/products`)
- **How will we handle authentication/authorization?** Is authentication handled separately from the request (i.e. Login endpoint) or are we using inline authentication (Basic Auth).
- **Are we going to need paging and sorting?** This is a feature that should work consistently within an API. These cross-cutting type of concerns should be fluid.
- **Will you support multiple content types?** Do you support `application/json` only? Or will you support `application/xml` or some other custom content types?
- **How will you use HTTP Response codes?** Does 404 mean the resource wasn't found? Or will you always return Bad Request? Although these are standardized, sometimes there are cases where you may standardize certain modes.
- **How will errors or problems be presented to the caller?** Besides the basic response code, what will the format of the error be? Will it be included in the primary response model? Or will you use a common model like [Problem Details](https://datatracker.ietf.org/doc/html/rfc7807)?
- **What will happen to additional properties?** Will you reject them outright? Will a Bad Request response be returned if extra fields are detected? Will they ever be used?

There are a lot more questions that could be asked, but these are some of the bigger ones.

Lastly, let's mention documentation. My preference for documentation is the [OpenAPI](https://www.openapis.org/) specification. It can be pretty complicated to do directly, so you can mock it up in your system first, then export it ([Swagger](https://swagger.io/)).

Once you have completed the JSON file, you can use it directly in your software or use a tool like [Redocly](https://redoc.ly/) to convert to an appropriate markup language for Wiki. You can host this internally or you can find a cloud provider to host it for you.

![Redocly Example](https://cdn-images-1.medium.com/max/800/0*xdMb7CFVw0XusavT.png)
*[Redocly Example](https://redocly.github.io/redoc/)*

> I think it is worth noting that for technologies like SOAP, GraphQL, and gRPC things are documented differently. Although, there are solutions to this, we will not be covering them here.

#### Library / SDK

When developing a library or SDK for external use, these contracts should be designed the same way as a Web API:

1. **Minimal Request / Response Footprint**: What are the minimum fields required in your system to operate the core functionality?
2. **Additional Data Requirements**: What are the fields that are needed by the clients? This could be identifiers they may require to be returned.
3. **Future Planning**: Do we expect this API to change a lot? What are common changes that we can foresee?
4. **Design Testing**: Always test the designs against use cases, to ensure compatibility
5. **Document Design**: Document the design for usage by clients and development.

The difference is in the implementation. For instance, you may provide factories, abstract classes (interfaces), project templates, or code generators. Regardless, the premise is the same, design to the generic case first.

The following are some guidelines when building a library or SDK for your clients,

- **Utilize Abstract Classes**: Don't force users to tap into concrete classes directly. This will significantly limit your ability to evolve.
- **Keep Contracts Small**: Don't create God abstract classes. Keep them as small as you can while retaining the functionality.
- **Use Extension Methods (C#)**: If using .NET, use extension methods to extend functionality around abstract classes and interfaces, rather than putting them on the interface.
- **Provide Factory Classes or Methods**: Use factories to allow your clients to quickly create the classes, instead of letting them create the classes directly.
- **Provide Builders**: Give the user some builders that will ease the pain of implementing a complex feature setup.
- **Http Clients**: When providing a library for connecting to a remote service, follow best practices in the language of choice, instead of trying to match to the literal request / response object formats. These may have to change in the future. You want a logical contract.
- **Maximize Encapsulation**: Attempt to protect everything until you have no choice. This means private classes until you need to provide it to a client. The more you protect, the better the chances of users not using the library incorrectly.
- **Separate Appropriately**: Separate out assemblies and packages appropriately. Don't throw everything into one big package. If you're trying to provide convenience, you can create a separate package that imports the individual packages as necessary.
- **Consistency**: Be consistent in approach and naming. Whatever you decide, don't deviate, especially for external clients.

Once you have a design that you believe will work, use it.

> Use your own libraries! You'll quickly find what is wrong with the library when you get frustrated with it.

The goal of libraries is to bring convenience, while allowing clients to use the contracts in the same way.

When you're ready to document your designs, document it as a developer. How would you like to see it? Use examples of documentation that you appreciate. Here are some I appreciate:

- [Dapper](https://github.com/DapperLib/Dapper)
- [.NET Core Configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0)
- [Automatonymous](http://masstransit-project.com/Automatonymous/use/creating.html)
- [Vue.js — Router](https://router.vuejs.org/installation.html)
- [ExpressJs — Routing](https://expressjs.com/en/guide/routing.html)

Documentation comes in all shapes and sizes. Ultimately, make it easy with examples and clear rules. This is the contract!

#### Data Storage

Storing data is a little different than the others. Although this process can work the same,

1. **Minimal Request / Response Footprint**: What are the minimum fields required in your system to operate the core functionality?
2. **Additional Data Requirements**: What are the fields that are needed by the clients? This could be identifiers they may require to be returned.
3. **Future Planning**: Do we expect this API to change a lot? What are common changes that we can foresee?
4. **Design Testing**: Always test the designs against use cases, to ensure compatibility
5. **Document Design**: Document the design for usage by clients and development.

There are some key things to keep in mind. Clients are very much internal customers. It is very rare to expose a database to external customers directly. For this reason, it is important to take the following into consideration,

- Are we using SQL, Graph, or NoSQL technologies?
- Will the database be used for reporting along with transactions?
- How many clients will use this database?
- Are there any concurrency concerns?
- How is change control and auditing of records handled?
- Are we performing soft deletes or hard deletes?
- What tables, or containers, will be hit the hardest?
- What do we do with large object data? Should it be separated for performance reasons?
- Will SQL be inline or in stored procedures (where available)?
- Will the data be partitioned? How many servers will be used?
- How does mirroring, replication, or availability groups get affected by the design?
- Will we delete data after some period of time?

And many more. Data design is a very difficult aspect of software. You don't really know the issues with your design until it is too late. This means that the design upfront is more about understanding the technology and the ramifications early on, before issues occur.

When working with different technologies, you have to focus on the nuances of each. Then document these designs.

I don't have any strong recommendation on documentation styles as it depends on your technology. Some examples,

- SQL Server — Use a Database Project with Comments
- Graph DB — Visio/Lucidchart: Generalized Graph Diagram describing Relationships
- NoSQL Document — Markdown: Expected JSON Storage Schema
- NoSQL Key-Value — Markdown: Key patterns and expected data

Regardless, these designs are important. I would even recommend that instead of just describing the structure, it is even more critical to describe the queries.

> Provide query examples with all data storage documentation.

This is far more valuable from a usage standpoint. Yes, we'll always need the structure to understand what is available. But from a data designer perspective, examples provide better context, and intent, to the developer. Include the most common scenarios in the documentation.

## Conclusion and Sign-Off

This article really focused on the ideas and recommendations when creating contracts. Of course, there are many more than described in this article. What I really want to express when building a platform is that any contract being built should be documented. Documentation is the key to any contract.

I'm not suggesting that we do this manually. You can create documentation with automatic tools, but documentation nonetheless is required to be successful with contracts. Without them, there is no agreement, no expectation of how the system will work or communicate.

I hope this article has helped provide more detail to contracts and their importance. Always get your contracts down early on. Don't wait, otherwise you'll find yourself reworking a lot of code. And don't skimp on the documentation. No, it isn't the most fun aspect of being a developer, it is the most critical, not only during design, but support.

Thank you for reading! Until next time!

## Bonus! Designing for Multiple Options

I suspect some folks will mention the idea of requirements and Agile. How can we create a contract without knowing all the details? Agile is about reacting to change quickly. Why deal with contracts early one?

Good question.

This is where flexibility in design comes into play. You may not know ALL the solidified requirements, but you probably know some. For the ones that are unknown, it is typically going to be one of a few options. It is often common to know what options are on the table. You just might not know which one is going to win. When this occurs, you have to realize that the deciders are debating, and if they're debating it means it is a close call. This means that it is very likely that both scenarios will be needed in the future as more clients come on board.

So, I ask you: Can you handle all the scenarios? Why not take each into consideration? I'm not suggesting build all the features right away, but at least make sure your contracts can handle the various scenarios. This will make your platform more flexible to change in the future, reducing your costs down the road.
