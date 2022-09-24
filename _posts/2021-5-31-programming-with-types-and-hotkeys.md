---
layout: post
title:  "Programming with Types and Hotkeys"
date:   2021-5-31 00:00:00 -0400
categories: jekyll update
---

## Introduction
People who have been programming for more than fifteen minutes have probably already gotten into a hairy debate about which type systems are good, and which ones are bad. As a big fan of `Scheme48`, `Racket`, `Python`, `Haskell`, and now `Idris2`, I have been developing opinions as well. As a short version, I am coming to the belief that there are a few type systems that are useful for different domains, and each of those systems has varying levels of quality by implementation.

## Disclaimers
Due to how vague these ideas are, I am not going to substantiate them much. Feel free to ask! Also, terminology and categories will be loose here. You can have more categories than this by mixing properties, but these are the most popular ones. `J` for example is in a funny area with `Smalltalk`, `mypy` and `numpy` make things confusing, `Scheme` is ideologically heterogenous and would require a separate post, and I don't understand `Elixir` well enough to comment. Arguably `Typed Racket` is similar to `Haskell`, but it is gradual. `Rust` has some cool things in it but the category it lives in is not populated enough to put other systems down unless you reach for less popular languages like `Clean` and `Formality` so it wins in category by default. `F*` and `Liquid Haskell` come to mind as having interesting systems as well that fall outside of these comparisons. I don't know `Typescript` but it looks cool. `Swift`, `Kotlin`, and `Scala` oddly fall into the nominal category for me. If you're disciplined you can leverage `case` `class`es for everything in `Scala` and end up with a `Haskell`-like feel, but the presence of classes removes a feeling of static safety for me. `Go` is oddly good but seems to stand alone. I am still getting acquainted with `Nim` and `Zig`. `Crystal` is nice but it takes a different tack. No opinion given on `Pony`, `Factor`, `Forth`, `Oz`, `Maude`, `Prolog`, `Mercury`, `Julia`, `Agda`, though `Koka` may be the future. I have forgotten `OCaml`. `Catln` is probably the language that is interesting me most after `Idris` but I don't understand it yet.

## Specifics
Here are some type domains:
1. `dynamic`, `weak`, `duck` typing (`Python` does this well. `Ruby` less well. `Javascript` poorly.)
1. `static`, `strong`, `nominal` typing (`Java` does this well. `C++` does this poorly.)
1. `static`, `strong` (what I think of as algebraic) typing (`Haskell` does this well. I am setting up to argue that `Idris` does it better.)
1. Swathes of grey areas inhabited by a mix of macros, contracts, assertions, and algebraic types in `Racket`. (This is why I reach for `Racket` first these days. If you don't know what you're doing you may as well be prepared to use anything and everything.)

## The Actual Argument

### Python
`Python` is really good in rapid prototyping and data discovery domains. It shines in domains where I can open a large CSV and start crunching. It has only hurt me a couple of times, and those times were always related to integrating my code with someone else's code. Within the data domain, `Python` never lets me down. `Ruby` does sometimes, so I reach for it less.

### Java
I love to complain about `Java`, but it is very, very good for calling other people's code and being surprised relatively rarely. Sometimes people break Liskov Substitution Principal (for example, https://stackoverflow.com/questions/25624251/list-addall-throwing-unsupportedoperationexception-when-trying-to-add-another-li/25624283)
When people are not doing terrible things (oddly most of the time) I know that I can call `Java` code and get good results. Working with APIs in `Java` is easy because I don't have to think about what types I am giving and what types I am getting. There is less surprise in integration. Oddly, `Scala` is more advanced but gives more surprise with `traits` and `implicits`. `Swift` is better most of the time, but it is inconsistent with how it handles overrides which can lead to confusion and naming mistakes. (To be clear I would choose `Swift` over `Java` 100% of the time due to its better handling of optionals and better support for functions. This is a nitpick that is focused on integration only)

### Haskell
`Haskell` is great for thinking in terms of invariants. To me, this is the magic of functional programming. Being able to trust things makes systems less stressful to work in. You can cover all possible cases in your code without having to be too creative. The type system helps quite a bit (notably I feel this way in `Scheme` too but for opposite reasons. Everything is a list so there are no surprises.). `Idris` takes this to the next level. The rest of this is about `Idris`.

