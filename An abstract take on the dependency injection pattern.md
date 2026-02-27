This article explores the design pattern called *dependency injection*. The pattern is explored from a relatively language-agnostic point of view in C#, since C#'s implementation of the pattern preserves the general principles of the pattern best.

At the end of the article, we discuss how dependency injection is done in Java- it's quite confusing if you haven't seen the C# way first!

# A typical dependency situation

Consider the following dependency situation, in which a class `Cls` depends both upon an interface `Intf` and on an implementation `Impl` of that interface. 

```java
public interface Intf { void HelperMethod(object input); }

public class Impl : Intf
{
    public void HelperMethod(object input) { /* implementation */ }
}

public class Cls
{
    public void Method(object input)
    {
        Intf intf = new Impl();
        intf.HelperMethod(input);
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

1. It allows us to change which implementation of `Intf` is used by `Cls` without modifying code in the body of `Cls`.
   - Having an easy way to swap one implementation out for another makes it easy to swap in a [mocked](https://blogs.perficient.com/2021/09/22/mocking-in-test-driven-development-tdd-with-javas-easymock/) implementation, which lends itself to test-driven development.
2. It delegates the responsibility of initializing `Cls`'s dependencies away from `Cls` itself.
   - When `Cls` doesn't engage in the rather individualistic practice of initializing its own dependencies, it is possible for a centralized manager to efficiently allocate dependency instances between  `Cls` and any other objects that require the same dependencies.

# A better dependency situation

To improve our dependency situation, we will pass the responsibility of creating dependencies (such as `Impl`) to a sort of central manager object. This object will be responsible for not only creating, but also managing, *all* of the dependencies of *all* objects in our program. It will do so by "injecting" the appropriate dependency instances into the constructors of the objects instances being initialized.

When this so-called *dependency injection* pattern is used, we will be in the following much improved dependency situation:

```
Cls --has--> Intf
Container --has--> Cls
Container --creates--> Impl
Impl --is--> Intf
```

Here, `Cls` depends only on `Intf` and not on `Impl`, as desired.

## Exposing a constructor

In order for the injection of the dependency `Intf` into `Cls` to be possible, we need to add a field of type `Intf` to `Cls`, and a constructor initializing it, so that there is a way for the manager object to pass an instance of  `Intf` type to `Cls`:

```c#
public class Cls
{	
	// New
	private Intf intf;
    public Cls(Intf intf)
    {
    	this.intf = intf;
    }
    
    // Same as before
    public void Method(object input)
    {
        Intf intf = new Impl();
        intf.HelperMethod(input);
    }
}
```

There's actually a nice shorthand for this in C# version 12 and later. We can define an implicit and primary[^1] constructor as follows:

```c#
// Changed: now using implicit constructor syntax
public class Cls(Intf intf)
{
    // Same as before
	public void Method(object input)
    {
        Intf intf = new Impl();
        intf.HelperMethod(input);
    }
}
```

[^1]: A constructor is *primary* if and only if every other constructor must invoke it at the start. In C#, all implicit constructors- like the above- are also primary.

## The central manager object

Once `Cls` has a constructor that can receive an instance of `Intf` type that's "injected" into it, we are ready to work with the central manager object responsible for performing these kinds of injections. In C#, the central manager object is a `ServiceProvider` instance. This name makes sense when you know that, in the C# .NET world, "dependencies" are often called "services"- a `ServiceProvider` "provides", or "injects", services (dependencies) into class instances by means of class constructors.

To get ahold of a `ServiceProvider`, we first have to specify the associations between the *dependency types* that appear in constructors and the *implementation types* that get injected into them. It *would* make sense if we told it that information by doing something like this:

```c#
using Microsoft.Extensions.DependencyInjection;
    
class Program
{
	static void Main(string[] args)
    {
        // Specify the associations between dependency types and implementation types.
    	IServiceCollection services = new ServiceCollection();
        services.AddAssociation<Intf, Impl>();
        services.AddAssociation<Cls, Cls>();
		
        // Build a manager that is aware of the associations.
        ServiceProvider manager = services.BuildServiceProvider();
        
        // Use the manager to get a Cls instance that has all dependencies injected into it.
        Cls cls = manager.GetRequiredService<Cls>();
        
        // Now we can call a method on the Cls instance that makes use of a dependency!
        cls.Method();
	}
}      
```

But... the method `IServiceCollection.AddAssociation<TIntf, Timpl>()` doesn't actually exist! I've just included it here for explanatory purposes. Instead of `IServiceCollection.AddAssociation<TIntf, TImpl>()`, we have to use one of the following:

* `IServiceCollection.AddTransient<TIntf, TImpl>()`
* `IServiceCollection.AddSingleton<TIntf, TImpl>()`
* `IServiceCollection.AddScoped<TIntf, TImpl>()`

So, actually valid code would be:

```c#
using Microsoft.Extensions.DependencyInjection;
    
