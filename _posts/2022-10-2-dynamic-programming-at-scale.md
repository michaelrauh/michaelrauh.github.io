---
layout: post
title:  "Dynamic Programming at Scale"
date:   2022-10-2 00:00:00 -0400
categories: jekyll update
---

## Introduction
`Dynamic programming` is a really vague term for a really common thing in programming. Lately, I have been trying to understand what ties together `advent of code`, `solver aided programming`, my `fold` program (the current version called `polyvinyl-acetate`) and `datalog`s. The unexpected answer (for me) is that they all tie in to dynamic programming in some way. I've noticed that there are more instances of recursive problem solving than people immediately recognize, and also that this expands even further when you allow that recursion to be deep, mutual, and tangled. Notably, I am not trying to solve anything that has temporal coupling, global ordering, random choices, or a concept of global shared resources.

## Current Tools
Currently, as mentioned, there are a few ways people tackle these problems. First, is through use of recursion. This is usually enough. When it is not enough, using something like `rosette` will certainly get you there. As a disclaimer, `rosette` is absolutely awesome, and if you haven't tried it yet, you should. The one place it falls short for me is in incrementally calculating answers for mostly disjoint relations, or scaling up to work on clusters. Last, datalogs, such as `differential-datalog` (and arguably `materialize` [both being descendents of `differential-dataflow`]) are very strong, but still miss some of the characteristics that I am looking for. Namely, they should scale up quickly, be opinionated in structure, be expressive, allow for arbitrary computation and data munging, allow for transients, and be `ACID` compliant. There is also significant startup pain around each for different reasons. That is not to say that they are not also awesome, of course. Oddly, googling dynamic programming libraries does not pull anything in this realm up. It mostly brings up stochastic dynamic programming libraries, which are cool too, but isn't what draws my interest.

## My Current Code
I have a program known as `polyvinyl-acetate` that is deeply flawed. It is intended to take in text and fold it into N-dimensional structures. Ideally, when it is done, it will have a nice ingestion dashboard, a nice results explorer, benchmarks for each logical bit of code (more on this later), observability built in, the ability to scale up worker nodes to process faster, fault-tolerance, the ability to inspect the event graph with cardinalities, and the ability to track provenance of an event. Most of this is started, but also most of it is not done. The interesting thing is that none of this has to do with folding text into N-dimensional structures. If I were to separate this out into library code and text-folding code, it would be easier to reuse the library bits for other things.

## Guardrails
This is not a "framework" that you put code in to. It should be a collection of tools. It should be fairly DRY and CoC.

## Initial design ideas
The big idea here is that this type of problem is well suited to a few basic patterns:
1. a single database set to serializable isolation
2. a queue of work that can be reordered for performance
3. an HTTP interface
4. a lifecycle. An initial design for this lifecycle could look like:

```
trait DataCycle {
    type Data;
    type Friend;
    type Found;

    fn load(id: i32, conn: &PgConnection) -> Result<Self::Data, anyhow::Error>;

    fn abort_early(data: Self::Data) -> bool {
        false
    }

    fn load_friends(conn: &PgConnection, data: Self::Data) -> Result<(Self::Data, Vec<Self::Friend>), anyhow::Error> {
        Ok((data, vec![]))
    }

    fn abort_late(data: Self::Data, friends: Vec<Self::Friend>) -> bool {
        false
    }

    fn process_data(data: Self::Data, friends: Self::Friend) -> Vec<Self::Found>;

    fn throttle() -> Option<i32> {
        None
    }
}
```
5. benchmarks
6. monitoring
7. the idea of immediate data, the friends of that data, and results. If results repeat they are ignored
8. the idea of finding data and fitting it into the bigger picture one at a time. The order of discovery should not matter.
9. the idea of data being permanent. It should not be a part of the workflow to revoke or edit anything inserted into the database. Transient data can be handled in RAM
10. The idea of workers that are all the same, so that they can take in any event and work on it.
11. fine-grain events to manage database access so that there is not too much locking.
12. A concept of uniqueness for each bit of data in the database.

## Next Steps
The next step is to start separating text folding code from more general code. Additionally, it would be interesting to use the library for other use cases. Ideally, with enough structure, it should be possible for a lot of help to be generated. For example, it should be possible to generate benchmarks for the process_data methods, and to tell the rate at which abort_late is called. It should also be simple to add APIs to insert, fetch, and count data. If you'd like to pair in, talk about it, or fork the code base, let me know!