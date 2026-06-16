---
title: "Following the Rules as a Developer"
summary: Uses the Dreyfus model of skill acquisition to argue that coding rules like YAGNI and KISS are essential guardrails for beginners but become flexible guidelines for proficient and expert developers who can weigh the cost of change.
category: "Developer Practices"
publishedDate: 2022-02-09
tags:
  - career
  - code-quality
  - architecture
  - design-patterns
keyTakeaways:
  - Rules like YAGNI, KISS, and SOLID exist to guide novice and intermediate developers; proficient and expert practitioners should treat them as flexible guidelines, not gospel.
  - The right question when deciding to break a rule is not "might we need this?" but "what is the cost — in time and future effort — of doing this versus not doing it?"
  - True cost in software development is measured in time, not money; code that works, is tested, and never has to be maintained is the highest-value outcome.
  - Self-reflection on your current Dreyfus stage is critical — overconfident competents apply rules dogmatically, while experts apply them selectively based on deep pattern recognition.
draft: false
---

Now, what I'm about to share may not be something we can all agree on. If you aren't the type of person to challenge the status quo, or worse, cannot reflect on your own abilities in an honest way, then I would recommend turning around now and forgetting you ever saw this post. This article is about how some of the "rules" we follow as gospel in the development world SHOULD be broken. The question really revolves around WHEN we should break them.

This won't be an exhaustive guide necessarily. It's more of an opinion piece. And on top of that, this is a piece that requires self-reflection. I cannot tell you who you are and when these instances are appropriate. This is a matter of skill and experience.

And on that note, I want to talk about skills first.

## The Dreyfus Model

