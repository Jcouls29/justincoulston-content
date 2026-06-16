---
title: "Let's Do Better with Log Messages, Pretty Please"
summary: A practical case for designing logging and exception handling strategies early, with context-rich messages that help developers self-correct production issues.
category: "C# & .NET"
publishedDate: 2022-12-28
tags:
  - dotnet
  - logging
  - csharp
  - code-quality
keyTakeaways:
  - Define your exception handling and logging strategy at the start of a project, not as an afterthought when bugs surface in production.
  - Catch exceptions at the outermost interaction boundary and re-throw with enriched context—configuration keys, valid value ranges, and actionable guidance—so the first responder can self-correct without a developer.
  - Structured logging with a scoped ambient context object lets you attach request parameters to error logs without scattering try/catch blocks throughout the call stack.
  - "Thoughtful log messages are a selfish investment: the time you spend writing them now will be paid back the first time you can diagnose a production issue from a Teams message alone."
draft: false
---

Can we focus on some core aspects of software early on, rather than waiting to the end?

![Photo by Alex Motoc on Unsplash](https://cdn-images-1.medium.com/max/800/0*5ClAiH5nTQLl3Rqe)
*Photo by [Alex Motoc](https://unsplash.com/@alexmotoc?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Have you ever been in that situation where your boss asks you to look at some code written 5 years ago by a developer no longer with the company and fix an issue? Have you ever noticed that 9 times out of 10, figuring out how to debug the application, find the logs and understand them takes up 80% of your time? Then you have to actually fix the issue?

Yea. This has happened to me time and time again…

So I began to ponder why this occurs. Why is it that every application requires us to fight for hours just to understand the basics of its inner workings? Why does it seem to happen with our own applications that we have written? I mean, I have forgotten how my own stuff works and couldn't remember where the logs were, how to view them, and read obscure, meaningless entries. It's a little embarrassing going back on my own code and realizing I have forgotten what an obscure error message means.

However, experience has taught me a few things over the years about how to make my life, but also other lives easier, when the time comes to debug our code and maintain an application throughout its lifetime.

## Maintenance-First Coding

So I'm not coming up with some new found "paradigm" to coding like "test-driven development" or anything like that. All I am saying is that maybe we should focus on how we're going to maintain our code ahead of time. What is our strategy going to be? This is an all too critical concept that feels bolted on too late in the game. Here are some questions that we should be asking ourselves early on when working on a green field project.

- Where will exceptions be caught in the application?
- How will exceptions be trapped, or bubbled, to the user of the application?
- Where will information be logged within our system? Will it be a log file or console only? What type of information will we provide?
- What type of context will be provided when issues do arise?
- Who will be the first responder to issues? Are there ways for us to provide them enough information to self-correct?

Issues within a system inevitably always happen. Someone is GOING to find a bug in your application. I can guarantee it. Even if you programmed it perfectly, someone will make up a "logical" bug because they gave you bad requirements… I've seen it happen. So we must be prepared to track these issues down and correct them… And quickly.

## Maintenance Goals

So what is the real goal we should be aiming for when we talk about maintaining an application (at least from the perspective of externally to the application)?

#### 1. Any issue that occurs within the system should be immediately understood from the messaging.

When something goes wrong, we should be able to know exactly what went wrong and where to fix it. Let's give an example,

If you work with SQL Server in .NET at all, you have seen this error from time to time:

> "An error has occurred while establishing a connection to the server. When connecting to SQL Server, this failure may be caused by the fact that under the default settings SQL Server does not allow remote connections. (provider: Named Pipes Provider, error: 40 — Could not open a connection to SQL Server ) (.Net SqlClient Data Provider)."

As a developer, you know exactly what this means, right? It basically says that (most likely) the connection string is incorrect. It could also mean that there is a firewall issue, but let's assume your network guys got it right.

If someone was to see this in a log, they may be able to quickly determine this as well. However, they may NOT know how to correct the issue. For instance, they may not know that the connection string is stored in `appsettings.json` or an `environment variable`. So now they go around hunting for the connection string. Wouldn't it be nicer if the exception looked more like this:

```csharp
try {
  // ... Sql Connection Code ...
} catch (SqlException ex) when (ex.Number == 53) {
  // https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/cc645611(v=sql.105)
  throw new InternalSqlException(@"
    Could not connect to Sql Server. 
    Please check the connection string within the 'appsettings.json' file 
    (Key: ConnectionStrings:Primary)", ex
  );
}
```

This tells me exactly where to look for the connection string, and what to check. Is this perfect? No. But it provides a bit of context to the user to check their setup and configuration before running back to the developer.

Another common example I like to use is required configuration. Many times there can be configurations for connection strings, credentials, feature flags, and more in a settings file. And depending on the type of configuration, we may want to validate (on startup) whether things are configured correctly.

So, let's say we have a facial recognition feature we want to enable. It requires a confidence level double to be defined. We want to make sure it is properly defined before starting up, otherwise we blow everything up. I like to do this so that during deployment we know immediately if there is an issue.

```csharp
public class FacialRecognitionFeature {
  public bool Enabled { get; set; } = false;
  public double? ConfidenceLevel { get; set; } = null;
}

public static class ConfigurationValidation {
  public static void Validate(FacialRecognitionFeature? feature) {
    if (feature?.Enabled == false)
      return;
    
    if (feature.ConfidenceLevel == null)
      throw new ValidationException(@"
        Facial Recognition Confidence Level is required.
        The confidence level must be between 0.01 and 0.99.
        (Key: Features:FacialRecognition:ConfidenceLevel)
      ", "Features:FacialRecognition:ConfidenceLevel");

    if (feature.ConfidenceLevel < 0.01 || feature.ConfidenceLevel > 0.99)
      throw new ValidationException(@"
        The confidence level must be between 0.01 and 0.99.
        (Key: Features:FacialRecognition:ConfidenceLevel)
      ", "Features:FacialRecognition:ConfidenceLevel");
  }
}
```

I think this is much better than simply saying something like `throw new ArgumentException("Incorrect value for the Confidence Level")` or some simple equivalent. It doesn't tell you where to look or what the values should be.

Now you might be saying, "Duh Coulston, this is obvious." But I never seem to find code like this that helps future developers and maintainers maximum their debugging of startup issues. And let's be honest, it's not easy. Well… it is easy, but we tend to be lazy right? We put it off too long and don't put the thought in until someone STARTS to have issues. I think some people feel like it isn't necessary up front. But if you could predict what would be needed in the future, we wouldn't have this problem to begin with… So why not do it now?

#### 2. (Opinion) Exceptions should be caught immediately before leaving the system.

This one might be a bit subjective, but here we go.

I believe that exception handling anywhere but the point of interaction with external users (or other systems) is superfluous. For example, I strongly push back on catching exceptions within class libraries. There are only a few special occasions I believe this is ok, and necessary, but not within the norm. Instead, they should be caught at the interaction point of the user or external system.

Now in #1, I showed exactly this. I caught an exception, but immediately re-threw it. This is one of those exceptions to the rule (no pun intended). I'm not catching, logging the exception, and not telling the stack something happened? No. I'm capturing it, then passing it along with more information. In this way, I'm still letting the top-level interface catch the exception.

For example,

```csharp
// Windows Forms
private void btnSubmit_Click(object sender, MouseClickEvent e) {
  try {
    // ... Button Logic ...
  } catch (Exception ex) {
    Logger.LogError(ex);
  }
}

// ASP.NET Core
[HttpPost("api/v1/submissions")]
public IActionResult SubmitForm([FromBody] Submission submission) {
  try {
    // ... Controller Logic ...
  } catch (Exception ex) {
    Logger.LogError(ex);
  }
}
```

To be clear, I am showing these examples as a point of clarification. However, I would typically abstract this try-catch portion out into common handlers that can be shared through Attributes or global handlers when possible. This of course assumes it will not crash the app unnecessarily.

The primary reasoning for this is that we can maintain our stack trace. If you throw and catch up and down the stack, you "can" lose information if done improperly. Plus you have to worry about exception handling in too many spots to be effective, in my opinion. Focus your exception handling around a smaller portion of code and you can be sure that handling is done in a common way.

#### 3. Structured Logging is your Friend

Sometimes the exception isn't enough information. Especially if you're not catching the exception where the context is generated. For instance, parameters going into a SQL statement may cause a "truncation" error, but you don't know which one. How then can we capture this context and add it to our logs in a way to be useful?

One thing I hate seeing is extraneous error messages that fill up logs and are never used (you know like `LogInformation` statements everywhere). I'd rather have the information I need show, when I need it, and only then. This requires two aspects to logging:

1. Capturing the context
2. Adding the context to the exception log

If using Serilog, #2 is fairly straightforward,

```csharp
Logger.LogError("Error Occurred: {Exception}.{NewLine}Context: {@Context}", 
  ex.ToString(), ContextData);
```

This will show the full stack trace of the exception and a JSON interpretation of the context object, given the JsonFormatter is used. Pretty useful.

However, #1 is a bit more difficult. There are really three ways, that I can think of that this can be handled:

1. Use the `Logger.BeginScope` built into .NET Core. This allows for additional data to always be logged with any `Log{Level}` statement made. However, it only works within scope and so bubbling up doesn't really work. Plus only certain Log Providers support it.
2. Use a custom exception to bubble this information up. You can add context to a dictionary within a custom exception so that it can reach your outer catch statement. However, this means breaking my #2 rule of only catching in one spot. You'll need to catch and re-throw for this to work.
3. An ambient context handler…

So, how do we solve this problem? Really #3 is the best way, unfortunately. We'll have to create a custom (scoped) handler that maintains parameters and context information throughout the life of a function call, and subsequently destroy it at the end of the scope. The downside to this is it really only works "easily" within a web application. For other types, you'd have to build this DI mechanism in yourself.

The following example is incomplete, but can provide an idea of what I'm talking about,

```csharp
public interface IFunctionContext {
  void Add(string key, object value);
  IReadonlyDictionary<string, object> Context { get; }
}

public void ConfigureServices(IServiceCollection services) {
  // Will leave the InMemoryFunctionContext implementation to your imagination
  services.AddScoped<IFunctionContext, InMemoryFunctionContext>();
}

public class BasicController {
  private readonly IFunctionContext _Context;

  public BasicController(IFunctionContext context) {
    _Context = context;
  }

  public IActionResult Post(Submission model) {
    try {
      _Context.Add("Submission", model);
      // ... Business Logic...
    } catch (Exception ex) {
      Logger.LogError("Error: {Exception}{NewLine}Context: {Context}",
        ex.ToString(), _Context.Context);
    }
  }
}

public class SqlService {
  private readonly IFunctionContext _Context;

  public SqlService(IFunctionContext context) {
    _Context = context;
  }

  public DataObject RunQuery(Parameters parameters) {
    _Context.Add("SqlParameters", parameters);
    // ... SQL Query ...
    return data;
  }
}
```

You can see fairly quickly how this can become tedious. Not really, an ideal solution is it? However, I hope you can also see the value in this approach. By providing more context, errors will actually mean more, lowering the time to a solution. Otherwise, you may end up having to get a production database pulled down, open up Visual Studio, start the debugger, add some breakpoints, etc. etc. Whereas if you have this type of context in place, none of that may be necessary. It may be as simple as adding the appropriate checks on the data, assuming you logged that information.

But then again, you also have to worry about what type of data you're logging. Is there PII data, passwords, or other sensitive data in your parameters? Now you have to be wary of this as well.

There are approaches to simplifying, and centralizing some of this logic so it doesn't have to weave an enormous web through your application. But that's best to be discussed in another article. Regardless, it has to be thought through, especially if you have a forensics team separate from your development team that helps track issues down in production.

## Conclusion

Ultimately, my goal in this article is to say this,

> Put some focus on how you write log messages, in files or otherwise, and who is going to read them. Make it easy to understand, locate the issue, and self-correct, when possible.

Take it from a selfish position, if you must. Forget those other people. Wouldn't it make your own life easier, if you could look at a log file and know immediately the issue? Isn't that better than those `NullReferenceException` that we always get? The moments I have taken to do this, has helped tremendously. In many cases, I will have the exception sent to me directly in a Teams message (or email), see it and respond with the solution without ever having to look at the code. That's because I added enough context to be useful, at the very least, to myself.

So save yourself some time and effort. Make it so others can debug their own problems and you'll see a huge difference in how people operate with your software!

Until next time!
