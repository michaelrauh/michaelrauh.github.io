---
layout: post
title:  "A Lifecycle for Append Only Data Management"
date:   2022-10-22 00:00:00 -0400
categories: jekyll update
---

## Introduction
In the previous two posts, the issue of fact databases and state merging are explored. State merging shows that the design of the dynamic programming post can be optimized significantly. Additionally, no merging logic need be implemented. One thing not built out here is that merging has a lifecycle that can reuse the below events as well.


## Algorithm
1. Data can be managed locally and globally. Merge events are global. Any time a local queue is empty (more on this later) a merge event is emitted and a DB is written. Later, a worker picks this merge up as a chunk of work and writes a new DB out. This eliminates one DB as it deletes two DBs when it does this.
2. Data can be marked public or private, and registries can specify batch size, number of batches, and timeout for how much to read off of the global queue before writing a merge event.
3. Local queues manage DBs as well
4. There are several methods to implement for data:
    5. abort_categorically - abort this event as there are no friends to act upon. This is different from get_friends which happens after friends are loaded. This is intended only to count friends.
    6. get_data - get the data to act on the event. This could be from an ECS, a hashmap, a database, a queue, etc.
    7. abort_data - abort the event because the data isn't interesting
    8. get_friends - get the friends of the data. Any data needed by the primary data to make an insight in the context of the event being handled
    9. abort_friends - abort because friends are not interesting
    10. search - use data and its friends to find new data
    11. save - save found items locally
    12. write - save all found results to a database
    13. public - whether this event will be written to the public or private queue
    14. priority - where on the queue this event should go. Ideally the order is such that recursion depth is lowest it can be, or aborts are maximized (these are likely the same. In pyolyvinyl-acetate order is currently by what grows the DB the slowest. This is misguided and based upon assumptions of non-merging and non-skipping.)
13. The handler that holds the different data implementors will also have a few properties:
    14. batch_size - how many public events are read from the queue at a time
    15. batch count - how many batches of events should be read from the queue before sending a merge event
    16. timeout - how long to wait for events on the public queue before accepting a partial batch or skipping a batch
17. The library should also enable easy benchmarking
    18. example_hit - data and friends that produce a "found" event
    19. example_miss - data and friends that don't result in anything
    20. example_near_miss - data and friends that almost result in something
21. There should also be handlers for the web interface
    22. count_api - how to get the count out of the DB
    23. insert_api - how to insert one of these types into the DB to create an initial event


## Conclusion
With all of this together it should be trivial to build a scaled system. The only remaining tricky bit is doing the operations necessary to take the three essential functions (managing the queue, responding to work, and monitoring the work) and scaling them into production. polyvinyl-acetate manages this through kubernetes, but it seems like a potential overreach to try to build this sort of thing into the proposed library.