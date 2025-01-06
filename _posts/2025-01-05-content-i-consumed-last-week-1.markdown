---
title: Week 1 - Content I Consumed Last Week
date: 2025-01-05 22:23:00 -0400
categories: [Content Consumption and Reflection]
tags: []
---

As we start off the new year, one of my goals is to consume less content and reflect more on what I consume.
I wanted to kick off a new initiative breaking down how I understood 1-3 pieces of content per week as a brief post here.
This is mainly meant as a means for keeping myself accountable and learning new stuff along the way.
I'll also be playing around with the formatting and structure to iterate until I find something that sticks. For now, I'll be sticking to a format that resembles my rough notes for the pieces of content.

Week 1 overview:
- Code Indexing at Scale With Glean

# Code Indexing at Scale With Glean

Engineers at Meta put out a [blog post](https://engineering.fb.com/2024/12/19/developer-tools/glean-open-source-code-indexing/) outlining Glean, an open source code indexing tool they build and use internally.

## TLDR
- Meta found that indexing code in their large code base for some languages (like cpp) was very slow due to long compile times
- They wanted to build a solution that would index their code asynchronously and share that across their entire development team
- They also wanted to build additional features on top of this solution to improve DevXP. This means that the solution itself has to be generalized enough to allow for future use-cases

## The Problem
Meta has a very large code base. They also use a Monorepo to keep all their code as a single source of truth. Engineers encountered a few problems related to indexing code when developing including:
- Indexing code for languages such as cpp would take a long time due to long compile times
- Each developer would have to index code in their local machines when instead it would make sense to share this across the entire team in a centralized location (since the codebase is the same)
- Meta wanted to build additional features on top of this indexing to improve other aspects of DevXP

## The Requirements
Meta wanted to focus on building a solution that could be extended for future use-cases instead of focusing too deeply on code-navigation. As a result, the requirements for the system they wanted to design were as follows:
- Generalized storage solution that is not specific to any one programming language
- General but unified way to query for data stored into the system
- Ability to add some language specific data to add some more powerful features. However this should be abstracted away as a layer in the schema to achieve a language neutral view over all the stored data


## The Solution
Glean uses a unified language for defining schemas as well as querying called Angle (anagram of glean). Users can define something called a Predicate (roughly looks like a Table in SQL). Each instance of the FunctionPredicate is called a Fact (roughly looks like a Row in SQL). This means that users would be able to query a Predicate which would return the associated Facts. Querying using Angle looks roughly similar to SQL (specifying a Where clause to filter results).

One key aspect to Glean is the concept of incremental indexing. Since Meta has a very large code base, they would not want to re-index the entire code base on every single change. Rather, they want to simply index the changes (and related data to the changes) instead. They mention this in terms of time complexity as moving the cost of indexing from O(repository) to O(changes) (which actually will end up being O(fanout) which includes re-indexing data depending on the changes as well). The solution that Glean uses here is stacking immutable databases on top of one another where each layer of the database can add information or hide information from previous layers but would look like a single database from the perspective of the client.

In general, an indexer would index source files asynchronously which would then be stored into Glean. Developer tools would then be able to use Angle to query for Predicates in Glean which would return a set of Facts. These can then be used for various use cases across the company.


## Use Cases
Meta currently uses Glean internally to support a wide range of use cases, some of which include:
- Code navigation at scale which is immediately available to all developers on IDE startup
- Cross language navigation to find service definitions or implementations for something like RPC calls
- Documentation generation since enough data is collected within Glean to reconstruct full details of the API
- Analyzing code changes for static analysis on complex lint rules as well as providing code navigation capabilities directly in code diffs

