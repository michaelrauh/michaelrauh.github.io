---
layout: post
title:  "The Point of Execution in a Program"
date:   2018-05-14 20:59:30 -0400
categories: jekyll update
---

# Point of Execution

## The things that make a program hard to debug and extend
1. Not knowing the state of the program
1. Not having a place to put new logic without affecting old logic

## What is the state of a program?
1. The data bound to each variable
1. The concrete type of each interface - this just allows code reuse and doesn't factor into this talk much. You could duplicate for each type. It does play into polymorphic dispatch a bit but even then it is not a huge deal
1. The call chain leading to the line of code that is being executed at a given instant
  * This is referred to as the point of execution at a point in time

## Why is it hard to add new logic without breaking old logic?
1. Logic interferes with itself.
1. When we don't practice open/closed principle, we reuse logic by putting it near other logic. The logic it is near gets altered and breaks

## How do we fix these things?
1. single use - this makes it so that your code doesn't try to do more than one thing. This is SUPER important, as you can't break things you don't touch
1. open closed - this is important as you shouldn't touch code and risk breaking it
The other parts of SOLID are extremely important as well, but they factor less into the idea that I am focusing on at the moment.

## Single use
1. Practice this at write time. When you make a new thing, make it single use

## Open closed
1. Practice this at edit time. By that I mean don't edit classes. This makes it clear that these two principles really form one idea that is hard to put into words. Something along the lines of "don't make me edit this now or later"

## How can I tell if I am breaking these ideas?
1. This is a deep question but I am going to focus on one part of it for this talk. Don't use if. Things that count as if are holding a map of functions, unless, case statements, and loops

## How do I get good?
Every time you are about to write an if statement in a function, make two functions. Then call those two functions from two different places. The nearer you are to the boundary of your code you are at this time, the easier this will be. If you are in a callback make it two callbacks. Done. If you are in logic below a callback, split it, split its caller, then split the callback. What if you can't split the callback? Use the router delegate pattern.

## The router delegate pattern
1. Pass the input into a class called somethingRouter
1. somethingRouter does magic and eventually calls a function that belongs to the original class
1. The original class was a controller. Now you have callbacks from the network, buttons, and your router all in one place
1. If you ever forsake this pattern, that will not harm you. Each split apart bit of code will have branching, but it will be more maintainable than if it had not been split. Worst case they all call the same helper if you want to deduplicate your code. If that helper must deal with different types, use an interface. Polymorphic dispatch will take care of you then, but ultimately it is just a trick to reuse code.
1. This gets at the essential idea of the talk - execution is all that matters. If you have serial data, convert it to execution as much as possible and then let that execution ride without branching. You'll end up with a beautiful web of code where it has one point that touches and the rest is a line out of the center.A fuzzball basically

## What if I branch but I am not doing it for side effect?
Sometimes you are just formatting output. Just use a partial function for this instead. It is a better expression of intent

## What if I am deep in legacy code?
1. If you can't go all the way with it, do this to just a section of the code. Start in a controller and call it a day. Any branching that takes place outside of a router should be split up until it ends up in a controller callback or a router that calls a controller callback
1. If the branching occurs too deeply and you don't want to commit, but you want to get some of the benefit, insert the strategy pattern. Branching occurs for one of three reasons.
  1. You need to transform data in a way that calls for a partial function
  1. You need to take serial data and perform different actions depending on domain (routing problem)
  1. You used to know your point of execution but now your code paths have collapsed (you have two different calls to the same function)
If it is the last reason at play here, then set the appropriate state on a state holder in each caller to the function. This doesn't have to be a direct call to the function, it can be an eventual call.

# The benefit
Once all of the code is split apart, adding functionality is a matter of adding a new class to your code and adding a new callback. You don't have to mess with the old code, you don't break things, and you still get to reuse code because you broke out repeated sections of code into bits that just take an interface type
