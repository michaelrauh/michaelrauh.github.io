---
layout: post
title:  "A Few Patterns for Callbacks"
date:   2019-04-21 08:59:30 -0400
categories: jekyll update
---

# Callbacks
Event driven systems tend to use either plain callbacks, or some abstraction on callbacks (such as data-binding or promises). For the sake of this post, I am thinking of plain callbacks, as are popular in Android pre-data-binding.

# A Rough Example

```
class Callback {

    private Callbacker callbacker;
    private Doer doer;

    public Callback() {
      doer = new Doer();
    }

    void bind(Callbacker callbacker) {
        this.callbacker = callbacker;
    }

    void doCallback() {
        callbacker.doThing();
    }
}

class Callbacker {

    private Callback callback;

    public Callbacker() {
        callback = new Callback();

        callback.bind(this);
    }

    void doThing() {
        Log.i("CALLBACKER", "made it");
    }
}

class Doer {
    public int doIt(String ignored) {
        return 5;
    }
}
```
This code demonstrates several antipatterns in making callbacks. We can make several enhancements to the code without completely overhauling it.

## Enhancements


The first issue with the example is that there is no dependency injection. This is better:

```
class Callback {

    private Callbacker callbacker;
    private Doer doer;

    @Inject
    public Callback() {}

    void bind(Callbacker callbacker, Doer doer) {
        this.callbacker = callbacker;
        this.doer = doer;
    }

    void doCallback() {
        callbacker.doThing();
    }
}

class Callbacker {

    private Callback callback;

    @Inject
    public Callbacker(Callback callback) {
        this.callback = callback;

        callback.bind(this);
    }

    void doThing() {
        Log.i("CALLBACKER", "made it");
    }
}

class Doer {

    @Inject
    public Doer() {}

    int doIt(String ignored) {
        return 5;
    }
}
```
The next demerit of this code is the fact that `bind` must be called before the callback occurs, or else an exception will occur. This can be fixed by binding inside the constructor, but using dagger for this introduces a circular dependency. The fix here is to not use dagger for that particular field, but to use it for any other fields. I think you see where I am going. I am going to have to say the F-word.

```
class Callback {

    private Callbacker callbacker;
    private Doer doer;

    public Callback(Callbacker callbacker, Doer doer) {
        this.callbacker = callbacker;
        this.doer = doer;
    }

    void doCallback() {
        callbacker.doThing();
    }
}

class CallbackFactory {

    private Doer doer;

    @Inject
    public CallbackFactory(Doer doer) {
        this.doer = doer;
    }

    public Callback create(Callbacker callbacker) {
        return new Callback(callbacker, doer);
    }
}

class Callbacker {

    private Callback callback;

    @Inject
    public Callbacker(CallbackFactory callbackFactory) {
        this.callback = callbackFactory.create(this);

    }

    void doThing() {
        Log.i("CALLBACKER", "made it");
    }
}

class Doer {

    @Inject
    public Doer() {}

    int doIt(String ignored) {
        return 5;
    }
}
```

That is, factory. Making a factory allows for partial binding. Since the callback is temporally bound (temporal binding is evil but sometimes unavoidable), it can't be supplied through dagger. That doesn't mean that the entire component must be carved out, though. Doer can still be supplied and bound at build time, then mixed in to the callback at create time. Now the biggest remaining issue is that, when we look inside of `Callbacker`, we have no idea that `doThing` is a callback method. We see a function that seems to just be sitting there, unused. To make it more clear that this function is called as a result of some event that is not under its control, it is better to put this method behind an interface. Now a casual glance shows the override annotation in the class. Another advantage to this is that this callback can then be replaced without too much trouble. Additionally, if there are many callbacks with the same pattern, a container could be made to hold a collection of the interface types. Most importantly, it hides any information that is not necessary to the callback. This ends up looking like below:

```
class Callback {

    private DoesThing doesThing;
    private Doer doer;

    public Callback(DoesThing doesThing, Doer doer) {
        this.doesThing= doesThing;
        this.doer = doer;
    }

    void doCallback() {
        doesThing.doThing();
    }
}

class CallbackFactory {

    private Doer doer;

    @Inject
    public CallbackFactory(Doer doer) {
        this.doer = doer;
    }

    public Callback create(DoesThing doesThing) {
        return new Callback(doesThing, doer);
    }
}

class Callbacker implements DoesThing {

    private final Callback callback;

    @Inject
    public Callbacker(CallbackFactory callbackFactory) {
        this.callback = callbackFactory.create(this);

    }

    @Override
    public void doThing() {
        Log.i("CALLBACKER", "made it");
    }
}

interface DoesThing {
    void doThing();
}

class Doer {

    @Inject
    public Doer() {}

    int doIt(String ignored) {
        return 5;
    }
}
```

## The Benefit
While the final code looks significantly longer than the first version of the code, it is also significantly more flexible. As someone who has been known to shout "YAGNE" as loudly as possible in public, I can understand why people may not want to jump through these extra hoops before it seems strictly necessary, but I have noticed that, as a rule, these refactors become necessary sooner than most people expect. Therefore, I usually jump directly to this design. The benefit is that there is always a place for new logic. If new collaborators are needed, and they are not temporally bound, they can be supplied to the factory. If many of these need to be made, there is an interface that unifies them. If this concrete implementation deprecates, it is already abstracted away by the interface. Testability is higher than might be expected due to the use of dagger. In short, changing to this pattern will make the code more maintainable. If you write code like this, people will greet you with a smile, the sun will shine more brightly and more often, and happiness will proliferate across the land.
