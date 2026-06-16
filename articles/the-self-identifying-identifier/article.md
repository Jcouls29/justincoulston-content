---
title: The Self-Identifying Identifier
summary: Why namespaced, path-style identifiers make data more predictable, reduce join complexity, and unlock scalable patterns in both SQL and NoSQL systems.
category: "C# & .NET"
publishedDate: 2022-12-06
tags:
  - database
  - architecture
  - design-patterns
  - sql
keyTakeaways:
  - An integer identifier is contextless without its surrounding schema; embedding scope directly in the ID makes records self-describing when viewed raw.
  - Slug-based namespaced identifiers are predictable, enabling bulk inserts and multi-table operations without waiting on database-generated sequences.
  - Adding a separate ScopeID column and indexing it allows fast partition-style queries without the performance penalty of full LIKE scans.
  - This pattern shines in NoSQL and Azure Table Storage (PartitionKey/RowKey), but can also stage a SQL schema for a future cloud migration.
draft: false
---

*Let's just make this easier on ourselves, why don't we…*

![Photo by Brett Jordan on Unsplash](https://cdn-images-1.medium.com/max/800/0*YQ4aKiC4-Q7iKn2j)
*Photo by [Brett Jordan](https://unsplash.com/@brett_jordan?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

There comes a point in every programmer journey where you simply wonder "why?" For instance, "Why do I re-write this same routine in every project I create?" or "Why does my boilerplate always take me longer than my core logic?" or "Why am I a programmer still?"

Each of these are valid questions, but one that has baffled me more and more is "Why do identifiers have to be integers?" Or the closely related one, "Why do identifiers have to be random like a GUID?" I mean aren't we trying to retrieve some data back that will be useful to us? Why are we using these random numbers and letters to locate this data?

Well that is a valid question. Here's my take…

## Identifiers and their Purpose

Typically, an identifier (ID) is used for a single purpose: locating a datum, or a set of data, based on a single addressable unit, the ID. There aren't too many other purposes to getting back an ID. You could argue that an ID is used as a summation tool (meaning, all the containing data can be summed by this identifier), or it can be used as a unique classifier of an entity (again for retrieval). But an identifier is about recollection, not identity, of some object.

An identifier, in and of itself, doesn't do us much good without context. For instance, in a traditional sense, an ID would be an `int` or a `bigint` within a database table, let's say `tblUsers`. In this case, the column would be named someting like `Id`, or `UserId`. It would start at `1`, or if you're one of those clever developers, at `1362`. From there it goes up and up and up as your system gains more and more users.

So when you get back this identifier from the system, it will be something like `984`. Let's just hope that you know that this refers to the `UserId` and that it comes from **OUR** table `tblUsers`, and not some other system that maintains users. That `984` is like the house number on your street. You could say that the scope of the **true** ID of this user is `984 UserId, tblUsers, MySystemDatabase, MyCompany`. That is to say, with all the context of the entity we're aiming to load, we must know all of these things to properly get the data back, much like a mailing address. However, I like to call this the identifier's **Scope**. And without scope, you cannot really know what entity in which that `int` refers.

## A Better Way?

This is a loaded sub-header, I know. That's why the question mark must be included. But is there a better way to make identifiers more useful? And if so, in what circumstances?

The reality of the classical `int` ID is that it doesn't take much space, 4 bytes typically. This means to pass this information around, you only need 4 bytes to fully identify the entity. This of course, assumes you know the context. However, when we write applications we can be confident (mostly) that we can keep that straight.

But in our modern world, 4 bytes vs. 256 bytes isn't a world of difference in most business applications. With 5G speeds and 1Gb Home Internet Service, it really wouldn't be noticed adding 252 bytes difference to a payload.

Now that isn't to say that I think we should be flippant about data efficiency, or compression algorithms, or any other means of saving traffic. Not at all. But if we can gain some value out of adding a bit more to our identifiers to lessen costs on the backend, why shouldn't we calculate that trade-off to ensure we have a sufficient ROI?

Let's talk about a more specific example, with a traditional relational database that includes typical SQL joins.

Let's say we have an tournament system where we are staging matches for an organization. We have the following tables:

```
tblOrganizations => Holds Organization Details
tblTournaments => Holds Tournament Master Record
tblMatches => Holds Match data for the tournament
```

Now, let's say we have a query where we want to get back all matches for a specific tournament within our organization. The query might look something like this:

```sql
SELECT M.*
FROM tblMatches M
JOIN tblTournament T
  ON T.TournamentID = M.TournamentID
JOIN tblOrganization O
  ON O.OrganizationID = T.OrganizationID
WHERE
  O.OrganizationID = @UserOrganizationID AND
  T.TournamentID = @TournamentID
```

This really isn't a big deal, but there are some joins that must be done to locate the information we need. In this simple example, we may not have a problem, but what if we could search on a single column? Maybe like the identifier itself? What if we namespaced our identifiers like we would a RESTful service? Can this simplify our system a bit?

Let's explore this a bit.

## Namespaced Identifier

Namespacing has been around a long time. It has helped us identify objects and entities of various kinds for decades. You see it with XML Schemas. You see it with code classes (`System.IO`). You see it everywhere. It is beneficial because we can categorize objects into types.

However, those types of namespaces are typically more static by nature. What if we went a more dynamic route like REST, where the prefix of identifier is the route to get there?

Taking the example above, what if we had the same table, `tblMatches` but we reshaped the ID a bit. Instead of having a `TournamentID` for joining, we change the `MatchID` to be something like this:

`/organization/**sparcpoint**/tournament/**2022-annual-tourney**/match/**a**`

So, what advantage does this add?

First, we don't have to join to another table to perform the search itself. Instead, we can search on the `MatchID` to find the matches we want.

Second, the identifier is self-descriptive. If we were to view the database table raw, we would be able to quickly identify what tournament and organization in which this match belongs.

Now, you may be saying to yourself, "This is great and all, but we use SQL Server and a `varchar` identifier like this will only slow things down, especially if we have to do `LIKE` statements." And you would be correct. If I saw an architect do this directly, I'd have a fit. You cannot make a change like this without having an impact on performance, even if you add indices.

This is why I like the idea of identifiers having a **scope**. A scope is a way to quickly identify where a record belongs in a larger segment. In the case above, I would say the scope for those matches are,

`/organization/**sparcpoint**/tournament/**2022-annual-tourney**`

We would of course, have to add this to our table…

Now our `tblMatches` has `ScopeID = /organization/**sparcpoint**/tournament/**2022-annual-tourney**` and a `MatchID = /organization/**sparcpoint**/tournament/**2022-annual-tourney**/match/**a**`.

Look at how the size of this table is growing from megabytes to gigabytes more quickly than before…

But we could index this table on `ScopeID` and `MatchID` and have an extremely scalable lookup in both cases. We can grab all matches quickly, and grab a single match quickly, even with millions of records in place.

## True Scalability

Let's make an assumption for a minute that storage is cheap, that going from a 10GB ($0.18/month) database to a 100GB ($1.80/month) database won't cost you much more. Let's also assume that you desire high-speed reads and non-contentious writes.

You'll notice that I did NOT use numbers for any of the bold pieces of the identifiers above. Instead, I used slugs for uniqueness. This means that multiple organizations can have the same tournament name. It also means that different tournaments, within the same organization, can have the same match names. **This is important**.

By using slugs, we have created a system of identifiers that are **predictable**. And when you have predictable identifiers, there are so many aspects of saving and loading data that becomes easier: Bulk Inserting, Composing new Entities, Transaction Consistency, Multiple Table Inserts, etc.

If in turn, you add scope, you can now create a suite of queryable tables that can be used to search much more quickly by simply creating many link tables (or if you're bold, a single query table). For instance, we may want to have a tournament tied to a specific game (let's say **Fortnite**). Instead of adding that to our `ScopeID` or `MatchID` directly, we can instead create a table called `tblQuery` that only has three columns: `ScopeID`, `Table`, and `ID`. We can then have entries below tying our tournament to **Fortnite**.

```
// Entry #1
ScopeID = /game/fortnite
Table = tblTournaments
ID = /organization/sparcpoint/tournament/2022-annual-tourney

// Entry #2
ScopeID = /game/fortnite
Table = tblMatches
ID = /organization/sparcpoint/tournament/2022-annual-tourney/match/a
```

Of course, you would index this table for quick access. And yes, you would still need to call the other table, but these queries would be exceptionally fast. You could expand on this table to include scopes with entities like `tags` or `categories` or even other types of metadata, as needed.

The limits of this design is really around your creativity of the queries. When you start doing `LIKE` queries and such, well… you're going to run into the same problems you have today, scanning. But if you can organize your queries into reasonable aspects, most, if not all, queries can run at virtually the same high-speed nature.

## Still Not Convinced?

Well neither am I… at least not for SQL relational tables. Relational tables will operate along the same performance metrics, if not better, if you stick with `int` and `bigint` identifiers while keeping the queries using proper indices. You do lose the predicability, of course, but that may be acceptable.

This is really a pattern more useful in the NoSQL space. A good example of this is Azure's Table Storage. Table storage is built around this concept of `PartitionKey` (`ScopeID`) and `RowKey` (`ID`). So our `tblQuery` would instead be converted to these two columns plus the table name itself. And since storage is cheap, copying the same data multiple times to different partitions is very practical, and recommended. In this way, you can achieve high-speed writes, and reads, while having a cheap source of storage, with a greater flexibility on the software side (I can detail this in another post).

But what if you only have SQL Server, or MySQL to work with? You could, in theory, transition your instance to a NoSQL-style database to prepare for a cloud jump in the future. For example, have this `ScopeID` and `ID` be standard in all tables. Then add a `VARCHAR(MAX)` (or equivalent) to the table to store JSON, XML, or binary. It would be (almost) equivalent to Table Storage from a speed perspective. You could still break out some columns for special cases where you can narrow a query down to the scope then do a scan on the remainder of records. This is something I would approve of from an architectural perspective. Even more so, that `ScopeID` can be used to shard the database across servers, as necessary. Hey… maybe just think about it…

I have used this technique in multiple ways over the last couple years and I have found that knowing the identifier ahead of time, having it be unique (almost) automatically, has made my code, the reasoning around my code, and the stability of the system much greater. This is due to the nature of not having to wait on the database to give me identifiers while also providing a natural RESTful interface to entities. By having this setup, even the web endpoints become naturally dynamic for the basic CRUD aspects because lookup is simply calling to the right table and using the provided path from the URI. A lot of the advantages from this approach is gained in the software development, more than in the database itself, I will admit.

I'm sure there are other things you may be thinking about to work around this idea. It's natural to do this. For instance, what about generating GUIDs on the server? Then you have predictability at least for bulk uploads, inserts, etc. But then you contend with clustered index issues over long spans. Or what about separating out `ScopeID` from the `ID` so the ID is shorter, but then you lose that direct connection within the table.

There are pros and cons to this technique for sure. But at the very least I hope I have provided something to think about when you work on that next project!

Comments commence… I expect this is an extremely debatable topic! So let's get it started!