## Idris
`Idris` is a brilliant language that takes ideas from `Haskell` and (in my opinion) smooths out the inconsistency to make something more ergonomic and more powerful. Modern `Haskell` is known for being covered in `pragmas` to enable extensions that everyone likes such as `GADT`s There is also a large contingent of people who want simple `Haskell` (I don't disagree with them at all. Using `Free Monads` as part of a standard `CRUD` stack seems a little bit suspect to me still). `Idris`, on the other hand, just builds these in. The biggest selling point, however, is in how it treats types as first class. `Haskell` puts a wall up between types and data but in `Idris` this wall is gone. This means that `Idris` is simpler, because it is more unified and you don't have to learn two domains. It is also much more powerful because you can tell the compiler what you want and then it can tell you what you can do to get it.

`Idris2` changes things up and is built on `Quantitative Type Theory` (It adds support for counting uses 0, 1, unbounded. It can be compared to linear types but with type families. This is oddly compatible with `Idris` as it supports type families natively [another `Haskell` pragma] [If I am getting the details wrong here, please let me know. I am still getting into the literature https://bentnib.org/quantitative-type-theory.html]). It also takes some of the terminology and makes it more familiar to most programmers. `Type classes` are called `interfaces`, and `parametric polymorphism` is referred to as `generics` and `overloading`. `return` is now `pure` (hindsight is 2020) and `data`, `type`, and `newtype` from Haskell are replaced with `data`, regular type level functions, and `GADT`s. In short, Idris goes back over the parts of Haskell that make it confusing, and simplifies out the rough edges. Part of this is possible due to first class types (such as eliminating `newtype`) but part of this is a result of learning from the past (eliminating `return` for example - this came about to make `do` notation more understandable, but it was developed before the discovery of `applicative` `pure`, and people are even more confused having both `pure`, and a `return` which is nothing like `Java`'s `return`)

In terms of the experience of programming in `Idris`, I assumed it would be like programming in `Haskell` but more annoying. I would lose inferred types, and would gain more up-front nitpicking of little details I don't care about in my type system. This stopped me from getting started for years. The reality is so far from that vision that it is hard to put across without an example. This is stolen from https://www.manning.com/books/type-driven-development-with-idris but in my own words:

### Example
Assume you have two lists that are of the same size. You want to make a third list that is the same size as the original two, but with each element in each list paired up with the corresponding one. For example:
`zip [1, 2, 3] ['a', 'b', 'c']` equals `[(1, 'a'), (2, 'b'), (3, 'c')]`

The type for this looks like:
`zip : Vect n a -> Vect n b -> Vect n (a, b)`

`n` above is the length of the lists. `a` is the type of things inside the first list, and `b` is the type of things in the second list.

When you put that signature in to an editor, you can have it generate a skeleton which must be correct using a hotkey.

```
zip : Vect n a -> Vect n b -> Vect n (a, b)
zip xs ys = ?zip_rhs
```

This says that `zip` takes in two vectors named `xs` and `ys` and returns something.

You can then split `xs` across all possibilities automatically:

```
zip : Vect n a -> Vect n b -> Vect n (a, b)
zip [] ys = ?zip_rhs_1
zip (x :: xs) ys = ?zip_rhs_2
```

This says that `xs` is either empty, or it is not. If it is not, then it has a first element, and then everything else. You can think of `xs` as a linked list.

Then you can split `ys` automatically.

```
zip : Vect n a -> Vect n b -> Vect n (a, b)
zip [] [] = ?zip_rhs_3
zip (x :: xs) ys = ?zip_rhs_2
```

Doing this shows some tricky logic. `Idris` knows that since `xs` and `ys` are the same length, empty `x` implies empty `y`. So there is no case where `x` is empty but `y` is not. Then you can ask it to guess the answer for the empty case:

```
zip : Vect n a -> Vect n b -> Vect n (a, b)
zip [] [] = []
zip (x :: xs) ys = ?zip_rhs_2
```

And there is only one variable in scope that satisfies the type constraints: the empty list.

Splitting `ys` in the second part gives you:

```
zip : Vect n a -> Vect n b -> Vect n (a, b)
zip [] [] = []
zip (x :: xs) (y :: ys) = ?zip_rhs_1
```

(we know that `y` is not empty because `x` is not. The vectors being the same length means there are only two cases, not four.)

Searching for the answer for the last bit, we get:

```
zip : Vect n a -> Vect n b -> Vect n (a, b)
zip [] [] = []
zip (x :: xs) (y :: ys) = (x, y) :: zip xs ys
```

Which makes sense when you think about it. We know that we need something of type `(a, b)` and we have an `a` and a `b`. We can't have any free variables that happen to be those types, because we don't know those types. Therefore we are restricted to `a` and `b` being passed in. Further, we need something of type `Vect n - 1 (a, b)` (this is conceptual and not a real type. `Idris` uses Peano style `Nat`s.) and we know that recursion gives us that.

This process shows that with specific enough types, some programming is tasks are trivially solvable. I am not aware of a formal specification on this, but a hunch tells me that there is some ratio of complexity between the type level programming and the runtime level programming that leads to this situation. If we put less info into the type system (for example, if we remove the length restriction on the vectors) then the solution gets more complex. We can also arbitrarily add complexity to the types and gain no benefit if we like.

On the other hand, what about the quality of the code that comes about? It is arguably ultra-high quality. There is a focus on making things total, and the language uses monads for effects. What this results in is what I think of as a high likelihood to pass a theoretical test that I am considering writing known as the data strength test. The test goes like this:

1. You write five functions.
1. I write a function combinator.
1. When the test is run, the combinator runs the functions in random order and combines the result. This is done under dirty, random input with lots of nulls, empty fields, wrong types, etc.
1. The result is tested to make sure it is correct.

In `Idris`, you are likely to write code that does well on this test. In `Java`, your code is likely to suffer. There are a few reasons for this:
1. Total functions means that you are encouraged to think about what your code will do for all inputs. You are also discouraged from writing code that will not terminate.
1. Functional purity means that when the functions are run in random order, that does not impact what they return (referential transparency is most likely a better word here). They are not, for example, changing the state of a global variable out of order.

Statistically, most bugs occur when a function gets data that is unexpected or at the wrong time. This system does not make these bugs impossible to write (the language is Turing Complete after all) but it points out to the user when they are about to do a bad thing, and suggests a good thing instead.

The takeaway here is that, while I feel that programming in `Haskell` is safer and more satisfying than programming in `Java`, I do not feel like it thinks for me. `Idris` goes the next step and tells you what it is that you have just implied by making the types so. `Haskell` is more of a mind to ask you to imply something and point out contradiction a moment later. That is nicer than `Javascript` telling you you're brilliant until the application crashes, but it still leaves something to be desired. In terms of how it feels to edit code, there is a really nice continuum of safety here. One common operation is to add a parameter to a function. In `Python` this can break things. In `Java` it warns you at compile time that the receiver must have the same arity as the caller, but it does not constrain you beyond that. `Haskell` has a similar feel (though return type constraints are another matter). In `Idris` you come in with type families and a totality checker out of the box. Changing any part of your code starts to beg the question: Have you changed the available cases? Any new guarantees? If so, maybe the search space of possible correct code is smaller. In the book `TDD with Idris` the title purposely harkens to `test driven development` but with types. This seems wildly appropriate to me. Writing `Idris` with the book's `Type, define refine` paradigm (and I am still getting started) feels an awful lot like a `red, green refactor` paradigm but with a friend in the computer who points out when the next bit of code can be simpler or is even statically knowable. In short, working in `Idris` feels oddly similar to pair programming in `Haskell`, where the tools are your pair partner. It really is a major positive change.

As a last odd note, there is something that Idris and Racket have in common that is missing in other systems: Strong Runtime / Compile time separation. In Racket, macros only use the info on disk to rearrange syntax. This could be part of a parsed DSL or pure Racket code. In Idris, actions result from running the program, and then calling exec on them makes them go. This is how Haskell works as well, but they hide the exec part which lowers the feeling of separation. For example:

```
ghci > getLine
a
> "a"

idris> getLine
io_bind prim_read
        (\x =>
           io_pure (prim__strRev (with block in Prelude.Interactive.getLine', trimNL (MkFFI C_Types
                                                                                            String
                                                                                            String)
                                                                                     (prim__strRev x)
                                                                                     (with block in Prelude.Strings.strM (prim__strRev x)
                                                                                                                         (Decidable.Equality.Bool implementation of Decidable.Equality.DecEq, method decEq (not (intToBool (prim__eqString (prim__strRev x)
                                                                                                                                                                                                                                           "")))
                                                                                                                                                                                                           True))))) : IO String

:exec getLine
a
"a"
*temp>

```

## conclusion
There are many ways to classify type systems, but some of the classifications tend to move together for reasons of feasibility and ergonomics (Crystal being the elephant-in-the-room counterexample [though it may be a late-arriver for feasibility reasons]). One way I think about type systems is about how easy it is to understand the consequences of change, and how easy it is to understand the risks of the inputs you get. Java, Haskell, Python, and Idris all excel at this, and Racket is a favorite for being a jack-of-all-trades.