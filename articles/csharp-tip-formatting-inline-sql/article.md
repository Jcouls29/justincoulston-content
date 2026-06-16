---
title: "C# Tip: Formatting Inline SQL"
summary: Practical tips for writing readable inline SQL in C# using verbatim strings, keyword capitalization, indentation, and proximity to execution code.
category: "C# & .NET"
publishedDate: 2022-02-10
tags:
  - csharp
  - sql
  - code-quality
  - database
keyTakeaways:
  - Capitalize SQL keywords and break at clause boundaries to make queries scannable at a glance.
  - Use the C# verbatim `@` identifier so multi-line SQL retains proper indentation without escape sequences.
  - Keep the SQL string physically close to the execution call so readers never have to hunt across the file.
  - Break overly complex queries into interpolated segments and use SQL comments to mark logical sections.
draft: false
---

Hey, this is just my way of doing things…

![Photo by Sunder Muthukumaran on Unsplash](https://cdn-images-1.medium.com/max/800/0*0ryIDF0jz9H6kh2w)
*Photo by [Sunder Muthukumaran](https://unsplash.com/@sunder_2k25?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

We all know how important formatting our code is to the quality of our codebase. We have learned the rules of indentation and style from many authors. In this article, I'm going to keep it simple. I'm going to provide some basic tips on formatting Inline SQL (if you're into that kind of thing).

> The first half will be about SQL in general, while the second half will go into more C# specifics.

## SQL

#### Capitalize SQL Keywords

SQL is a very English-esque query language. It is important that you can differentiate between what are SQL words and what are structure / data words.

*Bad*

```sql
select * from instances.productcosthistory where postedtimestamp > '2022-01-01'
```

*Good*

```sql
SELECT * FROM instances.productcosthistory WHERE postedtimestamp > '2022-01-01'
```

#### Break At Keywords

This just helps us locate sections of the query of interest more quickly.

*Bad*

```sql
SELECT * FROM instances.productcosthistory PC JOIN instances.products P ON P.ProductId = PC.ProductId WHERE PC.postedtimestamp > '2022-01-01'
```

*Good*

```sql
SELECT *
FROM instances.productcosthistory PC
JOIN instances.products P
ON P.ProductId = PC.ProductId
WHERE PC.postedtimestamp > '2022-01-01'
```

#### Indent ON Clauses

Helps to keep track of what is a part of the main keyword (or Join in this case)

```sql
SELECT *
FROM instances.productcosthistory PC
JOIN instances.products P
    ON P.ProductId = PC.ProductId
JOIN definitions.products PD
    ON PD.DefinitionId = P.DefinitionId
WHERE PC.postedtimestamp > '2022-01-01'
```

#### Please, Just Use Square Brackets

This one is probably the most controversial, but use brackets for anything defined in the database (i.e. Schema, Table, Column). It makes identifying these objects much easier in a query. However, any temporary "variables" I will not typically bracket.

*Bad*

```sql
SELECT *
FROM instances.productcosthistory PC
JOIN instances.products P
    ON P.ProductId = PC.ProductId
JOIN definitions.products PD
    ON PD.DefinitionId = P.DefinitionId
WHERE PC.postedtimestamp > '2022-01-01'
```

*Good*

```sql
SELECT *
FROM [Instances].[ProductCostHistory] PC
JOIN [Instances].[Products] P
    ON P.[ProductId] = PC.[ProductId]
JOIN [Definitions].[Products] PD
    ON PD.[DefinitionId] = P.[DefinitionId]
WHERE PC.[PostedTimestamp] > '2022-01-01'
```

#### Indent Multiple Conditions or Parameters

If we have a SELECT, WHERE, JOIN or anything with multiple clauses, add a line break and perform an indent. Basically, let's keep 1 line to one piece of information.

```sql
SELECT
    P.[ProductId],
    P.[Name],
    P.[Description]
FROM instances.productcosthistory PC
JOIN instances.products P
    ON P.ProductId = PC.ProductId AND
       P.CreatedTimestamp = PC.CreatedTimestamp
JOIN Definitions.Products PD
    ON PD.DefinitionId = P.DefinitionId
WHERE
    PC.PostedTimestamp > '2022-01-01' AND
    PD.Type = 0
```

This also applies to updates as well,

```sql
UPDATE [dbo].[KeyValues]
SET [Key] = 'Color',
    [Value] = 'Blue'
WHERE
    [Key] = 'Color' AND
    [Value] = 'Red'
```

#### Line Up Parameters in Insert Statements

For me, inserts are a bit different. You will likely have more fields to insert than select or update. So, do we really want a SQL statement that is 25 lines long? Probably not. So, instead of putting each parameter on its own line, let's put line breaks at reasonable horizontal widths and make sure to keep the parameters aligned.

```sql
INSERT INTO [Snapshots].[Snapshots]
    ([TargetTimestamp], [CreatedTimestamp], [SnapshotType],
     [Notes], [AssociatedUserId])
VALUES
    (@TargetTimestamp, @CreatedTimestamp, @SnapshotType,
     @Notes, @AssociatedUserId)
```

The same applies when inserting with a SELECT statement,

```sql
INSERT INTO [Snapshots].[Snapshots]
    ([TargetTimestamp], [CreatedTimestamp], [SnapshotType],
     [Notes], [AssociatedUserId])
SELECT
    [TargetTimestamp], @CreatedTimestamp, [SnapshotType],
    'Cloned Snapshot', @AssociatedUserId
FROM [Snapshots].[Snapshots]
WHERE [SnapshotId] = 4660
```

#### Separate Compound Queries by a Line and a Semicolon

This is a personal preference, but since you need a semicolon anyways, why not just use it between calls to clearly identify each statement?

```sql
DECLARE @Ids TABLE ([Id] INT NOT NULL)
;
INSERT INTO @Ids ([Id])
SELECT S.[SnapshotId]
FROM [Snapshots].[Snapshots] S
JOIN [Snapshots].[InventoryTransactions] IT
    ON IT.[SnapshotId] = S.[SnapshotId]
;
SELECT COUNT(*)
FROM [Snapshots].[Snapshots]
WHERE [SnapshotId] IN (SELECT [Id] FROM @Ids)
;
SELECT *
FROM [Snapshots].[Snapshots]
WHERE [SnapshotId] IN (SELECT [Id] FROM @Ids)
;
```

> **Clarification**: we don't actually HAVE to use semicolons (with T-SQL specifically). Most statements do not require it. However, there are a few that do (i.e. CTE Statements). Reality is that semicolons are in the ANSI SQL-92 standard, so I use them.

#### Indent SQL Blocks

When statements are grouped together with a block (i.e. `BEGIN TRANS` or `IF`) then indent all code within that block.

```sql
SET XACT_ABORT ON;
BEGIN TRANSACTION;
    INSERT INTO [Snapshots].[Snapshots]
        ([TargetTimestamp], [CreatedTimestamp], [SnapshotType],
         [Notes], [AssociatedUserId])
    VALUES
        (@targetTimestamp, @createdTimestamp, @snapshotType,
         @notes, @associatedUserId)
    ;
    DECLARE @SnapshotId INT = SCOPE_IDENTITY()
    ;
    INSERT INTO [Snapshots].[InventoryTransactions]
        ([SnapshotId], [Quantity], [ProductInstanceId],
         [InventoryLocationId])
    SELECT
         @SnapshotId, SUM(IT.[Quantity]), IT.[ProductInstanceId],
         IT.[InventoryLocationId]
    FROM [Transactions].[InventoryTransactions] IT
    GROUP BY
        IT.[ProductInstanceId],
        IT.[InventoryLocationId]
    ;
COMMIT;
```

```sql
SELECT *
FROM [Snapshots].[Snapshots]
WHERE [SnapshotId] = @SnapshotId
;
```

> **Note**: I put my semicolons at the end of `BEGIN TRANSACTION` and `COMMIT` on purpose here. This is already a block statement, and it is already visually isolated, so I do not see a reason to waste the line.

## C#

Now let's get to the C# portion of the article. All of the rules above apply, but we will now show them in C# and provide some readability tips.

#### Use the Verbatim @ Identifier

By using the [verbatim identifier](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/verbatim) when creating a string you give yourself a lot more flexibility on the style of your SQL. Everything we have shown above will work in these string literals.

<script src="https://gist.github.com/c364da869778947b3df5c9666ab4efc4.js"></script>

#### Always Start on a New Line

I think this one is obvious if you think about it.

<script src="https://gist.github.com/05459b82bf98470d4d3be4fa4af2ca24.js"></script>

#### Indent the SQL Once Past the Start of the Declaration

Just like with any other related code, indent past the declaration.

<script src="https://gist.github.com/cb69a2789fbf72bf42e2f4d92f28a921.js"></script>

> **Bonus Tip**: Always put the last quote and semicolon on a separate line and make sure it lined up with the declaration as shown above.

#### Keep The SQL Close to the Execution Code

Don't do the following,

<script src="https://gist.github.com/4e849e966be1eb80c32091385ceced5b.js"></script>

Here we have separated the SQL to the top of the class as constants. I think folks do this because it is "cleaner." But the whole point of what we're doing is to make it more readable so the code can be understood more easily. By having it away from the executing code, now we have to go find it. And when these classes get large, it gets frustrating trying to jump back and forth between the top of the class and the method.

Instead, let's do something like the following,

<script src="https://gist.github.com/d0ca5211a8aec4f5fff890eeef926eb5.js"></script>

To be honest, this probably could be more readable if it was not a lambda-style method. But still, it is closer to the execution so I can quickly see what is happening here, in the method. It's in one spot.

#### Break the Query Apart if it Gets Too Complicated

Sometimes queries are a monster to handle, especially in line with C# code. In my case I have some "providers" I use to more easily build up a query. I will utilize the interpolated verbatim combination, when necessary, to break up the complicated SQL,

<script src="https://gist.github.com/847e36661d2d653b5a3575909a8e2739.js"></script>

If you look closely, you'll see `{provider.JoinClause}` and `{provider.WhereClause}`. Granted, these clauses are dynamic as well, so this is the only way for me to accomplish the task without extremely complicated queries. But there are occasions where this is helpful from a readability standpoint. For me, it helps to say, "*Hey this part might be complex, let me go look at that specific logic*" rather than trying to parse it all in one giant SQL statement.

#### Put in Comments When Helpful

<script src="https://gist.github.com/e638074dd68255eebe9c8f6edb272e68.js"></script>

Sometimes you need to know where certain queries start and stop, and what section you're in. For instance, this query has it broken up by different inventory counts.

And Alternative to the above (that breaks some of my previous rules), may look cleaner to you. The following is the exact same query formatted differently,

<script src="https://gist.github.com/9c7da53d324d7d8323cd0629f10d9de2.js"></script>

I think this looks cleaner (sometimes; depends on my mood) because it contains less lines of code. But I'll let you decide.

## Conclusion and Sign-Off

Hopefully some of these tips give you some ideas of writing cleaner Inline SQL in your C# Application. Ultimately, it is for you to decide what is better from a readability standpoint. I think you should try to take your fellow developer into account when you're writing this code. It will always be easier for you to read (because you wrote it). But your partners in crime might not feel the same way.

Let me know what you think! Please **Follow Me** here on Medium. It helps encourage me to continue writing!

*I think, therefore I am, a buggy coder…*
