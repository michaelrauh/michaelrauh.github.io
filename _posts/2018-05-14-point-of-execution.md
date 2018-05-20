---
layout: post
title:  "The Point of Execution in a Program"
date:   2018-05-14 20:59:30 -0400
categories: jekyll update
---

# Point of Execution

## Questions associated with a a program that is difficult to maintain
1. What is the state of the program?
1. How can I add new logic without effecting old logic?

## What is the state of a program?
* The data bound to each variable.

This can be as simple as:
```python
  foo = 5
```
in which case we know that the variable has a value of 5. This rarely causes us to open a debugger. Here is a more typical (though simple) example
```python
  userMinAge = 5
  if userIsLoggedIn():
    userMinAge = 18
  determineElegibility(userMinAge)
```
In this case, we do not know the value passed in to determineElegibility, because it depends on whether the user is logged in. We don't necessarily know whether the user is logged in without entering a debugger


* The concrete type of each interface

  Here is a basic example:
  ```java
  Interface Animal {
    public String speak();
  }

  public class Dog implements Animal {
    @override
    public String speak() {
      return "woof!"
    }
  }

  public class Person implements Animal {
    @override
    public String speak() {
      return "hello!"
    }
  }

  public Class Introducer(){
    public String introduce (Animal animal) {
      return "the animal says" + animal.speak()
    }
  }
  ```

  but for the sake of argument, we could have not used this interface at all. We could have just duplicated the introduce method for each animal type. The interface builds a layer of abstraction that allows us to deduplicate code that is non-specific to implementation. If we bear in mind our Liskov substitution principle, the concrete implementation of our interface type should not matter in the scope of the code that uses the interface. If it does matter, the interfaces are not substitutable and the contract has been broken.


Together, these two facets are described by the stack of the program. There is only one other piece of information that is important - the most important piece of information is - what line of code is running right now? This is why debuggers are breakpoint driven. The first question to answer is always whether your code is running. This is the point of the talk, and is referred to as the point of execution at a point in time. In the ideal case, WHICH code is running should be trivially decidable based upon the architecture of the program. This is closely related to cyclomatic complexity. Data state is a consequence of what code ran in the past, and concrete implementations of the interface should not matter in interfaced code. Therefore, the question becomes "How do I make my code minimally complex?"

## Why is it hard to add new logic without breaking old logic?
1. Logic interferes with itself.
1. When we don't practice open/closed principle, we reuse logic by putting it near other logic. The logic it is near gets altered and breaks

## How do we fix these things?
1. single use - this makes it so that your code doesn't try to do more than one thing. This is SUPER important, as you can't break things you don't touch
1. open closed - this is important as you shouldn't touch code and risk breaking it
The other parts of SOLID are extremely important as well, but they factor less into the idea that I am focusing on at the moment.

## Single use
1. Practice this at write time. When you make a new thing, make it single use. The most obvious and simple metric for this is to ensure that your code has a cyclomatic complexity of one.

## Open closed
1. Practice this at edit time. By that I mean don't edit classes.

## Super Simple Principle
Write your classes such that they won't have to be edited later

## How can I tell if I am breaking these ideas?
1. This is a deep question but I am going to focus on one part of it for this talk. Don't use if. Things that count as if are holding a map of functions, unless, case statements, and loops. As simple example:
```python
if x == 1:
  print "one"
else:
  print "two"
```
This code will need to be edited eventually. Having two cases now means there will be more later. Don't think that now is the only exception.

## Obvious question
If I can't edit code, how do I tell my old code about my new code? Let's come back to this, as it depends. The first way that doesn't break this rule is by using metaprogramming or looping through files to discover new code, but that is cheating. see: Jekyll discovering new blog posts.

## An edge case in this discussion of branching, the obvious question, and open closed
What if I am just formatting data? More scientifically, what if I am not branching for side effect, but am merely trying to do a functional transform on a discontinuous domain? example:
```swift
func fizzbuzz(x: Int) -> String {
    if (x % 15 == 0) {
        return ("fizzbuzz")
    }
    else if (x % 3 == 0) {
        return "fizz"
    } else if (x % 5 == 0){
        return "buzz"
    } else {
        return String(x)
    }
}
```
This code is trying to take in a number, find it on a graph, and return a value based upon that. The graph is all jumpy though. Here is a simpler to graph representation of fizzbuzz:
```swift

func fizzbuzz(x: Int) -> Int {
    if (x % 15 == 0) {
        return 2048
    } else if (x % 3 == 0) {
        return 9001
    } else if (x % 5 == 0){
        return 1337
    } else {
        return x
    }
}
```
Here is the resulting graph for the first 100 points:
![fizzbuzz graph]({{ "https://michaelrauh.github.io/assets/fizzbuzz_graph.png" }})

The point here is that this is a different sort of branching than side-effect driven branching. Here is another way to implement this function:
```swift
func fizz(x: Int) -> String? {
    if (x % 3 == 0) {
        return "fizz"
    }
    return nil
}

func buzz(x: Int) -> String? {
    if (x % 5 == 0) {
        return "fizz"
    }
    return nil
}

func both(x: Int) -> String? {
    if (x % 15 == 0) {
        return "fizzbuzz"
    }
    return nil
}

func neither(x:Int) -> String {
    return String(x)
}

func fizzbuzz(x:Int) -> String {
    return both(x:x) ?? fizz(x: x) ?? buzz(x:x) ?? neither(x:x)
}
```
This is a partial function approach to solving the domain issue. Notice that in the event that the functionality is to be changed, only the fizzbuzz function will be altered. Additionally, the return value of fizzbuzz is not an optional. This means that it supports a full domain. Scala has more native support for partial functions than swift. Check it out! The point of this section is to show two things:

