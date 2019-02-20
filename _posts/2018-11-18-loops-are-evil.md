---
layout: post
title:  "Loops are Evil"
date:   2018-11-18 20:59:30 -0400
categories: jekyll update
---

# Loops and Evil
Similar to the dreaded `if` statement, people tend to write a lot of loops in their software. This is not necessarily seen as a bad practice. Some draw the line at nested loops, others just don't like to see loops that go on too long. In this post, I am going to argue that most loops are evil. I am focusing on `for` loops here, as I am assuming that `while` loops are a known evil.

# What is a `for` loop?
I am assuming that we all know what a `for` loop is. That said, there are some aspects that bear pointing out. Loops all have some things in common. They all have:
1. An iterable - the thing that is being looped over
1. Some form of index, pointer, or iterator that will keep our place

# What is so evil about that?
There is nothing evil in the parts of a loop. The truly evil thing is the way they can drive your code. There are two purposes to loops.
1. Run code that produces many side effects
1. Build a new collection that is like an old collection. This new collection may be the same size as the old collection, or it may be a different size.

# How do people use loops?
A loop can be visualized as a conveyer belt that is taking in one object at a time from a box (yes, I play factorio). While the object is on the conveyer belt the object has all sorts of things done to it. Here is an example of a loop taken from a totally real program written in python:

```
for pokemon in pokemons:
  pokemon.heal()
  pokemon.rest()
  pokemon.eat()
```

This is a loop that applies the same side-effect driven code to each pokemon. It is actually a fairly harmless loop. In instances where there are no conditionals, and there is no functional transform going on, looping for side effect actually makes sense. Here is a less harmless one:

```
evolved_learned_pokemons = list()
for pokemon in pokemons:
  evolved = pokemon.evolve()
  evolved_and_learned = evolved.learn_attack()
  evolved_learned_pokemons.append(evolved_and_learned)
```

This loop has a degenerate accumulator called `evolved_learned_pokemons` which starts out empty. At first glance, making the list is a pointless exercise. It does not help in any way. It only makes sense in the greater context. A better approach would be:

```
evolved_pokemons = [pokemon.evolve() for pokemon in pokemons]
evolved_learned_pokemons = [pokemon.learn_attack() for pokemon in evolved_pokemons]
```

This is a much stronger pattern. Instead of having a long conveyer belt, we have many short conveyer belts and the objects spend time in collections between operations. It also doesn't involve mutating the state of an accumulator over and over again. The value of this becomes more evident when you consider loops that return a smaller collection such as this one:

```
evolved_learned_fire_pokemons = list()
for pokemon in pokemons:
  evolved = pokemon.evolve()
  evolved_and_learned = evolved.learn_attack()
  if evolved_and_learned.isFireType():
    evolved_learned_fire_pokemons.append(learned)
```
Suddenly, the loop becomes trickier to deal with. If you want to act on everything, you need to act in the loop, but outside of the `if` statement. This is especially important if there are side effects, such as a pokedex update. A simpler pattern is:

```
evolved_pokemons = [pokemon.evolve() for pokemon in pokemons]
evolved_learned_pokemons = [pokemon.learn_attack() for pokemon in evolved_pokemons]
evolved_learned_fire_pokemons = [pokemon for pokemon in evolved_learned_pokemons if pokemon.isFireType()]
```
This results in having the same list as was obtained at the end, but it is simpler to debug, easier to add to, and easier to read. There are some concerns with this style though.

# Performance
I've noticed that people tend to have performance concerns with this style. Most of the time, this pattern will not produce a big enough performance hit to be an issue (appending to a list many times in python can be though). If performance is a concern, this style does not have to have worse performance. Lazy evaluation and method chaining will optimize this code to be as efficient as a plain loop. If there is a slight performance hit and a major improvement to maintainability, then that should most likely be counted as an improvement.

# The short version
`for` loops tend to balloon into unmaintainable messes. Once a loop stops being about getting the same side effect to happen over and over again and becomes about building collections, using more functional oriented `map` and `filter` style functions (in python a list comprehension can be either) will produce more maintainable code. This is especially true if the size of the collection being returned is different from the input collection. In the big picture, this is simply because it allows each operation to occur without the influence of the others, and allows the code to be more single responsibility and less context dependent.
