---
layout: post
title:  "Merging of Event Driven Fact Databases"
date:   2022-10-21 00:00:00 -0400
categories: jekyll update
---

## Introduction
In the last post, I talked about how fact databases (maybe a better term would be append-only dbs) and dynamic programming problems have to find a way to work in a `serializable` fashion. It turns out that this is incorrect! There is a general way to merge state in event driven fact based databases. The interesting thing here is that given this rule, this state can be saved in a variety of layouts including straight hashmaps, entity component systems, denormalized databases, normalized databases without explicit foreign keys, and normalized databases with explicit foreign keys (though I am slowly drifting toward thinking an ECS is the best way to go in this domain).

## Algorithm
The essential insight is that all data has a category, and it also has a "friend" category which is other data (this friend category can be a union type). If you think in terms of search, it becomes clear why this matters. Most algorithms have the data in question, and then some other data that is inspected and helps to solve the problem. In a graph traversal, points are friends with edges, and edges are friends with points. In relational databases, a friend is anything with a foreign key.

When building up a single database, it is done as described in the previous post (and the next one actually). At some point in time, it becomes expediant to merge these DBs. To do that, dump all of the data in one database into the other, and mark each piece of data by provenance. Specifically, it should recieve one of three tags: source, destination, or both. After that, replay the events of the source database on the destination. As the data comes in, there are a few questions that can be asked about it:

1. Is this data found in both databases (be careful here, there are usually rules for tolerance of sameness. Just because something is not identical does not mean it is not redundant). If so, ignore this event.
2. Look at the friends of the data. Is every friend in both databases? If so, ignore this event.
3. Look at the data and friends one last time. Are they all in one database and not the other? If so, ignore this event
4. Finally, if we get here, that means that the data is mixed and must be replayed. Replay the event on the combined database to capture any missing connections.

The interesting thing about all of this is that it should be rather fast. It seems relatively unlikely that the three skip checks all fail. Lots of data is either very redundant (the first and second cases) or a clique (the third case). 

There are also a couple of skips on top of all of this that can provide even more speed. In the next post I will bring up the full lifecycle hinted at in the previous. In particular, `abort_categorically` would preempt point 2.

## Conclusion
This method of state merging should generalize to any database with irrevocable facts, and so should be useful for a large class of problems. In particular, it should be useful in places where ETL pipelines suffer - data with high interconnectedness and high recursion depth.