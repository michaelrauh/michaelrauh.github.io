---
layout: post
title:  "Code, Chores, and Games"
date:   2022-10-23 00:00:00 -0400
categories: jekyll update
---


## Introduction
I've noticed that code that I like to work with has a lot in common with well-executed chores, and code that I don't like to work with has more in common with video games.


First, what makes code hard to work with? I've noticed some common patterns of issues:

1. aborting a loop early will give a partial transform inaccuracy
2. An if statement in a loop may also give a partial transform
3. initializing something and then filling it is a common pattern leading up to partial transform
4. Sometimes a transform touches other data it should not have, this would be an overactive transform inaccuracy
5. other transforms fail because of domain/range mismatch in chaining. 
6. still other transforms fail due to race conditions, probabilistic transforms, or unexpected transform order
7. some transforms are performed on data that should have not have been transformed. In particular, sometimes you want a fork when you get a linear chain. If you alter the data you may not have the input to put through the second filter
8. transforms access outside state, which changes under the transform or during the transform. This would give partially consistent or inconsistent results

This can be compared to doing laundry.
In laundry, clothing has properties
1. clean
2. folded
3. right side out
4. fits
5. color 
6. ownership
7. dryness

loads have properties too
1. size
2. exact contents
3. ownership
4. dryness

These properties can be thought of as invariants. When something bad happens during laundry, this can be a result of a broken invariant. For example, pocket contents are closed under walking and sitting. Violation of this invariant is a lost item inaccuracy, or a thing unexpectedly on floor or in chair inaccuracy. In laundry, clothing tends to stay inside-out or rightside-in in the laundry. It also maintains ownership, and, for the most part, color. It does not stay folded, and it tends to go from dirty to clean. It also tends to go from dry to damp. Interestingly, load dryness is an aggregate property.

On the other side, video games seem to have more in common with challenging code. In video games, there are several challenges such as:

1. previously used ingredient required
2. actions must be performed in a  specific sequence to avoid no-win situation
3. changing one thing changes two things and you only want to change one thing
4. necessary repeated action exhausts necessary resources
5. some point in time requires lots of resources 
6. end state is a result of overlapping actions that each change multiple things but also build on previous state

## Conclusion
If you want your code to be easy to extend, it may help to make it seem more like laundry than like Final Fantasy.