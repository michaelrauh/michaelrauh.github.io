---
layout: post
title:  "Function Application is Specialization"
date:   2022-10-19 00:00:00 -0400
categories: jekyll update
---


## Introduction
I've noticed a quiet idea floating around that Functional Application is Specialization. While this is literally true, the statement is more useful and less vacuous than it seems. An example can help to make this clear:

## Example
1. take a function f() = 5. this is specialized
2. generalize it by passing in 5
3. f(x) = x
4. Now it can be any number
5. Pass in 5. f(5) = 5. Now it is as specialized as before
6. f(x, y) = x * y is fairly generic. Partial application gives us
7. f(5) = 5 * y. This is more specific. 
8. The essence of refactoring is in finding what parts of the system need to be more flexible and what parts should be inflexible, dependent on other types, or move together. Thinking in terms of function application can be a start.
9. The other idea is one of cohesion and coupling. These are in tension with each other. Overgeneralizing something out such that it doesn’t own anything doesn’t buy you anything. You can think of it as a responsibility to hold certain data or mediate operations, or to know the order of things as well.

## Conclusion
Looking at a project architecture from the idea that certain parts should be general and other parts should be specialized, and that specialization is the process of taking something general and providing information to it helps to inform that architecture. In the big picture, a predictable architecture has stages of specialization in it. Each stage should have the responsibility of acknowledging and holding the data that it is given, while specializing those collaborators down the stack.