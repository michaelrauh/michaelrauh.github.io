---
layout: post
title:  "Thin Slicing a Language"
date:   2020-12-06 00:00:00 -0400
categories: jekyll update
---

## A Working Thin Slice

First, let us start with the ideal (don't worry if none of this makes any sense yet. I want you to see a small diff):

<script src="https://gist.github.com/michaelrauh/5ee9f895412ad0f6a19860fa87750afa.js"></script>

In this diff, a few lines of code are added. One in the syntax layer and one in the semantics layer, and you have a full new feature of the programming language that you've been developing. This is not the norm. 

To break this diff down a little, let's go file by file: 

<script src="https://gist.github.com/michaelrauh/203e7b149cc5787980cb11bccb0eb7d7.js"></script>

This is a line of code in the target language (affectionately known as #lang aoclop [pronounced eyoo-clop]). Reading it out loud sounds like "read file 1, delimiting by newline. Downscope. Identity. Upscope. 

What this means is that the program is a passthrough that will read in a file called 1.txt and output the contents of that file. Don't worry too much about what downscope means yet.

To the next file: 

<script src="https://gist.github.com/michaelrauh/64f55808edd724c455be6da80eab6fde.js"></script>

This is adding identity to the lexer, meaning that we now have a lexical entry for the identity function. It is missing a semantic binding. Entrepid readers will notice that this is not a full slice as we have not had to touch the parser. This is a result of making identity merely an op token. ops are things that fall between pipes, so it can be parsed and forwarded to the lexer without changing the grammar. 

Now the part we have been waiting for: adding semantics to identity
<script src="https://gist.github.com/michaelrauh/8b7f8b84392826b283bd28f76a8ec20e.js"></script>

This is a part of a macro that converts the word "identity" to the Racket function `identity`. Conveniently that already exists so no further work is needed. It appears twice to account for the fact that identity can appear as the tail of the expression or not. If it does not, it must apply to everything after it. There is a bit of refactoring that can be done here to avoid this repitition, but this early on it had not made itself clear, and it was never harmful so it stayed.

## The Full Code

Therefore the short version is that it would be nice to be able to have a DSL, and add to it using only a few lines of code. This is entirely feasible as long as you don't run into one of a long list of pitfalls. Before getting into those, here is a working state for the code:

<script src="https://gist.github.com/michaelrauh/85c04117f06830d67562edee88e3105a.js"></script>

This has slightly more in it than the previous thin slice as it lays out what was there before. In particular, it has a binding for the `nl` delimiter. This is a very small syntax rule. It additionally provides a binding for `read`. Conveniently, `read` can be a plain Racket function (not a macro) as all of the code around it is valid Racket by the time it is run. In general, macros are needed much less often than one would assume, as the parser emits valid s-expressions. Binding each of these to a macro or function allows for them to take on the desired semantics. The macro vs. function discussion is long, but the short version is that functions are generally preferable and should be used unless functions can't cut it. 

Next is the rest of the lexer. There should not be much in the way of surprise here.

Main wires things up.

The parser is written in the form of a small `brag` program which takes an EBNF grammar and produces a parser. This is exposed in the tokenizer. All of this should make sense. `Brag` is a big part of the inspiration for this post. It is such a nice DSL that it becomes an advertizement for using them everywhere.

Now for the pitfalls:

## Fake it 'Till You Make it 
This does not work. It is tempting to write the simplest code that will produce the input / output pairs that you want, but if it is done in a tricky fashion, it will be untenable in the long term. Here is an example: 

<script src="https://gist.github.com/michaelrauh/952676d791f24dcf6346c45e91de29e0.js"></script>

`div-each` has a call to map inside of the function. According to the functor laws, map is closed under composition (map f a . map g a = map f . g a), so you're fine, right? Obviously, wrong, or this example would not be in the pitfalls section. Doing this sets us up to have to implement everything in terms of functions with `map` inside of them. If a function can appear in multiple places (not necessarily in a place that is "downscoped") then you've made a place where you have to point to a different function instead of the mapping one. Now there is a context sensitive part to the whole operation where your macros need to detect if you're in a scope block and route to the right map. All because it seemed alright to cut this corner and put the call to map inside of the function instead of in a higher scope.

There is a near remedy here:

<script src="https://gist.github.com/michaelrauh/7ae3d3cf279dbaf4baf72e6af733fdb2.js"></script>

But that is not moving `map` to the right place. It must be moved to a place that moves with scoping operators. When there is a semantic disconnect between your source and target, it results in all sorts of patches.

The real issue was that the first scope block was "implied" in earlier versions of the source language. This was a mistake and was fixed here:
<script src="https://gist.github.com/michaelrauh/cc375c9501a682a0563040c2d4d8a692.js"></script>

## Carrying State
This makes it so that your code has to run in some special order, and it is not inspectable at compile or macro-expansion time. Fortunately it is easy to remove like here:

<script src="https://gist.github.com/michaelrauh/6cd6421d22e5e6daf78d59359672cca6.js"></script>

Eliminating these concerns makes it easy to see satisfying refactors where, if your macros evaluate to the same string, you're safe.

<script src="https://gist.github.com/michaelrauh/1c4ecfe7669f02bd146ab02c085d6dfc.js"></script>

## Takeaway
Refactor very early. Refactor here is driven by the difference between what is flexible and what should be flexible, and misses in the semantic mappings between source and target.

A DSL is defined by that which may be assumed. In Fowler’s book "Domain-Specific Languages" he says that the value is in limited domain and limitation of expressiveness to improve clarity and reduce bugs, as well as to make changes easier. If your DSL grows into a GPL, it loses value. I agree with this, but have a different viewpoint. In my opinion, DSLs increase your number of assumptions. Assumptions are things you don't have to think about. They also represent system inflexibility. The more system inflexibility you can introduce, the more flexibility you can achieve in the flexible bits, and the easier it is to add functionality. The issue is that you can get this wrong, and then you’re in a world of hurt. The fix to this is that you have to be very willing to throw away old code. Anything that does not map perfectly to what is desired from the standpoint of what was intended semantically to what is really there must be fixed early on or it will only get worse.

The next question to look at related to these ideas is in practical language oriented programming. I think that these advanced tools make it practical to thin slice delivery of a language without a big up front design. The question is in whether doing so allows for the foresight to know when to split languages, and whether there is strong enough support for disparate language composition. Racket makes it easy to mix domain specific languages so the idea of writing a dozen tiny languages to fix one problem seems more tenable than one would imagine. The ideal for domain specific languages would be something that feels like playing the new Super Mario Odyssey, but with much less time spent as Mario. In particular, there should be many, many languages that are so small that learning them and using them is effortless.