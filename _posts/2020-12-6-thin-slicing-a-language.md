---
layout: post
title:  "Delivering Software Projects as part of a regular agile cadence using custom domain specific languages while avoiding long design phases"
date:   2020-12-06 00:00:00 -0400
categories: jekyll update
---


## Introduction

I am writing this blog post in order to introduce an idea that has been on my mind about language oriented programming. I like language oriented programming a lot, but I am allergic to doing the dreaded big up front design. Pretty much all language oriented programming and Domain Specific Language materials assume or recommend a design phase (though some recommend they be shorter than others). (If I am wrong, please email me. I would like to read it) I am not saying that is a bad thing, but being someone who really likes language oriented programming, and also being someone who hates designing software when I could be writing it, I set out to do language oriented programming without any foresight (beyond that required to design the code I would write that day) by attempting to complete Advent of Code 2019 one prompt at a time by writing a programming language to solve each problem and potentially adding to it, or starting a new one when the next problem came along. A year later, I have made a lot of mistakes and managed to get through part of day two part one. Obviously this could be viewed as an abject failure. Therefore I am writing this post to go back in time and deliver it to myself. Its main message is that I was correct in assuming that you don't need a design phase. Is that a contradiction? No. I set out to prove that you did not need a design phase, but was woefully unequipped in other areas. This post will talk about two things:
1. What a thin slice of a language feature can look like in the best of times
2. Warning signs for when you have gone off the rails and are trending toward finding yourself finishing advent of code in November.
But first, a shout out to the best LOP book: https://beautifulracket.com. I would not have gotten anywhere without it. It is an inspiration, and everyone should buy it.

Most domain specific language and language oriented programming materials assume a design phase. That's bad. I don't like design phases. I am going to argue that you can skip the design phase as long as you:
1. Are careful not to write yourself into a corner, and
2. Don't mind deleting your code a lot.

Doing this will make you able to deliver software instead of designing things that do not work and then trying to deliver and then having to redesign.

## Adding the Identity Built in Function to AOCLOP

First, let us start with a line of code from a made up tiny domain-specific language (DSL). I named it aoclop (pronounced eyoo-clop). This is a program that takes in a file which has a column of numbers in it, and converts it into a list of those same numbers.

`read: 1 nl | v | identity | ^`

This is a line of code in the target language. Reading it out loud sounds like 

1. `read: 1 nl`
"read file 1, and assume each line in the file is a different value. 

2. `|`
And then ... 

3. `v`
Downscope. In step one we were looking at the whole file. Now we are looking at each line in the file separately.

4. `|`
and then...

5. `identity`
Multiply the single value by 1.

6. `^` 
look at the whole result. Not each individual number.

For the sake of this post, I will assume that everything except for `identity` is already there. I will now add `identity` to make the example above work. The first step is making it so that Racket is aware of `identity` as a term. Since it falls between `|` characters it is already picked up as something that does something. The trick now is to make it available as a "token" of code. That is done by adding this line to the lexer:

`["identity" (token 'OP lexeme)]`

Next, we have to imbue `identity` with what it does. The previous action made it a word. Now we give it meaning.
To do this, we have two options:
1. Make a Racket function that does what we want.
2. Make a Racket macro that turns what we have into Racket code that already does what we want.

I went with option two this time. So here is a macro that generates the Racket code that we want:
```
[((_ "identity")) #'(identity lambda-x)]
[((_ "identity") otherop ...) #'(identity (x-then-ops-in-order(lambda-x (otherop ...))))]
```

This shows the two ways that `identity` can be interpreted. It can either occur last in the list or not.
If it is the last thing, it will hit the top line, which reads essentially 
"if you see something and then the word identity, take that thing and convert it to Racket syntax which applies the Racket function `identity` to `lambda-x`, which is the thing that is being acted upon.

The second option is essentially the same, but it applies `identity` to whatever the other bits in the program spit out. This preserves order of operations. 

Either way, the Racket function `identity` gets run. It is a matter of whether it gets run immediately on the thing you're focused on, or whether there is other stuff in line first and it gets run on the result of that.

That is the whole slice! It is not a complex feature, but it is heartening what one can get done if there is not a ton of technical debt to wade through, and the tools are well tuned to the job. Now for the pitfalls:

## The Pitfalls

### Fake it 'Till You Make it 
This does not work. It is tempting to write the simplest code that will produce the input / output pairs that you want, but if it is done in a tricky fashion, it will be untenable in the long term. Here is an example: 

Remember the scope up and down functions? There was a time when those didn't really do anything, and there was an operator called `/` which would divide things. I made a function called `div-each` which worked only because it had access to all of the data in the file, even though it was only supposed to be looking at one thing at a time. Tricky! Here is how it looked:

```
(define (div-each number)
  (define (divnum x) (/ (string->number x) number))
  (define ans (map divnum data))
  (set! data ans))
```

This did not work, as it makes the function non-portable. Now if I add rules or even write different programs, this function that seemed to work suddenly isn't useful for anything. Making something that produces the right output is nice, but in the long term it is a waste of time if it breaks the semantic model of what you're working with. It doesn't help that it also has a call to set! which makes it necessary to run all actions in order and carries global state. I tried fixing it with the below:

```
[(op "/" NUMBER) #'(map (λ (x) (/ (string->number x) NUMBER)) data)])
```

But that is not moving `map` to the right place. It must be moved to a place that moves with scoping operators. When there is a semantic disconnect between your source and target, it results in all sorts of patches.

## Takeaway
Refactor very early. Refactor here is driven by the difference between what is flexible and what should be flexible, and misses in the semantic mappings between source and target.

A DSL is defined by that which may be assumed. In Fowler’s book "Domain-Specific Languages" he says that the value is in limited domain and limitation of expressiveness to improve clarity and reduce bugs, as well as to make changes easier. If your DSL grows into a GPL, it loses value. I agree with this, but have a different viewpoint. In my opinion, DSLs increase your number of assumptions. Assumptions are things you don't have to think about. They also represent system inflexibility. The more system inflexibility you can introduce, the more flexibility you can achieve in the flexible bits, and the easier it is to add functionality. The issue is that you can get this wrong, and then you’re in a world of hurt. The fix to this is that you have to be very willing to throw away old code. Anything that does not map perfectly to what is desired from the standpoint of what was intended semantically to what is really there must be fixed early on or it will only get worse.

The next question to look at related to these ideas is in practical language oriented programming. I think that these advanced tools make it practical to thin slice delivery of a language without a big up front design. The question is in whether doing so allows for the foresight to know when to split languages, and whether there is strong enough support for disparate language composition. Racket makes it easy to mix domain specific languages so the idea of writing a dozen tiny languages to fix one problem seems more tenable than one would imagine. The ideal for domain specific languages would be something that feels like playing the new Super Mario Odyssey, but with much less time spent as Mario. If you have not played that game, you should try it. The short description is that he can throw his hat and embody bad guys. The controls change each time you do this, but they are inevitably so simple that this is not an issue. In particular, there should be many, many languages that are so small that learning them and using them is effortless.