class Program
{
	static void Main(string[] args)
    {
        // Specify the associations between dependency types and implementation types.
        // (Use AddTransient<,>() instead of the fake but more intuitive-sounding AddAssociation<,>()
        // from above.)
    	IServiceCollection services = new ServiceCollection();
        services.AddTransient<Intf, Impl>();
        services.AddTransient<Cls, Cls>();
		
        // Build a manager that is aware of the associations.
        ServiceProvider manager = services.BuildServiceProvider();
        
        // Use the manager to get a Cls instance that has all dependencies injected into it.
        Cls cls = manager.GetRequiredService<Cls>();
        
        // Now we can call a method on the Cls instance that makes use of a dependency!
        cls.Method();
	}
}      
```

If you're just learning about dependency injection for the first time, I wouldn't recommend worrying over these variants too much. Just use `AddTransient<,>()` for everything and pretend it says `AddAssociation<,>()`. 

# Miscellaneous notes

## C# .NET terminology

I've explained *some* of the terminology for dependency injection used in the C# .NET world. Here's the complete set of translations to the general "dependency" terminology I've favored in this article.

| C# .NET terminology  | General terminology                                     |
| -------------------- | ------------------------------------------------------- |
| service              | dependency                                              |
| service provider     | manager object                                          |
| service registration | the specification of dependencies to the manager object |
| service resolving    | the injection at runtime of a dependency                |

## Service lifetimes

Here's an explanation of the method variants from earlier: 

- `IServiceCollection.AddTransient<TIntf, TImpl>()`
- `IServiceCollection.AddSingleton<TIntf, TImpl>()`
- `IServiceCollection.AddScoped<TIntf, TImpl>()`

In addition to specifying a dependency-implementation association, each variant specifies a different "service lifetime" for the dependency in the association, that is to be enforced by the manager object. We can think of these methods as corresponding to some helpful pseudocode:

| Actual method in C#            | Pseudocode                                                   |
| ------------------------------ | ------------------------------------------------------------ |
| `AddTransient<TIntf, TImpl>()` | `AddAssociation<TIntf, TImpl>(serviceLifetime: "transient")` |
| `AddSingleton<TIntf, TImpl>()` | `AddAssociation<TIntf, TImpl>(serviceLifetime: "singleton")` |
| `AddScoped<TIntf, TImpl>()`    | `AddAssociation<TIntf, TImpl>(serviceLifetime: "scoped")`    |

A dependency with *transient* service lifetime is created anew every time it is requested from the managing container; each instance of a transient dependency is used independently of all other instances.

A dependency with *singleton* service lifetime is created once, the first time it is requested from the managing container, and shared among all objects that use it.

Dependencies with *scoped* service lifetime a little more complicated, so we don't cover them in this introduction.

## Inversion of control

Control is in some sense inverted in dependency injection. Instead of `Cls` controlling it's *own* dependencies, the manager object does. Manager objects are sometimes called *inversion of control (IoC) containers* for this reason.

Note: while dependency injection is an example of inversion of control, not all inversion of control is dependency injection. [This article](https://martinfowler.com/bliki/InversionOfControl.html) by Martin Fowler details other examples of inversion of control.

## Dependency injection without constructors

Using class constructors is to pass an initialized dependency to a class is called *constructor injection*. In my opinion, it is the most intuitive way to do dependency injection, and superior to all the alternatives... but alternatives do exist. There's *setter injection*, which uses setter methods to pass dependencies, and *field injection*, which [abuses reflection techniques](https://stackoverflow.com/a/53744544) to modify `private` fields from outside the class in which they are declared.

## Dependency injection in other languages

### Java and the Spring Framework

Java does not have native support for dependency injection like C# does. Nevertheless, large enterprise projects written in Java that use dependency injection exist and support many important applications. Most of these projects use the Spring Framework for Java, a third party framework that provides dependency injection capability.

Associations between dependency types and implementation types are specified a little differently in the modern version of the Spring Framework. Spring makes use of Java annotations to specify dependency configurations, rather than explicit method calls like was the case like in C#. 

In Java Spring Framework, the analog to the C# .NET `Main()` method filled with `AddTransient<TIntf, TImpl>()` calls is a class annotated with `@Configuration`, like this:

```java
@Configuration
public class AppConfig
{
    // In Java Spring, "bean" means "dependency". Thus, anything annotated with @Bean is interpreted
    // by the manager object to be a dependency.
	@Bean
	public Intf1 intf1Builder()
    {
		return new Impl1();
	}

	@Bean
	public Intf2 intf2Builder()
    {
		return new Impl2();
	}
}
```

Although, the more popular approach in Java Spring is to specify each dependency-implementation association locally, *per-dependency* (i.e. per interface), like this:

```java
// "Component" also means "dependency" in Java Spring.
@Component
public class Impl extends Intf { }
```

Essentially, any class `Impl` extending an interface `Intf` that is annotated with `@Component` is assumed to associate the dependency type of `Intf` with the implementation type of `Impl`.

Lastly, in the Spring Framework, one has to annotate the constructor with `@Autowired` if the manager object is to actually go ahead and perform constructor injection. 

```java
public class Cls
{	
	private Intf intf;
    
    @Autowired
    public Cls(Intf intf)
    {
    	this.intf = intf;
    }
    
    public void Method(object input)
    {
        Intf intf = new Impl();
        intf.HelperMethod(input);
    }
}
```

The name `@Autowired` was likely chosen for this purpose because sometimes, instead of talking of "injecting" dependencies, people speak of "wiring up" dependencies. Both phrases mean the exact same thing; it's a shame, in my opinion, that something more intuitive like `@Injected` wasn't used instead.

### Angular

Angular, the web application frontend framework, uses an annotation-based approach to dependency injection similar to that of Java Spring Framework that is arguably much more intuitive.