> The **Dreyfus model of skill acquisition** is a model of how learners acquire [skills](https://en.wikipedia.org/wiki/Skill) through formal instruction and practicing, used in the fields of [education](https://en.wikipedia.org/wiki/Education) and [operations research](https://en.wikipedia.org/wiki/Operations_research). Brothers [Stuart](https://en.wikipedia.org/wiki/Stuart_Dreyfus) and [Hubert Dreyfus](https://en.wikipedia.org/wiki/Hubert_Dreyfus) proposed the model in 1980 in an 18-page report on their research at the [University of California, Berkeley](https://en.wikipedia.org/wiki/University_of_California,_Berkeley), Operations Research Center for the [United States Air Force](https://en.wikipedia.org/wiki/United_States_Air_Force) Office of Scientific Research. The model proposes that a student passes through five distinct stages and was originally determined as: novice, competence, proficiency, expertise, and mastery.
>
> [- Wikipedia](https://en.wikipedia.org/wiki/Dreyfus_model_of_skill_acquisition)

The Dreyfus model is a pretty good guide to an individual's skills. It gives a means to measure ourselves against our abilities by taking mental functions and categorizing them appropriately. Although no model is perfect, I do feel that I have seen this model applied in practice more than once.

We all know that "years of experience" is the worst indicator of skill level. It may inform us of the potential of an individual, but it isn't a guarantee. What I mean by this is that a developer that started coding 6 months ago is very unlikely to have acquired a mastery in the art of software development. In fact, they're just getting started. However, 20 years of experience could be as good as 1 year of experience repeated 20 times. There is no guarantee that because a developer has been working for 20 years, that they have mastery over the skill.

Although, no one is certain of the origin of the quote, a popular rendition is from [Shibumi](https://amzn.to/3J2y3l4),

> *"You can gain experience, if you are careful to avoid empty redundancy. Do not fall into the error of the artisan who boasts of **twenty years experience** in craft while in fact he has had only **one year of experience–twenty times**. And never resent the advantage of experience your elders have. Recall that they have paid for this experience in the coin of life, and have emptied a purse that cannot be refilled."*

> [- Trevanian, Shibumi](https://www.aamboli.com/quotes/book/shibumi)

But I digress. The Dreyfus Model is an interesting concept. It can actually help when communicating with developers and understanding how they adapt to various situations through the skill growth cycle. Different individuals are in various seasons with their experience, and we need to gauge this level to allow us to discuss and train them in specific topics. It can also inform us when they are not exactly ready for a specific methodology or technique. It can also inform us of what we need to learn to continue on our path of mastery.

The following quotes come from an awesome book on the topic I bought 5 years ago, that I highly recommend, called [Pragmatic Thinking and Learning: Refactor Your Wetware](https://amzn.to/3slkkz0) by Andy Hunt. He walks through learning and the Dreyfus Model as it applies to programming (and a bunch of other topics). He is the same author of [The Pragmatic Programmer](https://amzn.to/3opIE1G) (along with David Thomas).

Let's walk through the five stages of the Dreyfus Model as defined by Andy.

#### Stage 1: Novices (Beginning)

> Novices are very concerned about their ability to succeed; with little experience to guide them, they really don't know whether their actions will all turn out OK. Novices don't particularly want to learn; they just want to accomplish an immediate goal. They do not know how to respond to mistakes and so are fairly vulnerable to confusion when things go awry.

> **Novices need recipes.**

> - Pragmatic Thinking and Learning: Refactor Your Wetware

The important aspect here is that novices (or beginners) need **rules**. They need tutorials. They need guidance. And this is natural. When you first start programming, you're looking up tutorials, buying books that teach For-Loops, If-Then branching, and other keywords of the language. Rules provide the initial guidance for any developer at the start of their journey.

#### Stage 2: Advanced Beginners

> Once past the hurdles of the novice, one begins to see the problems from the viewpoint of the advanced beginner. Advanced beginners can start to break away from the fixed rule set a little bit. They can try tasks on their own, but they still have difficulty troubleshooting.

> **Advanced beginners don't want the big picture**

> - Pragmatic Thinking and Learning: Refactor Your Wetware

At this point, your skills are advancing but you still pour over documentation and have to look up some tasks, but you can start to do your job without too much guidance assuming the tasks are fairly basic, or repetitive. They can use advice from others, but it will still be based on past experiences.

#### Stage 3: Competent

> At this third stage, practitioners can now develop conceptual models of the problem domain and work with those models effectively. They can troubleshoot problems on their own and begin to figure out novel problems-ones they haven't faced before. They can begin to seek out and apply advice from experts and use it effectively.

> **Competents can troubleshoot.**

> - Pragmatic Thinking and Learning: Refactor Your Wetware

At this stage, developers are adept at getting things done. But this is also where developers lack self-reflection about their abilities and I have found a lot of developers sit here for a long time if they ever leave.

#### Stage 4: Proficient

> Proficient practitioners need the big picture. They will seek out and want to understand the larger conceptual framework around this skill. They will be very frustrated by oversimplified information.

> **Proficient practitioners can self-correct.**

> - Pragmatic Thinking and Learning: Refactor Your Wetware

Here we see developers really take hold of their skills and focus on improving. They have the ability to take practical advice and learn from others without having to directly experience failure. They watch and "self-correct" as failures occur while having the ability to study.

The thought of "oversimplified information" reminds me of folks getting frustrated (on this platform) with basic articles on development. Things that are somewhat obvious or poorly described.

#### Stage 5: Expert (Mastery)

> Experts are the primary sources of knowledge and information in any field…The expert knows the difference between irrelevant details and the very important details, perhaps not on a conscious level, but the expert knows which details to focus on and which details can be safely ignored. The expert is very good at targeted, focused pattern matching.

> **Experts work from intuition**

> - Pragmatic Thinking and Learning: Refactor Your Wetware

And this is the last stop. Experts are not common. Approximately 1 to 5% of the population is considered expert in any given skill (J.T. Hackos and D.M. Stevens. *Standards for Online Communication*. John Wiley & Sons, New York, NY, 1997).

This is both encouraging and disheartening.

Am I an expert? Are you? Can we become experts? Here is my last quote from the book.

> When you are not very skilled in some area, you are more likely to think you're actually pretty expert at it.

> In the paper, Unskilled and Unaware of It: How Difficulties in Recognizing One's Own Incompetence Lead to Inflated Self-Assessments (Journal of Personality and Social Psychology. 77[6]:1121–1134, 1999)…

> - Pragmatic Thinking and Learning: Refactor Your Wetware

The last quote is more about us. In order for us to properly evaluate where we are, we must self-reflect. This is key in software, relationships, and in life. It is critical to our growth.

We should attempt to be aware of which category we fall into as it directly affects how we learn and our ability to achieve mastery.

## Why the Rules Exist

Rules exist for beginners, advanced beginners and competents. They don't really exist for proficients and experts.

Why?

Because **"Rules ruin experts."**

As mentioned before, rules are in place to help guide developers down a path of best practices. And in most cases, experts came up with these rules, for the very purpose of guiding us, the developer, along our path to mastery. But before the rules existed, it had to be discovered by said "experts." They put the work, time, and "emptied the purse of life" for our sake.

Now, when I say rules, what I really mean are all the things we have been taught over the years. And as you read this list, understand that I agree with these "rules" of development. But we should recognize that they act as "guidelines" at a certain point in your journey.

- **YAGNI — You Aren't Gonna Need It**: We should presume that some capabilities should not be built now because we don't really know if we'll need it.
- **KISS — Keep it Simple "Silly"**: Simplicity should be a key goal in any design
- **Single Responsibility Principle**: There should never be more than one reason for a class to change.
- **Open-Closed Principle**: Software Entities…should be open for extension, but closed for modification
- **Dependency Inversion Principle**: Depend upon abstractions, not concretions.
- **Composition over Inheritance**: Achieving polymorphic behavior and reuse through composing objects rather than inheriting from base classes.

And the list goes on.

I will reiterate, that these are all important and critical principles in software development, and in no way am I suggesting that we should suddenly STOP using them. In fact, even if I was suggesting that, and you trusted my every word, you would be violating the very thing I'm trying to express…**Rules ALWAYS have exceptions**.

## Exception to the Rule

Let's take a simple example for now. A popular one I hear is the YAGNI principle. Typically, it goes something like this…

> **Developer A**: "Do we really need to worry about adding a table right now to track performance metrics?"

> **Developer B**: "Well, what happens when we start having performance problems 6 months from now and we have no idea why?"

> **Developer A**: "When that happens, we'll put the feature in, but we don't really know if we'll have any issues."

> **Developer B**: "I guess that's true, but then we'll have to re-build the code and deploy."

> **Developer A**: "Maybe, but I think YAGNI. Why put the work in now? I think we should just wait"

> **Developer B**: "Ok. That makes sense. It might be wasted effort."

Who's right in this situation?

In my opinion, neither of them. The reason. No one really has a good argument. YAGNI isn't an argument, in my humble opinion, ever. Why won't you need it? Give me a better reason than "We might not need it." But on the other side, building / deploying code isn't a good argument either in this context.

Why?

Because no one discussed cost. I think **Developer B** was definitely closer, but it depends on the cost of re-building and deploying the code. If building and deploying the code requires hours of work, a team of people to deploy, and multiple QA iterations, then **Developer B** was right.

However, if code can be re-built and deployed within minutes, while being fully tested, then **Developer A** was right.

In this imagined scenario, **Developer A** is an idealist and **Developer B** is fearful of the future; both of which are probably competent (or below) in their skills. Neither is what we want to be. We want to be pragmatic in our approach. This is where a proficient (or an expert) play the part. This is where intuition and past experience is required to make judgement calls.

In this case, the "rule" we're considering breaking is YAGNI. Should we? We don't know from this example. But we can say that there may be cases where breaking YAGNI makes complete sense, both from a business perspective, and developer efficiency perspective.

So, can I give you a hard and fast rule about WHEN to break a rule? No, not really, but I can provide a bit of guidance. If ever in doubt,

> Base a decision on cost.

But what is "cost?" Let's talk about that for a minute.

## The Cost of Development

Cost is usually associated with Money ($$$). However, this is a cheap approach to the topic (pun intended).

> No. Cost is really about time.

It's our most valuable asset. Naturally, time can be translated to money, but more to the point, we shouldn't waste what little time we have in this world on repetitive tasks.

So, when I think cost, I think of time it takes to perform some tasks, and the efficiency of any effort expended (say that 5 times fast). The more efficient a task becomes, the less time it will take, and in turn, the most value we get out of performing it. For this reason, I will break ANY rule that gets in the way of efficiency.

My skill is software development, and 80% of the time, I follow all the rules above. However, occasionally, I develop a method, class, or framework that breaks a couple of these rules. This occurs most often when I write a sub-system that I do not want to work on EVER AGAIN.

Some of these principles are built around the idea that we're going to be touching the code over and over again. For the sake of readability, and simplicity, we follow these rules to ensure that other developers can understand the purpose. But what's better than code that works, is tested, and is easy to maintain?

> Code that works, is tested, and never has to be maintained.

This doesn't happen often, but occasionally, you'll stumble upon a problem, solve it in a novel way, and it will work over and over again. You then decide to push that code into a library that you can re-use over and over again. And then years will go by before ever having to look at that code. And yes, I have experienced this, although not often. I find there is a greater chance of this occurring when you do consider the future and potential changes; especially when you understand the business and how they operate.

#### Personal Example

Many years ago, I helped develop a payments system for a company I was working for. The boss had very specific needs at the time, and we needed to make those requirements happen. However, I knew this guy well and understood that he would eventually want some additional changes that weren't being considered because it wasn't a need at the time.

But I knew better.

The idea of YAGNI was thrown around a lot. However, I was able to win out the argument, mind you, with a lot of self-doubt along the way. The idea was around adding additional flexibility to the system "just in case."

Without going into too much detail, lo and behold, not 8 months later, he asked for a change that, although not exactly what I imagined, could be handled by the system. No changes actually necessary. He was, in fact surprised we could handle, what he thought, was huge requirement changes without redeploying the code.

I admit I took a risk pushing against senior developers at the time, but it worked out in the end. And to be completely honest, after about 3 years of the code running without any changes, we did finally get a requirement we couldn't handle. We had to make a change to code and structure.

But man was it a good run, especially for what we built.

Although, it wasn't perfect, the point is, the code worked for years handling new requirements without ever needing to change. We simply had to configure it a bit different through the UI, which was exactly the point. The code wasn't pretty, and I was by no means an expert, but after experiencing that forethought, it clicked, and I understood that sometimes rules can be broken and can benefit the business and the developers.

You have to remember, by not having to work on that code, I was able to work on other applications critical to the business, instead of maintaining this one. And from a cost perspective, it was extremely low cost. The amount of time to add the flexibility was approximately an additional 2 weeks of work upfront. With no cost for almost 3 years. That's a win in my book.

## Conclusion and Sign-Off

It is possible to break the rules and add value to the company, or product you're working on. I have experienced this. And although folks might balk at this in principle, it's a matter of what I value most: my time. And by saving my time, I am able to keep the cost of software low, while still innovating in the field.

As a beginner developer, I would recommend that you continue to follow the "rules" to the best of your ability and master them. Without understanding why they're in place, you'll struggle to understand when they should be broken. The example I presented was an instance where I broke "YAGNI," however, there have been cases where I have broken "Composition over Inheritance," "KISS," and others. It's never haphazard, nor is it out of laziness. Instead, it comes down to cost. And cost, and efficiency, are my driving forces. If I can build software that I do not need to maintain, that's a win. Yes, eventually you will most likely have to touch the code, so keeping it clean, easy to read are still critical, but exceptions may be necessary from time to time to achieve the goal of keeping your time expended on a problem to a minimum.

As you move closer and closer to becoming an "expert" in your craft, these decisions inevitably become clearer. And sometimes, it may be that you have to experiment with different decisions before understanding why something does and doesn't work. But that's what you must do to become and best developer you can become. Be experimental. Be humble. And above all else, be efficient.

*If you liked this article, follow me here on medium. It encourages me to continue writing!*

To Infinity and Beyond, *the basics of coding*!
