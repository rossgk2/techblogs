# Dependency injection in C# .NET

I've decided to write a tutorial on how to accomplish dependency injection in C# .NET, because the Microsoft documentation of dependency injection libraries is unfortunately way too sparse, and the Microsoft dependency injection [tutorial](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-usage) is a little convoluted and complicated.

Fortunately, C# .NET's implementation of dependency injection is pretty straightforward. In my opinion, it's way more straightforward than the implementation provided by Java's Spring Framework. If you understand the basics of the dependency injection concept but haven't yet tried it out in practice, C# .NET could be your best bet.

## Prerequisite: dependence on interfaces

Traditionally, a class instance might depend on a helper method from a helper class like this:

```c#
public class Controller
{
	private Dependency dependency;

	public Controller()
	{
		dependency = new Dependency();
	}

    /* A Controller instance controller will use the helper method
        whenever controller.controllerMethod() is called. */
	public int controllerMethod()
	{
		int temp = dependency.helperMethod();
		return someMethod(temp);
	}
}

public class Dependency
{
	public int helperMethod()
    {
        /* Implementation not shown. */
    }
}
```

In the above, `Controller` depends on a concrete type, `Dependency`. It's better practice, though, to have  `Controller` depend on an *interface*. 

So, a slight improvement to the above is achieved by creating an `IDependency` interface, having  `Dependency` implement `IDependency`, and changing the type of the `dependency` member from `Dependency` to `IDependency`:

```c#
public class Controller
{
    // Changed: the type of `dependency` is now `IDependency`
	private IDependency dependency;

    // Same as before
	public Controller()
	{
		dependency = new Dependency();
	}

    // Same as before
	public int controllerMethod()
	{
		int temp = dependency.helperMethod();
		return someMethod(temp);
	}
}

// New
public interface IDependency
{
    int helperMethod();
}

// Changed: now implements the IDependency interface
public class Dependency : IDependency
{
	public int helperMethod()
    {
        /* Implementation not shown. */
    }
}
```

Take note of the code in the constructor of `Controller`. Since it associates the interface-typed `dependency` member with a particular implementation of that interface, it is called *dependency configuration code*.

## Mediator objects and dependency injection

In the above approach, dependency configuration is localized to each class-with-dependencies.

Another arguably better way of doing things is maintaining all the dependency configuration code in one place- having a *global* dependency configuration, in other words.

How can we accomplish this?

Well, we could have some sort of mediator object that's aware of the global dependency configuration:

```c#
using Microsoft.Extensions.DependencyInjection;

class Program
{
    static void Main(string[] args)
    {
        /* Global dependency configuration. */
        /* Don't worry about what "AddTransient" means. For the purposes of this tutorial, you can read "AddTransient<TBase, TImpl>" as something like "AddAssociation<TBase, TImpl>"; it simply associates a base type (often an interface) to a derived type. */
		var services = new ServiceCollection();
        services.AddTransient<IDependency, Dependency>();
        services.AddTransient<Controller, Controller>();
        
        /* This mediator object is aware of the global dependency configuration. */
        ServiceProvider mediator = services.BuildServiceProvider();
    }
}
```

Since the mediator object knows the global dependency configuration, then, whenever, whenever we request a controller instance from the mediator, it could "inject" the appropriate dependency instances into the constructors of the controller instances being initialized.

Of course, this would require us to change the constructor of `Controller` so that it can indeed be "injected" into:

```c#
public class Controller
{
    // Same as before
	private IDependency dependency;

    // Changed: used to accept no inputs, now accepts an IDependency input so that a managing container can "inject" an IDependency in
 	public Controller(IDependency dependency)
	{
		this.dependency = dependency;
	}

    // Same as before
	public int controllerMethod()
	{
		int temp = dependency.helperMethod();
		return someMethod(temp);
	}
}
```

There's actually a nice shorthand for this in C# version 12 and later. We can define an implicit and primary[^1] constructor as follows:

```c#
// Changed: now using implicit constructor syntax
public class Controller(IDependency dependency)
{
    // Same as before
	public int controllerMethod()
	{
		int temp = dependency.helperMethod();
		return someMethod(temp);
	}
}
```

[^1]: A constructor is *primary* if and only if every other constructor must invoke it at the start. In C#, all implicit constructors- like the above- are also primary.

Now, when we need a `Controller` instance, we can get one by using the mediator.

```c#
using Microsoft.Extensions.DependencyInjection;
    
class Program
{
	static void Main(string[] args)
    {
        /* This global configuration code is the same as before. */
    	var services = new ServiceCollection();
        services.AddTransient<IDependency, Dependency>();
        services.AddTransient<Controller, Controller>();
        ServiceProvider mediator = services.BuildServiceProvider();
            
        /* Get a Controller instance that has a Dependency instance injected into the IDependency input of its constructor. */
        Controller controller = mediator.GetRequiredService<Controller>();
	}
}       
```

# .NET terminology

.NET uses slightly different terminology for dependency injection concepts than what's used in this article. Here's how the terms in .NET map to the terms used in this article.

| Microsoft phrase     | Phrase used in this article                           |
| -------------------- | ----------------------------------------------------- |
| service              | dependency                                            |
| service provider     | mediator object                                       |
| service registration | the storing of dependencies in the managing container |
| service resolving    | the injection at runtime of a dependency              |

# References

I referenced the following two articles in developing my understanding of `IServiceCollection`: [(1)](https://www.stevejgordon.co.uk/aspnet-core-dependency-injection-what-is-the-iserviceprovider-and-how-is-it-built), [(2)](https://www.stevejgordon.co.uk/aspnet-core-dependency-injection-what-is-the-iservicecollection).
