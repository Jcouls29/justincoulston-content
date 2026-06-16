---
title: "Building a Platform: Part 0"
summary: An introduction to Standard Abstraction Layers and what defines a software platform, using the vehicle analogy to map abstraction levels from hardware to systems.
category: "Software Architecture"
publishedDate: 2022-02-02
tags:
  - architecture
  - design-patterns
  - interfaces
  - assemblies
keyTakeaways:
  - A platform is a layer of abstraction between external services and your internal application code, designed to reduce cognitive load and increase development efficiency.
  - The Standard Abstraction Layers model (Hardware, Kernel, Interpreter, Framework, Service, Interface, System) gives a consistent vocabulary for reasoning about where your code lives and what it depends on.
  - The level of complexity of a system is proportional to the number of abstract layers required to sufficiently maintain it.
  - Building a platform is about transforming complex steps into the minimum number of required actions for the next layer, minimizing mistakes and increasing speed.
draft: false
relatedArticles:
  - building-a-platform-part-1
  - building-a-platform-part-2
  - building-a-platform-part-3
  - building-a-platform-part-4
---

## Series Table of Contents

**Part 0: Standard Abstraction Layers and Defining the Platform**
[Part 1: Types and Process of Generalization](https://justin-coulston.medium.com/building-a-platform-part-1-cf543658bfe3)
[Part 2: The Architecture of Your Platform](https://justin-coulston.medium.com/building-a-platform-part-2-cc8998716246)
[Part 3: Designing Great Contracts First](https://justin-coulston.medium.com/building-a-platform-part-3-7d63d2a3d9d9)
[Part 4: Implement and Test Contracts](https://justin-coulston.medium.com/building-a-platform-part-4-91fa2173c1b7)
Part 5: Continuous Integration Early Steps
Part 6: Evolving the Platform
Part 7: Dreaded Documentation Details
Part 8: Distribution of the Platform

## Series Introduction

We live in a world of abstractions. In most cases, we never notice these layers. Sometimes we simply call it engineering. But they exist everywhere.

Let's take a vehicle for example. When you accelerate on the gas pedals of your car, there are a number of layers between your foot and the movement of the vehicle. Not all layers are physical. Some are only logical layers, but layers, nonetheless.

![Photo by Justin Coulston](https://cdn-images-1.medium.com/max/800/1*jcBhgp8m4tjUhQ6zbYgRdg.png)
*Photo by Justin Coulston*

It is a bit difficult mapping these to a feat of engineering like building a car but let's walk through each piece from the top.

1. The **System** as a whole is the Vehicle itself (the finished product)
1. We **interface** with the vehicle with a pedal (or accelerator)
1. This in turn utilizes an abstract **service** called "Speed Control" for the vehicle.
1. A vehicle has a few **frameworks** (or sub-systems) that are used for speed control including Fuel Injection and Combustion
1. These frameworks are made up of components that **interpret** the desired user action by utilizing the clutch and torque converter "interfaces."
1. The **kernel** is made up of the major supporting aspects of the vehicle like transmission and driveshaft.
1. And finally, the core aspects of the vehicle are the **hardware** we're trying to utilize like the engine and wheels.

At first glance it will probably not make a lot of sense why these fit into the spots they fit into. For instance, Transmissions and clutches are still hardware pieces, but they are not sitting at the hardware layer.

Each layer is built off the layer before it, however, it does not imply that additions cannot be added at each layer. The transmission is a supporting mechanism to the hardware (engine and wheels) allowing for gears shifts and speed control. The clutch and torque converter interpret inputs and outputs for the transmission and driveshaft. We create a system for use at a higher level with a "Speed Control" service. In this example, it is more an abstract concept than a physical service in the system.

The goal of this series to provide a reasonable understanding of abstractions and building a platform for your organization. Building platforms is not trivial and requires an understanding of how abstractions are built upon each other to form more complex structures. This first article in the series will break down the layers and their purpose according to software engineering.

Let's jump in!

## Standard Abstraction Layers

In order to understand what a platform can provide to your organization, we first need to understand what it looks like without a cohesive platform. Let's talk through each layer of Standard Abstraction Layers (SAL) model.

![Photo by Justin Coulston](https://cdn-images-1.medium.com/max/800/1*VVW1Ojp56jvqM_s4eu0kQw.png)
*Photo by Justin Coulston*

## External / Vendor Layers

Let's start with the layers we're most familiar with: What we buy and build our software with, External Layers.

An external layer is more of an informal grouping of layers. It is more about defining what your organization buys or utilizes and you don't have to maintain. Of course, your organization can build and maintain all layers of a system. This is atypical and cost-prohibitive in most scenarios.

**Hardware**

The hardware is the layer where all other aspects of the system sit. It is the physical, or mechanical, engine. In software, this is a Server, Laptop, Phone, Smart Watch, etc.

The hardware layer itself must have some interface to work with the system. In most cases, it will be instruction sets (Intel x86, ARM, Microprocessors)

> Example - [Intel 64 and IA-32 Architectures](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf)

**Kernel**

Most hardware instruction sets do not abstract any of the capabilities of the hardware and instead give you a 1-to-1 match to the hardware. For this reason, kernels are created to provide another layer of abstraction.

A kernel is a set of compiled instructions, interfaces, and capabilities grouped to form some re-usable system. In most cases, we see a kernel as an operating system (Windows 11, macOS, Android, Linux).

Kernels will provide the core aspects of resource management (memory, storage, processes and threading, etc).

> Example - [Linux Kernel](https://github.com/torvalds/linux)

**Interpreter**

Although you can program directly off the kernel, and this is perfectly valid, in today's world, we want to write code that is cross-platform and can run on many different hardware and OS platforms.

The most common approach is to use a "literal" interpreter. This can be thought of as a virtual machine or isolation environment. Some common examples are the [.NET JIT Compiler](https://docs.microsoft.com/en-us/dotnet/standard/managed-execution-process#compiling_msil_to_native_code), [Java Virtual Machine](https://en.wikipedia.org/wiki/Java_virtual_machine), and [Chromium JavaScript Engine](https://v8.dev/).

What's important to understand about the interpreter layer is that it isn't necessarily required but is a form of abstraction to help developers more easily create software.

**Framework**

A framework is a standard set of libraries, services, and connectors into the OS or interpreted language to ease development. For example, if working with the [.NET Core Framework](https://github.com/dotnet/runtime), the code will be compiled to an intermediate language (MSIL). The tooling designed around the framework and the standard libraries is what is most familiar to developers.

> This is a good place to pause and mention that at this point, developers typically stop. In many ways, developers choose frameworks at the beginning of a project and build from scratch from this standard set of libraries and tools. We want to fix this.

> Other Examples - [Vue.js](https://vuejs.org/), [Angular](https://angular.io/), [MFC Library](https://docs.microsoft.com/en-us/cpp/mfc/mfc-desktop-applications?view=msvc-170)

**Service**

The service layer is the topmost layer of a typical development stack that is not developed internally. These include all the libraries built on top of frameworks that are created to solve a specific need. These include things like [Json.NET](https://www.newtonsoft.com/json), [Vuetify](https://vuetifyjs.com/en/), [libsodium](https://doc.libsodium.org/).

To be clear, service(s) may have "framework" in their name. For instance, Vuetify is a "Design Framework." However, in line with this model, it sits on top of vue.js and so can be considered a part of the service layer.

It should also be noted that services do not necessarily need to live on your selected stack. For instance, web services or SaaS products live at this layer as well. Think [Auth0](https://auth0.com/), [Grammarly](https://www.grammarly.com/) or [Twilio](https://www.twilio.com/). These are external services that will be utilized by your applications and so live at the service layer.

> **Another important note!** These layers are a matter of perspective. Although, Auth0 may be considered at the Service layer in my system, to Auth0, it may be at their Application Layer or System Layer. This distinction is important. The model is contextual from the organization standpoint, not globally.

## Internal Layers

The most common set of layers developed internally are Interface and System level layers. Many of the companies I have worked with lack the "platform" layer. We'll talk more on this specific layer later.

**Interface**

The interface layer is about the interaction between any of the following: users, external systems or internal processes. Many of us create interfaces daily. This could be desktop applications that our users will use. It could be background services that process information. It may be web services that are accessed by other systems. We are defining "interface" as a contractual agreement between the developed application and its audience. An audience doesn't necessarily need to be users, but any computer, process, or system.

**System**

A system is a group of interfaces that work together to form a complex array of interactions to solve a target need or problem. For instance, you may be building an inventory system. You could have the following:

1. Web UI where the users manage inventory and products
1. A background service that monitors barcode scanning applications for warehouse operations
1. A web service that controls information flowing in/out of a database

Each of these is a separate interface that forms an inventory system.

A system is in the last layer and provides direct value to our audience. Our goal is to speed up the process of providing value to our audience. In order to accomplish this, we will need abstractions. Let's walk through the idea of abstractions first.

## Theory of Abstractions

There are several definitions of abstractions out there, although most are considered informal.

> Informally, abstraction can be described as the process of mapping a representation of a problem onto a new representation

> - [A theory of abstraction, Fausto Giunchiglia](https://www.sciencedirect.com/science/article/abs/pii/000437029290021O)

Another interesting take on abstractions,

> An abstraction can be seen as a [compression](https://en.wikipedia.org/wiki/Data_compression) process, mapping multiple different pieces of [constituent](https://en.wiktionary.org/wiki/constituent) data to a single piece of abstract data; based on similarities in the constituent data, for example, many different physical cats map to the abstraction "CAT". This conceptual scheme emphasizes the inherent equality of both constituent and abstract data, thus avoiding problems arising from the distinction between "abstract" and "[concrete](https://en.wikipedia.org/wiki/Concrete_%28philosophy%29)". In this sense the process of abstraction entails the identification of similarities between objects, and the process of associating these objects with an abstraction (which is [itself an object](https://en.wikipedia.org/wiki/Abstraction#Physicality)).

> - [Wikipedia](https://en.wikipedia.org/wiki/Abstraction)

I think both of these definitions are valid and define our needs very well. The idea that abstracting is a "compression process… based on similarities…" is exactly what we need.

> An abstraction, or abstraction layer, is a compressed layer of functionality, or data, with appropriate similarities for the models being generalized mapped in a way to simplify upper layer models and interactions.

> - Justin Coulston

## Why is abstraction important?

Can you imagine if you tried to write every software application in straight assembly language? And it only works on one type of computer?

Well… that would be awful.

This is the obvious answer. We all know that abstractions are important conceptually, because we use them constantly. But why should you create a layer of abstraction? Or why should you keep a mindset of continuous abstraction.

Abstractions are the essence of efficiency in software development. Every time you create a function that is reused by more than one other function, you have created an abstraction. Every time you create a shared library where you can store common functionality, you have created an abstraction. Every time you create an interface or abstract class, you (obviously) create an abstraction. And you do this every time to save yourself work, time, and future grief when requirements change.

So, my goal is not to tell you that abstractions are important. You already know that. You do it every day. The goal is to show a process of disciplined abstractions paving the way to forming a solid platform. We want to build software faster, with higher quality, and one that meets the needs of the consumer. This means we need to create an underlying structure, and maintain this structure, that will continuously make our software process better.

## Defining the Platform

Let's look at the stack again,

![Photo by Justin Coulston](https://cdn-images-1.medium.com/max/800/1*6C3-jv7m0vYaBJ5oeVSlHQ.png)
*Photo by Justin Coulston*

In this image, I am showing a single block called "Platform." In this representation, I am implying that we should build a layer between all services we use externally that will ease the burden of development at the interface level. However, it should be noted, that 1 layer may not be enough. As more and more complex requirements are supplied, more and more layers of abstractions will be required.

> The level of complexity of a system is proportional to the number of abstract layers required to sufficiently maintain said system.

Imagine that we want to build a Calculator application. In all honesty, we probably wouldn't care one bit about a platform. However, what if we wanted to build [SPICE](https://en.wikipedia.org/wiki/SPICE) software (Electronic Circuit Simulator) that requires complex matrix calculations? Well, in this case you will want to build a layer or two of abstraction to make maintaining that software easier. Why? Because maintenance of complex software is difficult if you have to constantly re-invent the same, or similar functionality, every time it is needed.

Let's take it to the extreme. What if we want to build a starship? Something that can travel light years away. In this case, to allow humans to mentally manage a starship (with all the complex sub-systems: atmosphere, artificial gravity, FTL drives, weapon systems, warning systems, etc.), there will need to be enough layers to decrease the cognitive load from all the complex systems in a way that can be managed from a small set of interfaces. Humans can only hold approximately 7 pieces of information in their short-term memory. We have to optimize for this limitation. By mapping these layers to simpler interfaces (like the gas pedal), that will achieve the same outcomes, provides a more maintainable structure long-term.

And now we're at the climax of the topic.

> A platform is used to transform, or map, complex ideas to simple ideas to ease the cognitive load on the target audience.

To offer a more practical example, maybe our company has a fairly large set of inventory operations that must be performed. If an operation, like removing an item from inventory, took 25 steps to accomplish manually, there are bound to be mistakes due to the large number of actions required. We should, in this case, build a platform that will provide the user the ability to only action 1 or 2 steps. These steps are mapped to our 25 steps in some way making the chances of mistakes minimal, while also increasing efficiency.

Then simply put, our platform is used to minimize mistakes and increase efficiency by abstracting complex steps (or workflow) into a minimum number of required actions for the next layer. Since we're writing this platform for ourselves, it is important that we design it to make ourselves more efficient.

As new requirements are introduced, more layers may be needed. However, for the sake of this series, we will introduce only 1 layer. After that, it will be your responsibility to take it to the next level, as you utilize what we learn here in the future.

## Next Steps

The remainder of the series we will be discussing the process of creating abstractions and building a platform. The platform will be built and designed using .NET Core and C#, as this is my preferred language. However, the principles here will be easily translated to other frameworks and services.

This series does not inherently have an end in mind. This is because there is a plethora of topics to be discussed. The "Platform" is how we're going to build the future, so I suspect we will be creating new standards of development as we move through this.

## Conclusion and Sign Off

The idea of abstractions and "The Platform" is an exciting topic for me. It is complex, yet simple; A new way of working, while being a classic process. It can redefine your way of thinking, making you more efficient with every project you are assigned. I'm excited about what this series can bring out.

If you have any thoughts or suggestions, please comment below. And be sure to **follow me** on medium, as I release new topics all the time!

Until next time!
