---
layout: post
title:  "Using Language Oriented Programming in Tandem with Extreme Programming Practices"
date:   2020-12-06 00:00:00 -0400
categories: jekyll update
---

## Introduction

I like [Language Oriented Programming](https://en.wikipedia.org/wiki/Language-oriented_programming) (LOP) a lot, but I am allergic to doing the dreaded big up front design. Pretty much all LOP and [Domain Specific Language](https://en.wikipedia.org/wiki/Domain-specific_language) (DSL) materials assume or recommend a design phase. 

Last year, I set out to do LOP without any foresight beyond that required to design the code I would write that day. The challenge that I chose as my framework for this was [Advent of Code](https://adventofcode.com) (AoC) 2019. There were two rules to this challenge: 

1. I was not allowed to use any existing programming language to solve any part of any of the problems. 
1. I was not allowed to look past the current prompt until it was completed.
1. I could only use the [Racket](https://racket-lang.org) ecosystem. I chose this after looking at the available language workbenches. (This was rule 3/2)

Notice that there was no rule that they must all be solved in the same DSL. It is perfectly okay to build on to an old DSL or to start over.

When it comes to the choice of using Racket, I compared the seven entrants to the [Language Workbench Challenge](https://2016.splashcon.org/track/lwc2016#event-overview) and came away liking two of them. Racket won out over MPS because MPS has vendor lock, is closed source, and reminds me of Java. I like MPS but not as much as Racket. There was also the option to do it in some other language which is not a language workbench. This would have been a poor choice in my opinion because language workbenches allow for rapid prototyping, and that seemed necessary if I wanted to get anywhere quickly. I did not want to go to the trouble of implementing an interpreter in Haskell, Python, Java, or C, because those languages seemed better suited to implementing languages which had a design already. Language workbenches seemed better for designing a language rather than implementing them. When it comes to language implementation, [Crafting Interpreters](http://craftinginterpreters.com) has an excellent handling of the subject, and even it prototypes the language in Java before moving to C. Prototyping in Racket seemed like the next step on that spectrum to me. If you're interested in learning Racket, I recommend [The Little Schemer](https://www.amazon.com/Little-Schemer-Daniel-P-Friedman/dp/0262560992) for fundamentals of Scheme, [SICP](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book.html) for great material on using Scheme to build data abstractions and function abstractions, [Realm of Racket](https://nostarch.com/realmofracket.htm) for learning general purpose Racket, [Fear of Macros](https://www.greghendershott.com/fear-of-macros/all.html) for macros in Racket, and [Beautiful Racket](https://beautifulracket.com) for LOP.

The idea behind the AoC LOP challenge was to simulate a Speed to Value delivery style project, where you don't have the luxury of looking too far ahead. AoC was also chosen for its broadness. It is a stress test of sorts because the domain of AoC is very large. The only thing the prompts have in common is that they all read one file and produce one output that can be typed in to a box. In short, AoC is unpredictable like a real world problem, but broader than a a good deal of real problems. It also has the charm of not constraining your tool set, and not including anything that requires user input or networking.

A year later, I have made a lot of mistakes and have not managed to complete the first prompt. Obviously this could be viewed as an abject failure. Therefore I am writing this post to go back in time and deliver it to myself. Its main message is that I was correct in assuming that I didn't need a design phase. 

The reason I was not able to complete this in a timely fashion was because I did not know about the major pitfall involved in this sort of task. In this post I will talk about two things:
1. What a thin slice of a language feature can look like in the best of times.
2. Warning signs for when you have gone off the rails and are trending toward spending eleven months on a twenty minute task.
3. Another shout out to the best LOP book: [Beautiful Racket](https://beautifulracket.com). I would not have gotten anywhere without it. It is an inspiration, and everyone should buy it. (This is thing 3/2).

## Case Study: Adding the Identity Built in Function to AOCLOP

First, let us start with a line of code from a made up tiny DSL. In this language, all programs are a single line of code. I named it aoclop (pronounced eyoo-clop) for mysterious reasons. This is a program that takes in a file which has a column of numbers in it, and converts it into a list of those same numbers. I call this program `cat`. Note that `cat` is not a prompt from Advent of Code. It is easier than the easiest prompt, so it seemed like a good baby step.

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
look at the whole result. Not each individual number."

For the sake of this post, I will assume that everything except for `identity` is already there. I will now add `identity` to make the example above work. 

## Syntax

The first step is making it so that Racket is aware of `identity` as a term. Since it falls between `|` characters it is already picked up as something that does something. This is a result of a bit of magic in the parser (written in [brag](https://docs.racket-lang.org/brag/)) which looks like:
```
#lang brag
aoclop-program : /NEWLINE read scope-block /NEWLINE
read           : /READ INTEGER delimiter
scope-block    : /PIPE /DOWNSCOPE /PIPE all-ops /PIPE /UPSCOPE
delimiter      : DELIMITER
all-ops        : op*
op             : OP | OP /PIPE
```
There is a lot here, but the important part is that an `op` is something that falls between pipes. The pipes are among the symbols that are preceded by a "/" character (meaning "cut"), so they are used for parsing, but they are dropped before they get any further. An `aoclop-program` has a `read` statement, then a `scope-block`. A scope block has a `pipe`, a `downscope`, a `pipe`, some number of `ops`, a `pipe`, and then an `upscope`. This means that `identity`, as something that falls between pipes, will be considered an `op`. I have left out some glue code because it is boring and will confuse you. If you really want to see the connections they are in `main.rkt` and `tokenizer.rkt` but I don't expect to update those for awhile.

The trick now is to make `identity` available as a `token` of code. That is done by adding this line to the lexer:

`["identity" (token 'OP lexeme)]`

After this addition, the lexer looks like:

```
(define aoclop-lexer
  (lexer-srcloc
   ["read:" (token 'READ lexeme)]
   ["\n" (token 'NEWLINE lexeme)]
   ["nl" (token 'DELIMITER lexeme)]
   ["floor" (token 'OP lexeme)]
   ["identity" (token 'OP lexeme)]
   ["|" (token 'PIPE lexeme)]
   ["^" (token 'UPSCOPE lexeme)]
   ["v" (token 'DOWNSCOPE lexeme)]
   [digits (token 'INTEGER (string->number lexeme))]
   [whitespace (token lexeme #:skip? #t)]))
(provide aoclop-lexer)
```
Together the tokenizer, parser, and lexer can turn: `read: 1 nl | v | identity | ^`

into this: `(aoclop-program (read 1 (delimiter "nl")) (scope-block (all-ops (op "identity"))))`

complete with all sorts of nice things like source location so that errors can be reported to the correct place in the code. Conveniently, the above string looks like a lisp s-expression. This makes it really easy to write macros and functions for. In the big picture, this is already code. It simply has no semantic bindings.

## Semantics

Next, we have to imbue `identity` with what it does. The previous action made it a word. Now we give it meaning.
To do this, we have two options:
1. Make a Racket function that does what we want.
2. Make a Racket macro that turns what we have into Racket code that already does what we want.
3. Both (this is option 3/2).

I went with option two this time. This was probably a bad choice but I like macros. 

Below is the line of code that handles the identity function's semantics. This is taken out of context for simplicity, but the context will be there in the next block if that makes you nervous. "#'" is shorthand that generates a Racket syntax object. Macros in Racket take in syntax and produce syntax. This is useful because there is more to syntax then you see. There is a concept of source location and bindings. Some macro forms hide the "#'" behind still more macros, but that does not mean they are not there.

```
[(op x "identity") #'(identity x)]
```

This shows how `identity` can be interpreted. 
"if you see an op, and then something, and then the word identity, take that thing and convert it to Racket syntax which applies the Racket function `identity` to the thing.

It is a part of this larger function that turns operations into Racket code ("stx" is shorthand for "syntax"):

```
(define-syntax (op stx)
  (syntax-case stx ()
    [(op x "identity") #'(identity x)]
    [(op x "floor") #'(floor x)]))
(provide op)
```

That in turn has to be called from somewhere:
```
(define-syntax-rule (aoclop-program read-expr (scope-block (all-ops op ...)))
  (map (λ~> op ...) read-expr))
(provide aoclop-program)
```
The `define-syntax-rule` form is a convenient macro that allows you to take in one pattern and produce a different syntax pattern. The input to this is the entire parsed program. The output is a mix of Racket code and more half-baked code to grab with a macro later.

`op ...` is every operation. This macro is merely surrounding the `op`s with a map call (to replace the scope block) and a [threading macro](https://docs.racket-lang.org/threading/index.html) (to simulate function chaining). `λ~>` is a threading macro that takes a series of functions and threads `x` through them in order. That is why the receiver macro `op` takes in `x` even though you don't see it. `λ~>` is the progenitor of this phantom `x`. With this threading macro in place, `x` will be substituted in at first with a single number from the file, and then with any ops run before.

For the input:
`(aoclop-program (read 1 (delimiter "nl")) (scope-block (all-ops (op "identity") (op "floor"))))`

You will see:

```
(op x identity)
(op (op x identity) floor)
```

passed in to the macro. The first time, `x` is a `x` (a number from the list we are mapping over), and the second time `x` is `(op x identity)` which evaluates to a number eventually.

That is the whole slice! It is not a complex feature, but it is heartening what one can get done if there is not a ton of technical debt to wade through, and the tools are well tuned to the job. Now for the pitfall:

## The One Big Pitfall
It is tempting to write the simplest code that will produce the input/output pairs that you want. If you do that but make the wrong sort of shortcut to get there, you will regret it.

Remember the scope up and down functions? There was a time in the misty past when those didn't really do anything, and there was an `op` called `/` which would divide things. I made a function called `div-each` which worked only because it had access to all of the data in the file, even though it was only supposed to be looking at one thing at a time. Here is how it looked (ignore that it also mutates global state. I already said this was a mistake):

```
(define (div-each number)
  (define (divnum x) (/ (string->number x) number))
  (define ans (map divnum data))
  (set! data ans))
```

This did not work, as it makes the function non-portable. Now if I add rules or even write different programs, this function that seemed to work suddenly isn't useful for anything. Making something that produces the right output is nice, but in the long term it is a waste of time if it breaks the semantic model of what you're working with. 

A design phase could have prevented this kind of issue. If I had bothered to design up front a bit, I could have just made it a part of the design that the `scope-block` is semantically equivalent to `map` and that would shake out. 

Without the design phase it becomes necessary to make sure that your code is consistent at all times. Because there is no "gate" of design, there is no time at which you make sure that you're not breaking your own rules. Therefore, consistency must be achieved continuously or at the least at a branch or commit level. If you make a scope block in your syntax and ignore it semantically, you're breaking your design, whether that design is expressed in a design document or in code. This has sort of an Extreme Programming flavor so I find it appealing. Note that I am not talking about [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) here. There are different opinions on that, and I am not against sprinkling or even dumping syntactic sugar into a DSL. I am talking about when one piece of syntax steals another piece of syntax's semantics. When you look at a specific macro or function in your expander, you should not see that function or macro introducing semantics that should be attributed to a different function or macro.

## Takeaway
Be very careful about implementation at every phase in order to assure syntactic and semantic consistency. Misses in the semantic mappings between source and target will only get worse over time.

## Philosophical Rambling 
A DSL is defined by that which may be assumed. If your DSL grows into a General Purpose Language (GPL), it loses value. This concept is handled very well in [Domain-Specific Languages](https://martinfowler.com/books/dsl.html). DSLs increase your number of assumptions. Assumptions are things you don't have to think about. They also represent system inflexibility. The more system inflexibility you can introduce, the more flexibility you can achieve in the flexible bits, and the easier it is to add functionality. The issue is that you can get this wrong, and then you’re in a world of hurt. The fix to this is that you have to be very willing to throw away old code. Anything that does not map perfectly to what is desired from the standpoint of what was intended semantically to what is really there must be fixed early on or it will only get worse. Of course, this is a moving target as features are added.

## Next Research Questions
The next question to look at related to these ideas is in practical LOP. I think that these advanced tools make it practical to thin slice delivery of a language without a big up front design. The question is in whether doing so allows for the foresight to know when to split languages, and whether there is strong enough support for disparate language composition. Racket makes it easy to mix domain specific languages (see the above example with Brag) so the idea of writing a dozen tiny languages to fix one problem seems more tenable than one would imagine. The ideal for DSLs would be something that feels like changing tools while working on a carpentry project. In particular, there should be many, many languages that are so small that learning them and using them is effortless. It would be odd if people said they only did carpentry using saw, or only using lacquer. That does not even make sense. In practice, there is a general purpose tool (hand) which drives switching of small, highly specialized tools. This is the ideal that software development should aspire to. The two specific questions become:
1. Can LOP be done quickly without an up front design? I am sure it can be done, but am not sure about how long it will take.
1. Is the judgement required to decide when to continue expanding a DSL vs. starting a new one beyond what people are capable of?
1. What language should be used to glue DSLs together? Another DSL, or the implementation language of the DSLs (this is question 3/2)?

## Next Steps
I am planning to keep working on [aoclop](https://github.com/michaelrauh/aoclop) even though it is way over budget and over schedule. If you would like to help, or even just want to talk about it, please let me know! There is a lot that did not make it in to this post.