1. not all code can be closed for editing, but most of it can be. In this example, the fizzbuzz function breaks open closed principle. The partial functions don't.
1. Not all branching has side effects, so not all branching requires freaking out and redoing your architecture. Now to discuss the type that does involve side effects:

## Branching for side effect:

First, blasphemy:
```swift
@IBAction func mainButton(sender: UIButton) {
    switch sender.tag {
        case SelectedButtonTag.First.rawValue:
            println("do something when first button is tapped")
        case SelectedButtonTag.Second.rawValue:
            println("do something when second button is tapped")
        case SelectedButtonTag.Third.rawValue:                       
            println("do something when third button is tapped")
        default:
            println("default")
    }
}
```
This is much better:
```swift
@IBAction func buttonOne() {
  println("do something when first button is tapped")
}
@IBAction func buttonTwo() {
  println("do something when second button is tapped")
}
@IBAction func buttonThree() {
  println("do something when third button is tapped")
}
// default will never happen if there are three buttons
```
That was an easy fix! What if we are deeper in the code though?
```swift
@IBAction func buttonOne() {
  doThing(1)
}
@IBAction func buttonTwo() {
  doThing(2)
}

func doThing(x: Int) {
  if (x == 1) {
    println("I guess the first button must have been tapped")
  } else {
    println("I guess the second button must have been tapped")
  }
}
```
That becomes:
```swift
@IBAction func buttonOne() {
  doThingOne()
}
@IBAction func buttonTwo() {
  doThingTwo()
}

func doThingOne() {
    println("The first button must have been tapped")
}

func doThingTwo() {
  println("The second button must have been tapped")
}
```
This strategy can be applied recursively until there is no branching anywhere in the code. You'll end up "unzipping" the program like so:

![unzipping schematic]({{ "https://michaelrauh.github.io/assets/unzipping.png" }})

That concludes this blog post.

But what about serial data, you ask?

## The router delegate pattern
1. Pass the input into a class called somethingRouter
1. somethingRouter does magic and eventually calls a function that belongs to the original class
1. The original class was a controller. Now you have callbacks from the network, buttons, and your router all in one place
1. If you ever forsake this pattern, that will not harm you. Each split apart bit of code will have branching, but it will be more maintainable than if it had not been split. Worst case they all call the same helper if you want to deduplicate your code. If that helper must deal with different types, use an interface. Polymorphic dispatch will take care of you then, but ultimately it is just a trick to reuse code.
1. This gets at the essential idea - execution is all that matters. If you have serial data, convert it to execution as much as possible and then let that execution ride without branching. You'll end up with a beautiful web of code where it has one point that touches and the rest is a line out of the center.A fuzzball basically

```swift
@IBOutlet age Int!

@IBAction func userDidTapContinueButton {
  if (age > 17) {
    println("WELCOME!")
  } else {
    println("GET OUT")
  }
}
```
This seems impossible to fix. From a certain perspective, it is. We can give it an amalgam of the partial function treatment, though.

```swift
protocol RouterDelegate {
  func userWasOldEnough
  func userWasUnderage
}

class viewController: RouterDelegate {
  override func viewDidLoad() {
    super.viewDidLoad()
    router.delegate = self
  }

  @IBAction func userDidTapContinueButton {
    router.route(age)
  }

  func userWasOldEnough() {
    println("WELCOME!")
  }

  func userWasUnderage() {
    println("GET OUT")
  }
}

class Router {
  public delegate: RouterDelegate?

  public func route(_ age: Int) {
    if (age > 17) {
      delegate.userWasOldEnough
    } else {
      delegate.userWasUnderage
    }
  }
}
```
So we have replaced one class with a complexity of 2, with two classes, with a complexity of 1 and 2. Complexity has gone up. Is this better?
Yes. Now, adding functionality to the app is a matter of adjusting routing. Routing can be done either through adding callbacks directly, parsing
user input in a router, or by parsing service responses in a router. A single controller may have many routers, but it should not have any internal branching. Here is a basic schematic:

![router schematic]({{ "https://michaelrauh.github.io/assets/router.png" }})

## What if I am deep in legacy code?
1. If you can't go all the way with it, do this to just a section of the code. Start in a controller and call it a day. Any branching that takes place outside of a router should be split up until it ends up in a controller callback or a router that calls a controller callback
1. If the branching occurs too deeply and you don't want to commit, but you want to get some of the benefit, insert the strategy pattern. Branching occurs for one of three reasons.
  1. You need to transform data in a way that calls for a partial function
  1. You need to take serial data and perform different actions depending on domain (routing problem)
  1. You used to know your point of execution but now your code paths have collapsed (you have two different calls to the same function)
If it is the last reason at play here, then set the appropriate state on a state holder in each caller to the function. This doesn't have to be a direct call to the function, it can be an eventual call.

# The benefit
Once all of the code is split apart, adding functionality is a matter of adding a new class to your code and adding a new callback. You don't have to mess with the old code, you don't break things, and you still get to reuse code because you broke out repeated sections of code into bits that just take an interface type
