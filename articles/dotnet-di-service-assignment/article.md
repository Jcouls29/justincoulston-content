---
title: .NET DI — Service Assignment
summary: Three progressively better approaches to injecting specific interface implementations into particular services using ActivatorUtilities, without registering concrete types in the DI container.
category: "C# & .NET"
publishedDate: 2024-01-19
tags:
  - dotnet
  - dependency-injection
  - csharp
  - design-patterns
keyTakeaways:
  - The built-in .NET DI container does not natively support assigning a specific implementation of an interface to one service but not another; a factory function or ActivatorUtilities is required.
  - ActivatorUtilities.CreateInstance resolves remaining dependencies from the container while letting you override specific constructor arguments, keeping concrete types off the global registry.
  - A fluent InjectedServiceCollection builder (Option #2) offers the cleanest API and makes the intent clear at the call site, trading a small amount of initial complexity for long-term readability.
  - None of these patterns handle injecting two instances of the same type into different constructor slots, so they are best suited for platform-level framework code rather than everyday service wiring.
draft: false
---

Specify implementations for injection into specific services…

![Photo by Kelly Sikkema on Unsplash](https://cdn-images-1.medium.com/max/800/0*QPy0SoBtGObdOVCi)
*Photo by [Kelly Sikkema](https://unsplash.com/@kellysikkema?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Ugh, I would say that as I get older, and utilize more and more contracts, the more frustrated I get with how DI Containers work. I mean, they're not overly complicated, and honestly, most are fairly feature-complete. But there is always that one thing I need to do that it cannot easily do, or is not easy to figure out on "The Google." Recently, that "thing," was the ability to assign an implementation of an interface to a specific service, without having to completely build out a factory function…

This is my story.

## Problem Statement

Let's start with what I am trying to achieve.

Let's say I have two serializers, `ISerializer<TOutput>`, with concrete types `JsonSerializer` and `XmlSerializer`. The contract really doesn't matter for this discussion, but for those that MUST have it, let's say it has the following contract,

```csharp
public interface ISerializer<TOutput>
{
  TOutput Serialize<T>(T value);
  T Deserialize<T>(TOutput value);
}
```

Again, it really doesn't matter, so comments on why this would be a terrible interface is moot.

What I would like to do is have two classes within my DI container take these implementations. For example, let's pretend I have these services,

```csharp
public class WebConfigLoader
{
  // Need: XmlSerializer
  public WebConfigLoader(ISerializer<string> serializer) { ... }
}

public class TableStorageRepository
{
  // Need: JsonSerializer
  public TableStorageRepository(ISerializer<string> serializer) { ... }
}
```

Now, you may ask immediately, why not simply inject these services like the following,

```csharp
public class WebConfigLoader
{
  public WebConfigLoader(XmlSerializer serializer) { ... }
}

public class TableStorageRepository
{
  public TableStorageRepository(JsonSerializer serializer) { ... }
}
```

Well, I could absolutely do that, but it has two major downsides for my goals as a developer, and an architect,

1. I have no way to unit test this directly. And I now have directly injected the serializer and cannot easily mock it (although, yes it is still possible).
2. It implies that these are now required types on the service, which is not necessarily true. I want this decision to be made at the DI container layer when the application is being put together. These services, depending on their context may have different needs at different times.

So, the goal is to hook these up at the DI Container level (i.e. `Startup.cs`) and not imply the dependency at the concrete type level. Let's look at the classical approach. Then we can look at other way(s).

## Classical Approach

The way I have done this in the past is by simply using an implementation factory to perform this connection. Here is an example of this approach,

```csharp
public void ConfigureServices(IServiceCollection services)
{
  services.AddSingleton<XmlSerializer>();
  services.AddSingleton<JsonSerializer>();
  services
    .AddSingleton((p) => 
      new WebConfigLoader(p.GetRequiredService<XmlSerializer>()));
  services
    .AddSingleton((p) => 
      new TableStorageRepository(p.GetRequiredService<JsonSerializer>()));
}
```

This has always been my normal approach to this problem. This is because the built-in service provider by Microsoft does not provide a direct way to assign a specific implementation of `ISerializer<string>` to another service.

This approach isn't really that bad, but it does become a problem with higher-level services that have 5 to 10 dependencies. At that point, it becomes kind of a pain to deal with…

I'd prefer to use a different approach, that doesn't require extreme amounts of typing. I would also like a version, that doesn't require these concrete implementations to be directly registered to the container. There are a lot of cases where having them registered is not desired.

## ActivatorUtilities.CreateInstance

The `ActivatorUtilities` ([reference](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.activatorutilities.createinstance?view=dotnet-plat-ext-8.0)) static class is a very useful set of functions that allows for the creation of class instances by providing an `IServiceProvider` and any number of parameters.

The following is the method signature for `CreateInstance`,

```csharp
public static object CreateInstance(
  IServiceProvider provider, 
  Type instanceType, 
  params object[] parameters
);
```

The `provider` is used to fill any dependencies required by the type given by `instanceType`. If you were to only use these two arguments, nothing changes for the instance you're trying to create. However, the third argument `parameters`, makes all the difference to my use-case. Any values passed into this array will override any values given by the provider. Here is how we can make a difference and create specific instances for our services.

Let's show an example doing this directly,

```csharp
public void ConfigureServices(IServiceCollection services)
{
  services
    .AddSingleton((p) => 
      ActivatorUtilities.CreateInstance(p, typeof(WebConfigLoader), 
        ActivatorUtilities.CreateInstance(p, typeof(XmlSerializer))));
  services
    .AddSingleton((p) => 
      ActivatorUtilities.CreateInstance(p, typeof(TableStorageRepository),
        ActivatorUtilities.CreateInstance(p, typeof(JsonSerializer))));
}
```

Here, we're creating the `WebConfigLoader` dynamically using `ActivatorUtilities`. The dependency `XmlSerializer` is also created using `ActivatorUtilities`. This provides the benefit of making it so `XmlSerializer` doesn't need to be registered with the container while filling in any other dependencies `WebConfigLoader` may need.

Of course, this is extremely ugly, and if this is where we were to stop, I'd say just do what we were doing earlier. At least it looked better. But NO! We want this to be easy. So, let's throw out some API ideas to make this cleaner.

```csharp
// Option #1
services.With<TImplementation>(s => s.AddSingleton<TS, TI>()...);

// Option #2
services
  .With<TImplementation>()
  // ...
  .With<TImplementation>()
  .AddSingleton<TS, TI>()
;

// Option #3
services.AddSingletonWith<TS, TI>(params Type[] implementations);
```

These options are just my first passes at some design.

#### Option #1

This seemed to me to be the quickest and easiest to implement. You can create a very simple extension method that provides this functionality.

For example,

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection With<TImplementation>(this IServiceCollection services, Func<IServiceCollection, IServiceCollection> configure)
    {
        var innerServices = configure(new ServiceCollection());
        var implementationFactory = 
            (IServiceProvider provider) 
                => ActivatorUtilities.CreateInstance(provider, typeof(TImplementation));

        foreach(var descriptor in innerServices)
        {
            if (descriptor.ImplementationType != null)
            {
                services.Add(new ServiceDescriptor(
                    descriptor.ServiceType, 
                    (IServiceProvider provider) 
                        => ActivatorUtilities.CreateInstance(
                            provider, 
                            descriptor.ImplementationType, 
                            implementationFactory(provider)), 
                            descriptor.Lifetime)
                          );
            } else
            {
                services.Add(descriptor);
            }
        }

        return services;
    }
}
```

Of course, this implementation is fraught with issues. First, it only allows you to inject one service for a given set of services. This is a severe limitation and really eliminates its usage as a solution.

Also, it really doesn't allow you to inject with any other methodologies. This extension will not have any effect on registrations that utilize implementation factories or instance types, even though the way it reads, it implies it might. Not having clear intentions will cause issues, questions, and complaints. So, probably not the best… but it does work.

#### Option #2

This option is marginally more complicated than Option #1. Here is a quick and dirty implementation,

```csharp
public static class ServiceCollectionExtensions
{
    public static InjectedServiceCollection With<TImplementation>(this IServiceCollection services)
    {
        return new InjectedServiceCollection(services, new[] { typeof(TImplementation) });
    }
}

public class InjectedServiceCollection
{
    private readonly IServiceCollection _Services;
    private readonly Type[] _InjectionServices;

    public InjectedServiceCollection(IServiceCollection services, Type[] injectionServices)
    {
        _Services = services;
        _InjectionServices = injectionServices;
    }

    public InjectedServiceCollection AddSingleton<TService, TImpl>()
    {
        _Services.AddSingleton(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TImpl), 
            CreateInjectionServices(p)));
        return this;
    }

    public InjectedServiceCollection AddSingleton<TService>()
    {
        _Services.AddSingleton(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TService), 
            CreateInjectionServices(p)));
        return this;
    }

    public InjectedServiceCollection AddScoped<TService, TImpl>()
    {
        _Services.AddScoped(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TImpl), 
            CreateInjectionServices(p)));
        return this;
    }

    public InjectedServiceCollection AddScoped<TService>()
    {
        _Services.AddScoped(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TService), 
            CreateInjectionServices(p)));
        return this;
    }

    public InjectedServiceCollection AddTransient<TService, TImpl>()
    {
        _Services.AddTransient(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TImpl), 
            CreateInjectionServices(p)));
        return this;
    }

    public InjectedServiceCollection AddTransient<TService>() where TService : class
    {
        _Services.AddTransient(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TService), 
            CreateInjectionServices(p)));
        return this;
    }

    public InjectedServiceCollection With<TWithImplementation>()
    {
        return new InjectedServiceCollection(_Services, _InjectionServices
          .Concat(new[] { typeof(TWithImplementation)}).ToArray());
    }

    private object[] CreateInjectionServices(IServiceProvider provider)
    {
        return _InjectionServices
          .Select(type => ActivatorUtilities.CreateInstance(provider, type))
          .ToArray();
    }
}
```

Here we make it look like you're continuing to add directly to `IServiceCollection` when in fact you're adding to a custom builder class. The benefit to this approach is two-fold:

1. You can inject more than one service into a single implementation, if desired.
2. The intent and limitations are more clear, in that, you cannot use factory implementations after starting the process.

Personally, this is very close to what I need. It is a bit more complicated, and to make the code production-ready would take some more balanced checks. However, the style of coding used within a `ConfigureServices` routine is much nicer.

#### Option #3

This last option is nice from the fact that it is simple. It is really just syntactic sugar on top of `ActivatorUtilities.CreateInstance` for easier use with `IServiceCollection`.

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddSingletonWith<TService, TImpl>(this IServiceCollection services, params Type[] injectedTypes)
        => services.AddSingleton(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TImpl), 
            injectedTypes.CreateInstances(p)));
    public static IServiceCollection AddSingletonWith<TService>(this IServiceCollection services, params Type[] injectedTypes) where TService : class
        => services.AddSingleton(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TService), 
            injectedTypes.CreateInstances(p)));

    public static IServiceCollection AddScopedWith<TService, TImpl>(this IServiceCollection services, params Type[] injectedTypes)
        => services.AddScoped(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TImpl), 
            injectedTypes.CreateInstances(p)));
    public static IServiceCollection AddScopedWith<TService>(this IServiceCollection services, params Type[] injectedTypes) where TService : class
        => services.AddScoped(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TService), 
            injectedTypes.CreateInstances(p)));

    public static IServiceCollection AddTransientWith<TService, TImpl>(this IServiceCollection services, params Type[] injectedTypes)
        => services.AddTransient(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TImpl), 
            injectedTypes.CreateInstances(p)));

    public static IServiceCollection AddTransientWith<TService>(this IServiceCollection services, params Type[] injectedTypes) where TService : class
        => services.AddTransient(typeof(TService), 
          (p) => ActivatorUtilities.CreateInstance(p, typeof(TService), 
            injectedTypes.CreateInstances(p)));

    private static object[] CreateInstances(this Type[] types, IServiceProvider provider)
        => types
            .Select(type => ActivatorUtilities.CreateInstance(provider, type))
            .ToArray();
}
```

Yes, this makes it simple. In fact, as I write this, I realize how simple of a problem this was to solve…

## Conclusion

Ultimately, it is a simple problem to deal with. Any of these options works, with #1 being the worst and #2 being the best. None of them are feature complete and none of them are robust. For instance, what would happen if we want to inject two services of the same type into different constructor arguments? It cannot handle this well, but then again that's not its purpose. For now, it accomplishes exactly what it was intended to accomplish.

Why exactly is this not in a library somewhere?

Microsoft.Extensions.DependencyInjection?

Scrutor?

I don't know. It's simple enough to do on my own. So, I'll just add it when needed. It has only been needed in systems that are built around a Framework (Platform) of reusable code.

Anyways, I hope this low quality article provides some insight into a simple problem to solve in .NET :) Until next time!
