---
title: "Advanced C# — Common SQL Table Structures"
summary: How to design a shared base table structure for multiple entity types and build a generic C# repository framework around it using Dapper, custom attributes, and a SQL statement builder.
category: "C# & .NET"
publishedDate: 2022-02-02
tags:
  - csharp
  - sql
  - database
  - design-patterns
keyTakeaways:
  - A common base model (InstanceId, Name, Description, Categories, CustomAttributes) shared across entity types allows a single generic repository class to handle CRUD for all of them.
  - Custom C# attributes on model classes encode the table names for each entity, letting the repository discover the right tables at runtime without hard-coding them per type.
  - Dapper's SqlMapper type handlers bridge custom collection types (StringCollection, CategoryCollection) to SQL columns, keeping model properties expressive rather than collapsing them to primitives.
  - A lightweight SQL statement builder for WHERE clauses, JOINs, and pivot tables eliminates fragile string concatenation in complex search queries while keeping raw SQL control in the developer's hands.
draft: false
---

*And building a framework to use them*

![Photo by Ruchindra Gunasekara on Unsplash](https://cdn-images-1.medium.com/max/800/0*03EfwbpSQ-9JRdTk)
*Photo by [Ruchindra Gunasekara](https://unsplash.com/@ruchindra) on [Unsplash](https://unsplash.com)*

In this article we're going to walk through a thoughtful SQL Table design that gives us an efficiency boost in C#. We will accomplish this gain by ensuring that we utilize common data elements and building a framework in C# around these elements, using common SQL Statements to share code.

The article will be broken down into the following sections:

1. Problem Statement
1. Model Design
1. Table Structure and Design
1. Attribute Usage
1. Service Definition
1. Basic Add and Get Implementation
1. SQL Statement Builder Framework
1. Advanced Searching

Let's get started!

> **Note**: We will need two NuGet packages for this project: [Dapper](https://www.nuget.org/packages/Dapper/) and [Dapper.Contrib](https://www.nuget.org/packages/Dapper.Contrib/)

## Problem Statement

Our goal is to build a simple inventory system. However, we want a generic inventory system, one that allows for items to be categorized and custom attributes to be added. The categories themselves can have categories and attributes. This provides a means to add some hierarchy to the customer implementation and utilize categories as "tags" in some scenarios.

We will create only two models: Category and Product (however many more can be created). We want to utilize the same underlying repository class and re-use as much SQL and code as possible. To make the interaction with SQL a bit easier, we will utilize the [Dapper](https://github.com/DapperLib/Dapper) micro-ORM for its CRUD capabilities and syntactical sugar in executing SQL.

## Model Design

First thing's first. Let's design our data model in C#.

#### Base Instance

We want to have a common subset of information for every instance. A common model will allow us to create a general table structure and build an efficient system. Let's use the following class for all entities in our system as a base.

<script src="https://gist.github.com/a6fd1665d0bcfb9b912e31c34722409f.js"></script>

This model is fairly simple:

- **InstanceId**: Identifier of the entity
- **Name**: A unique name for the entity
- **Description**: An optional description of the entity
- **SystemOwned**: A flag indicating whether this is owned by our code or the user has defined this instance (useful for building modules on top of our structure)
- **Categories**: The associated categories for the instance
- **CreatedTimestamp**: When the entity was first created
- **CustomAttributes**: Any custom attributes associated with this instance.

You'll notice a few attributes on this model that will be become handy in the future. These are Dapper-exclusive and allow us to use built-in CRUD methods within Dapper. `[Key]` is used to define the primary key of the system, while `[Computed]` indicates that we should not include the property on updates (see [Dapper.Contrib](https://github.com/DapperLib/Dapper.Contrib)).

We'll go ahead and define `CategoryCollection` next,

<script src="https://gist.github.com/20d3e248573b2f320d9c9dfd8a4c52f4.js"></script>

I left out some implementation details from this code snippet for simplicity (like `IEquatable` and `Equals` / `GetHasCode()` implementations). The purpose of this class is to hold what categories are associated with the instance.

We should also define `CustomAttributes`,

<script src="https://gist.github.com/0219f86d5b28cf5d2b2ccde145ce31e9.js"></script>

Again, left out a number of helper functions and overrides that would typically be used in production.

Now that we have a baseline, let's create our specific data models.

#### Category Instance

A category can act as a "tag" for any entity in our system. A category itself can also have "tags" applied to itself. This allows a lot of flexibility with how we structure the higher-level inventory system. For instance, we may have different types of products: Stocked Inventory, Raw Materials, or Assembled Products. Each of these would be a separate instance of Category.

However, we could also use categories to tell the system about a more hierarchical set of products. For instance, what if we sold trading cards (like Pokémon). Pokémon is a game, but cards are broken down into game sets (XY, Double Crisis, etc). In this case, an individual card (a product) may have two categories: Pokémon (the game) and Double Crisis (the game set). Then the game set, Double Crisis, would have a category of Pokémon giving us a link to the card game.

This type of structure is extremely flexible for many types of inventory systems.

<script src="https://gist.github.com/3a5f1ecd01a765119f60cab7b80e6d30.js"></script>

#### Product Instance

A product is defined as an entity that can be inventoried and tracked. Like the category, it can have "tags" associated and custom attributes. However, we will add two additional properties that are a little more specific to our product types: `ProductImageUris` and `ValidSkus`. These are used in other parts of the system for specific customer cases.

<script src="https://gist.github.com/4072d76cf86f7b5f478cabc614011377.js"></script>

These additional properties are designed as collections of strings. Let's show this definition,

<script src="https://gist.github.com/37fa2a7a2a935949c3406d0f69434972.js"></script>

> `StringCollection`, `CustomAttributes`, and `CategoryCollection` could have very easily been `List<string>`, `Dictionary<string, string>`, and `List<CategoryCollectionEntry>` however these types do not express the intent of the types. Intent is helpful when reading code. Plus, we will want the types to be targets in our Dapper setup.

## Table Structure and Design

Now that we have our models, let's create our tables. Each entity will need a set of three tables: Instance Table, Category Table, and Attributes Table. The core structure of these tables is virtually the same.

#### Category Instance

<script src="https://gist.github.com/a474fba54d9e09856ecb83d0b7825ef2.js"></script>

*Instance Table*

<script src="https://gist.github.com/8ba1e6e4f727f35f2845128cf8f0620b.js"></script>

*Category Table*

<script src="https://gist.github.com/324749847cab34c7cbd8781c41250e7c.js"></script>

*Attribute Table*

The last two tables are a standard set of link tables where each category can have any number of other categories. And each category can have any number custom attributes. The primary keys are utilized to provide uniqueness on the link tables.

#### Product Instance

<script src="https://gist.github.com/03bed9f7ead6374a1039555131e7aafc.js"></script>

*Instance Table*

<script src="https://gist.github.com/d86d90a202bf3a99b35c8cd649ccbc64.js"></script>

*Category Table*

<script src="https://gist.github.com/b5bbd53ae8631bcdee760d486d299832.js"></script>

*Attribute Table*

There are a couple differences between these tables.

1. The name of the product is not required to be unique. Through experience, products can have the same name but have different manufacturers.
1. The product has additional properties in the table where the type is `VARCHAR(MAX)`. I would not normally condone this in a design, but for our use case, it works.

> It should be noted that we are utilizing SQL Server in this article to reject insertions based on the data provided. For instance, if you try to create a category with the same name, SQL will reject it. However, for products it would not.

#### User-Defined Table Types

In order to get some of this information into the system efficiently, we're going to need a couple UDTT in SQL Server. In particular, we're going to want to upload data for `CategoryCollection` and `CustomAttributes`.

> See my article [Advanced C# — SQL Server Bulk Upload](https://blog.devgenius.io/advanced-c-sql-server-bulk-upload-57ad6be6e6a1) for a more detailed explanation.

<script src="https://gist.github.com/de6b9fb2618396d79cc9c7ac7bec0428.js"></script>

*Custom Attribute List UDTT for uploading CustomAttributes*

<script src="https://gist.github.com/04320b98f8b7f6daddfbcbd5c1c57ced.js"></script>

*Integer List UDTT for uploading CategoryCollection*

With these two types, we'll be able to upload a single entity with one `INSERT` statement.

## Attribute Usage

Each of our models has different tables associated. Yes, the tables are very similar. However, we're going to need to tell our repository service about these tables somehow. I prefer to keep these definitions as close to the model as possible.

To do this, we're going to use attributes. We're going to use an existing attribute from Dapper.Contrib called `TableAttribute` and create a custom attribute called `PropertyTableAttribute`.

The `TableAttribute` tells Dapper.Contrib what table to perform the CRUD operations (see [Special Attributes](https://github.com/DapperLib/Dapper.Contrib#special-attributes)). By default, the table name will be the name of the entity pluralized. However, we are using schemas and we'll need to customize to match by using this attribute.

The second attribute, `PropertyTableAttribute`, is used to define the other tables for Categories and Custom Attributes.

<script src="https://gist.github.com/40e5737e4915cb70b64bf6b279b8d018.js"></script>

We will then decorate each of our models with these attributes based on the tables we created.

<script src="https://gist.github.com/cf2237f7da59580656683450e41b9e4c.js"></script>

## Service Definition

Now that we have defined our models and tables, it's time to define our service.

#### Service Interface

The repository interface will be called `IInstanceRepository`. We will leave out all other CRUD operations, except for `AddAsync`, `GetAsync`, and `FindAsync`.

<script src="https://gist.github.com/e2f538e3e2e41b1754a118a007470305.js"></script>

*Interface Definition*

#### SQL Server Implementation

We will be creating an implementation with SQL Server, utilizing Dapper and a custom `ISqlExecutor` (see article [C# Tip: SQL Executor Service](https://blog.devgenius.io/c-problem-sql-executor-service-deb459132a50) for more details).

<script src="https://gist.github.com/b50819b7262ef27abc2189ee5ae0e09a.js"></script>

*SQL Executor Interface*

<script src="https://gist.github.com/6fe1e8a07f18179aa6c92102ce84ffbb.js"></script>

*SQL Executor Implementation*

There is a lot happening in this class. We could have a whole article on this by itself. However, for now it may be best to study the class and hit me up with questions.

The key points to note are the following:

- In the constructor, we are pulling out our attributes and storing the results in local properties. This class should be a singleton and thus it is a one-time cost.
- When writing categories and custom attributes, we are utilizing our UDTT as previously defined, and producing a custom parameter (for Dapper) with the `.ToSqlParameter()` extensions.
- I am using a simplified approach to overwriting categories and custom attributes where I delete all associated rows first, then add them back in. This isn't the most performant and can be optimized further.
- Within the `GenerateFindQuery` method we are using a generalized filter query to first find the instance identifiers. Then we perform several `SELECT` queries based on those results to return multiple result sets. I have found this to be highly efficient from an indexing standpoint.
- SQL is being concatenated. When developing code like this, we must be extremely cautious to ensure that no user data ends up in the SQL without being parameterized.

There is one last class that is used here (not mentioned yet). And this is the `InstanceReader` class. This class is used to take the results from a `Get` or `Find` query and create concrete instances. The class is defined below.

<script src="https://gist.github.com/1a636ef90b92ad1eca29cd763403769e.js"></script>

I won't go into any detail on this class in this article but wanted to include it for completeness-sake.

## Basic Add and Get Implementation

Now that we have created our core framework for our shared instances, we should implement the functions in the repository. We will only focus on the `AddAsync` and `GetAsync` functions in this section. We'll need to add a little bit more to our framework for the advanced searching.

<script src="https://gist.github.com/bb80d77963155a81ea4d2e4a87af37d4.js"></script>

#### `Task<int> AddAsync(TInstance instance)`

This function is used to add the instance into the system.

We utilize the Dapper.Contrib method `InsertAsync` as described in their documentation. It automatically parameterizes the properties and inserts the record into the table. To make this work as shown here, we have to match our record property names with the database column names. Since we did this, the data is seamlessly added.

Next, we write the attributes and categories using the executor we defined in the previous section.

If you pay attention close enough, I have left something out of the process thus far. This function works great for all the simple parameters we have created. But what about the special `StringCollection` properties defined in the `ProductInstance` class?

For the `InsertAsync` function to convert this seamlessly, we have to "teach" it what to do in these cases. At some point during the initialization of the system, we have to add a type handler for `StringCollection` to Dapper's `SqlMapper` class.

<script src="https://gist.github.com/826a5364a9f0fc5285e57a13416690d1.js"></script>

Without going into too much detail here, just know that our collection of strings will be turned into a JSON string that can then be stored in the `VARCHAR(MAX)` field. We could also change it to be a simple comma-delimited list as well, depending on our needs.

#### Task&lt;IEnumerable&lt;TInstance&gt;&gt; GetAsync(params int[] instanceIds)

The get function is used to bulk retrieve instances from the system. I defined it this way because it is easier to go from multiple instances to one, with an extension, than the other way around. With the efficient methods we're using to query, we don't have to sacrifice performance for a small number of instances over larger lists.

In this method, we are simply providing a filter that selects ALL instance ids provided. The use of the filter will become more apparent when we do advanced searches.

## SQL Statement Builder Framework

I'm a SQL guy. I'm not a huge fan of too many black box implementations. This is why I choose not to utilize Entity Framework or other ORM(s) that produce SQL Statements for you. Instead, I like the control and ability to optimize the way I like.

However, when we start dealing with complex queries we do run into some significant hurdles when it comes to building them up. In response to this issue, I built a simple set of classes that eases my pain when filtering and generating reports.

Let's take a look at these classes.

<script src="https://gist.github.com/906717ec04322f0328c859ff01f6fe68.js"></script>

*Where Clause Builder*

This class allows me to quickly build a where clause assuming `AND` operations.

<script src="https://gist.github.com/badbca6d73c2e67b5b8d01444aba03df.js"></script>

This class allows me to quickly add `JOIN` tables to my query.

<script src="https://gist.github.com/79dc082ef777c75f980487fb365cd783.js"></script>

Lastly, is the pivot table. An issue arose with my queries in which the custom attributes search provided false results because of the naive searches I was doing. I utilized some basic inner joins and filtered on these joins for each attribute. However, if I wanted to search for a product that had 2 attributes that matched, it failed. Instead, I would get any instances that had 1, or both, of the attribute values (acting as an OR) operation.

In order to make it work, I needed to create a pivot table allowing for the `AND` operation to occur with multiple attributes. This class eases that pain a bit.

## Advanced Searching

Now we're at the `FindAsync(ExactMatchQuery)` method. The goal of this method is to search based on various optional properties supplied to the function. Let's show the code first, then walk through it.

<script src="https://gist.github.com/f17c8cc9f8b99cb81f66089c20c565b2.js"></script>

There is a lot happening here and a lot of complex SQL logic. However, here is the just of the algorithm:

1. We start by creating sub-query joins for properties like `Categories`, `ExceptionCategories` and `CategoryIds`. We use sub-queries because of the nature of the different link tables defined at the instance level.
1. We then start the `WHERE` clause. We add to our custom `WhereClause` class based on which properties have been set on the query. The `if` checks determine when we add an entry to the class.
1. Due to the complexity of `CustomAttributes` we check for this need and form the pivot table accordingly. If we don't need the attributes for the search, we optimize for this scenario.
1. We then create our filter query. Remember, all we're attempting to pull back are the instance ids. We have abstracted out the rest of the query (that pulls the real data) into the `ISqlExecutor<TInstance>` implementation.
1. Lastly, we add the parameters for the SQL and run the query.

The result of this query is a collection of found instances. This flexible query approach allows me to easily add new filtering capabilities by adding more optional properties to query, while easily adding the implementation as a new `if` statement.

## Conclusion and Sign-Off

Whew. That was a lot.

This code is fairly complex. You may be asking whether this level of complexity is worth the effort. I should say that the codebase did not start out this way. As I continued to refactor, and refactor, and refactor, it eventually ended up here. Originally, all the implementations were completely separated out with an additional 1000+ lines of code. You see there are a lot of entities in the system that use similar attributes. But as I continued to refactor, I noticed a lot of similarities and was able to pull this line count down by almost half.

But again, is the complexity worth it? I guess it depends on your perspective. I don't have to touch this code anymore. In fact, adding a new entity type is a matter of convention at this point. As long as I inherit from `Instance` and add the appropriate attributes to the class, I don't have to touch this code. In this respect, I've internally built a reusable framework. This is where the advantage comes in. However, if you find yourself editing this code often, then the complexity may not be worth the effort. It does take significant cognitive load to process this code after months of distance. But I'll let you be judge of whether this code is worth it or not.

I hope you found some useful techniques within this article. If you have any questions, feel free to comment below. Also, if you liked this article, follow me here on medium. It encourages me to continue writing!

Until next time!
