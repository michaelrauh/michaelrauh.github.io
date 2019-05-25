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
class Callbacker {

    private Callback callback;
    private Doer doer;

    public Callbacker() {
      doer = new Doer();
    }

    void bind(Callback callback) {
        this.callback = callback;
    }

    void doCallbacker() {
        doer.doIt("before")
        callback.doThing();
        doer.doIt("after")
    }
}

class Callback {

    private Callbacker callbacker;

    public Callback() {
        callbacker = new Callbacker();

        callbacker.bind(this);
    }

    void doThing() {
        Log.i("CALLBACK", "made it");
    }
}

class Doer {
  void doIt(String message) {
      Log.i("DOER", "running code: " + message);
  }
}
```
This code demonstrates several antipatterns in making callbacks. We can make several enhancements to the code without completely overhauling it.

## Enhancements


The first issue with the example is that there is no dependency injection. This is better:

```
class Callbacker {

    private Callback callback;
    private Doer doer;

    @Inject
    public Callbacker() {}

    void bind(Callback callback) {
        this.callback = callback;
        this.doer = doer;
    }

    void doCallbacker() {
      doer.doIt("before")
      callback.doThing();
      doer.doIt("after")
    }
}

class Callback {

    private Callbacker callbacker;

    @Inject
    public Callback(Callbacker callbacker) {
        this.callbacker = callbacker;

        callbacker.bind(this);
    }

    void doThing() {
        Log.i("CALLBACK", "made it");
    }
}

class Doer {

    @Inject
    public Doer() {}

    void doIt(String message) {
        Log.i("DOER", "running code: " + message);
    }
}
```
The next demerit of this code is the fact that `bind` must be called before the callback occurs, or else an exception will occur. This can be fixed by binding inside the constructor, but using dagger for this introduces a circular dependency. The fix here is to not use dagger for that particular field, but to use it for any other fields. I think you see where I am going. I am going to have to say the F-word.

```
class Callbacker {

    private Callback callback;
    private Doer doer;

    public Callbacker(Callback callback, Doer doer) {
        this.callback = callback;
        this.doer = doer;
    }

    void doCallbacker() {
      doer.doIt("before")
      callback.doThing();
      doer.doIt("after")
    }
}

class CallbackerFactory {

    private Doer doer;

    @Inject
    public CallbackerFactory(Doer doer) {
        this.doer = doer;
    }

    public Callbacker create(Callback callback) {
        return new Callbacker(callback, doer);
    }
}

class Callback {

    private Callbacker callbacker;

    @Inject
    public Callback(CallbackerFactory callbackerFactory) {
        this.callbacker = callbackerFactory.create(this);

    }

    void doThing() {
        Log.i("CALLBACK", "made it");
    }
}

class Doer {

    @Inject
    public Doer() {}

    void doIt(String message) {
        Log.i("DOER", "running code: " + message);
    }
}
```

That is, factory. Making a factory allows for partial binding. Since the callback is temporally bound (temporal binding is evil but sometimes unavoidable), it can't be supplied through dagger without calling bind in the dagger module. That doesn't mean that the entire component must be carved out, though. Doer can still be supplied and bound at build time, then mixed in to the callback at create time. Now the biggest remaining issue is that, when we look inside of `Callback`, we have no idea that `doThing` is a callback method. We see a method that seems to just be sitting there, unused. To make it more clear that this method is called as a result of some event that is not under its control, it is better to put this method behind an interface. Now a casual glance shows the override annotation in the class. Another advantage to this is that this callback can then be replaced without too much trouble. Additionally, if there are many callbacks with the same pattern, a container could be made to hold a collection of the interface types. Most importantly, it hides any information that is not necessary to the callback. This ends up looking like below:

```
class Callbacker {

    private DoesThing doesThing;
    private Doer doer;

    public Callbacker(DoesThing doesThing, Doer doer) {
        this.doesThing= doesThing;
        this.doer = doer;
    }

    void doCallbacker() {
        doer.doIt("before")
        doesThing.doThing();
        doer.doIt("after")
    }
}

class CallbackerFactory {

    private Doer doer;

    @Inject
    public CallbackerFactory(Doer doer) {
        this.doer = doer;
    }

    public Callbacker create(DoesThing doesThing) {
        return new Callbacker(doesThing, doer);
    }
}

class Callback implements DoesThing {

    private final Callbacker callbacker;

    @Inject
    public Callback(CallbackerFactory callbackerFactory) {
        this.callbacker = callbackerFactory.create(this);
    }

    @Override
    public void doThing() {
        Log.i("CALLBACK", "made it");
    }
}

interface DoesThing {
    void doThing();
}

class Doer {

    @Inject
    public Doer() {}

    void doIt(String message) {
        Log.i("DOER", "running code: " + message);
    }
}
```

## The Benefit
While the final code looks significantly longer than the first version of the code, it is also significantly more flexible. As someone who has been known to shout "YAGNE" as loudly as possible in public, I can understand why people may not want to jump through these extra hoops before it seems strictly necessary, but I have noticed that, as a rule, these refactors become necessary sooner than most people expect. Therefore, I usually jump directly to this design. The benefit is that there is always a place for new logic. If new collaborators are needed, and they are not temporally bound, they can be supplied to the factory. If many of these need to be made, there is an interface that unifies them. If this concrete implementation deprecates, it is already abstracted away by the interface. Testability is higher than might be expected due to the use of dagger. In short, changing to this pattern will make the code more maintainable. If you write code like this, people will greet you with a smile, the sun will shine more brightly and more often, and happiness will proliferate across the land.
