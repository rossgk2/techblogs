This article will take a relatively abstract look at the design pattern called *dependency injection* (or *inversion of control*). I feel that most articles about dependency injection get too bogged down in the particulars of whatever example is being used to demonstrate the structure. In this article, we'll present pure abstraction. 

Well, maybe not *pure* abstraction- we do have to pick a particular programming language, after all! We will use Java in this article. If you don't know Java, don't worry too much. We'll stick to "basic" Java- nothing esoteric.

# A typical dependency situation

Consider the following dependency situation, in which a class `Cls` depends both upon an interface `Intf` and on an implementation `Impl` of that interface. 

```java
public interface Intf { void helperMethod(Object args); }

public class Impl extends Intf
{
    @Override
    public void helperMethod(Object args) { /* implementation */ }
}

public class Cls
{
    public void method(Object args)
    {
        Intf intf = new Impl();
        intf.helperMethod(args);
    }
}
```

`Cls` *depends* on `Impl` because it requires knowledge of the `Impl` type in order to execute `new Impl()`. To restate, our current dependency situation is:

```
Cls --creates--> Impl
Cls --has--> Intf
Impl --is--> Intf
```

We want to be in a dependency situation in which `Cls` depends only on `Intf` and not on `Impl`. I.e., we want to *decouple* the implementation of `Cls` from any particular implementation of `Intf`. As described in some [Microsoft documentation](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection), this decoupling is desired because:

1. It allows us to change which implementation of `Intf` is used by `Cls` without modifying code* in the body of `Cls`.
   - Having an easy way to swap one implementation out for another makes it easy to swap in a [mocked](https://blogs.perficient.com/2021/09/22/mocking-in-test-driven-development-tdd-with-javas-easymock/) implementation, which lends itself to test-driven development.
2. It removes the need for manual configuration of `Impl`'s dependencies.

\* If you're using Spring Framework for Java, then you change the injected implementation by changing an annotation within the body of `Cls`. I don't count this as changing "actual code" in the body of `Cls`! Admittedly, it *would* be better if changing the interface implementation didn't touch *anything* inside `Cls`, so that the configuration code (the code specifying which interface is to be injected) is completely separate from the implementation code. C# .NET's way of doing dependency injection is better about sticking to this rule.

# A better dependency situation

To improve our dependency situation, we will pass the responsibility of creating `Impl` to some `Container` class that manages `Cls`. When commanded to do so, `Container` will "inject" a `new`ly created `Impl` instance into `Cls`. One way to perform this "injection", *constructor injection*, is to pass an `Impl` instance to a constructor of `Cls` that accepts an `Impl`. (Of course, `Cls` must have a public with-arguments constructor for constructor injection to be possible). There are other forms of dependency injection, such as *setter injection* and *field injection*. (In Java, field injection is predicated on [abusing reflection techniques](https://stackoverflow.com/a/53744544) to modify `private` fields).

 This setup is called *dependency* *injection.* Implementing dependency injection places us in the following much improved dependency situation:

```
Cls --has--> Intf
Container --has--> Cls
Container --has--> Intf
Container --creates--> Impl
Impl --is--> `Intf
```

Now, `Cls` depends only on `Intf` and not on `Impl`, as desired.

In addition to solving the decoupling problem, dependency injection comes with the extravagant benefit of allowing for the "injection" of a single interface implementation into different classes that depend on the same interface type, when doing so makes sense. In other words, our new dependency situation also allows for the sharing of one `Intf` instance between class instances.

## Summary

To summarize, here are the two main benefits of dependency injection:

1. Dependency injection decouples the implementations of classes from the implementations of those classes' dependencies. Decoupling of interfaces from implementations is desirable because...

   1.1. It allows us to change which implementation of an interface is used by a dependent class without modifying code in the body of the class.

   - Having an easy way to swap one implementation out for another makes it easy to swap in a [mocked](https://blogs.perficient.com/2021/09/22/mocking-in-test-driven-development-tdd-with-javas-easymock/) implementation, which lends itself to test-driven development.

   1.2. It removes the need for manual configuration of an injected implementation's dependencies.

2. Dependency injection allows us to share a single interface implementation instance between multiple classes that depend on said interface, when applicable.

## Inversion of control

Since, in dependency injection, `Container`, rather than `Cls`, controls what and when `Cls`'s dependencies are injected, control has in some sense been inverted. Dependency injection is thus an example of *inversion of control.* For this reason, a class such as `Container` is often referred to as the *inversion of control container*, or *IoC container*.

Note that while dependency injection is an example of inversion of control, not all inversion of control is dependency injection. [This article](https://martinfowler.com/bliki/InversionOfControl.html) by Martin Fowler details other examples of inversion of control.

# Code

Here's code that implements the dependency injection pattern. (The following is basically an abstract extrapolation of code given in [Martin Fowler's article](https://martinfowler.com/articles/injection.html) on dependency injection).

```java
public interface Intf { void helperMethod(Object args); }

public class Impl extends Intf
{
    private Object args;
    public Impl(Object args) { this.args = args; }

    @Override
    public void helperMethod() { /* implementation that uses this.args */ }
}

public class Cls
{
    private Intf intf;
    public Cls(Intf intf) { this.intf = intf; }

    public void method() { intf.helperMethod(); }
}

public class Container{ /* implementation will be kind of complicated */ }
   
public class Main
{
    /* A config file typically performs performs the task of this method. */
    private Container configureContainer()
    {
       Object args = ... // get the arguments that should be passed to the Impl constructor
       Container cntr = new Container();
       
       /* The below line tells cntr all the information it needs to execute the statement
       "Intf intf = Impl(args)". */
       cntr.registerComponentImplementation(Intf.class, Impl.class, args);
       
       /* This next line tells cntr all the information it needs to execute the statement
       "Cls cls = new Cls(intf)". */
       cntr.registerComponentImplementation(Cls.class);
       return cntr;
    }
   
    public static void main(String[] _args)
    {
       /* This is how we call cls.method(). */
       Container cntr = configureContainer();
       Cls cls = (Cls) cntr.getComponentInstance(Cls.class);
       cls.method(); // This executes the same task as "cls.method(args)" did in the original situation.
    }
}
```

## Extra: did we give `Container` unnecessary information?

One particular about the lines involving `cntr.registerComponentImplementation()` wasn't immediately clear to me, and might be confusing to you, too. My question was: is it necessary to pass `Intf.class` to the first call of `registerComponentImplementation()`? It seems that there should exist an implementation of `Container` such that if we execute the following, `cntr` does what we would expect behind the scenes.

```java
cntr.registerComponentImplementation(Impl.class, args);
cntr.registerComponentImplementation(Cls.class);
```

That is, it seems that `cntr` would have enough information to do `new Cls(new Impl(args))`. This is because `cntr` has a `Cls`, and `Cls` knows that `Impl` is an `Intf`. And, for the sake of argument, even if we assume that `cntr` somehow *didn't* know about this through `Cls`, the JVM itself knows that `Impl` is an `Intf`- after all, executing `new Cls(new Impl(args))` doesn't require that we type cast `new Impl(args)` to `Intf`!

**Answer:** upon investigation, I found that it is just convention to pass more information than is strictly necessary in certain dependency injection frameworks.
