I've decided to write a tutorial on how to accomplish dependency injection in C# .NET, because the Microsoft documentation of dependency injection libraries is unfortunately way too sparse, and the Microsoft dependency injection [tutorial](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-usage) is a little convoluted and complicated.

Fortunately, C# .NET's implementation of dependency injection is pretty straightforward. In my opinion, it's way more straightforward than the implementation provided by Java's Spring Framework. If you understand the basics of the dependency injection concept but haven't yet tried it out in practice, C# .NET could be your best bet.

# Dependency injection recap

Here's a quick recap on what dependency injection entails. If you want more detail, [this article](https://blogs.perficient.com/2021/09/22/an-abstract-take-on-the-dependency-injection-pattern/) I wrote may be helpful.

In general, whether dependency injection is in play or not, classes may specify types- called *dependencies*- that they have *has-a* relationships with. 

In dependency injection, instances of classes are not responsible for creating instances of their dependencies. Instead, a managing container maintains *has-a* relationships with instances of the classes, and the user specifies to the container which implementations of the dependencies they want to use by calling one of the container's methods, or by writing so-called "configuration code" that is interpreted by the container. At runtime, the container "injects" these implementations into the class instances.

Why use dependency injection? The main point is to *separate interface from implementation*. Why is this important? I suggest you read the linked article above for more details.

# .NET dependency injection terminology

The first thing to be aware of when learning dependency injection in C# .NET is that Microsoft uses some alternative terminology when discussing dependency injection concepts. If you want to be able to understand the Microsoft documentation, you need to be aware of this terminology. So, here's some vocabulary:

| Microsoft phrase     | Meaning                                               |
| -------------------- | ----------------------------------------------------- |
| service              | dependency                                            |
| service registration | the storing of dependencies in the managing container |
| service resolving    | the injection at runtime of a dependency              |

# `ServiceDescriptor`

The [`ServiceDescriptor`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor?view=dotnet-plat-ext-6.0) class is what represents a service (recall that "service" means "dependency"). The most down-to-earth constructor of `ServiceDescriptor` is as follows:

```
public ServiceDescriptor (Type serviceType, Type implementationType, ServiceLifetime lifetime)
```

So, we see that in C# .NET, a service essentially wraps the type of the dependency, the type of the preferred implementation for said dependency, and the "lifetime" of the dependency.

## `ServiceLifetime`

In my opinion, "lifetime" should really be called "instantiation multiplicity", since the value of `lifetime` in the above constructor determines whether or not the management container is to create multiple instances of the dependency, and, if so, how to do so.

Specifically, [`ServiceLifetime`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicelifetime?view=dotnet-plat-ext-6.0) is an `enum` that can take on the value `Singleton`, `Transient`, or `Scoped`. 

- `Singleton` indicates that the management container (which we have not seen yet) will ensure that only one instance of the service will be created throughout the program lifetime. All class instances which depend on the service will share the same service.
- `Transient` indicates that the management container will ensure that a new instance of the dependency will be created whenever a different class instance needs it.
- The meaning of `Scoped` is a little complicated for a first pass at dependency injection in C# .NET. If you want to learn about it, read [this](https://stackoverflow.com/questions/38138100/addtransient-addscoped-and-addsingleton-services-differences).

## `ServiceDescriptor` properties

You already saw the `ServiceDescriptor` constructor, which is what's most important in regards to understanding `ServiceDescriptor`. For a bit more detail, here are the public properties that are wrapped by the `ServiceDescriptor`:

```c#
public Func<IServiceProvider,object>? ImplementationFactory { get; } // a factory method that stores instructions on how to build an instance of the implementation type
public object? ImplementationInstance { get; }
public Type? ImplementationType { get; } // type of the wrapped instance, ImplementationInstance
public ServiceLifetime Lifetime { get; }
public Type ServiceType { get; } // type of the wrapped interface
```

Some of the above may be confusing, so here are some clarifying notes:

- `Func<T1, T2>` represents a function that takes an argument of type `T1` as input and returns a type `T2` instance as output. Thus, the `ImplementationFactory` property is a function that takes an `IServiceProvider` as input and returns an instance of the implementation as output. `ImplementationFactory` can be thought of as wrapping instructions for how to create an instance of the implementation instance.
- For any type `T`, the expression `T?` is shorthand for `Nullable<T>`, which represents a *nullable* version of the type `T`. A type is called *nullable* if compiler errors are *not* thrown when a `null` value of said type is attempted to be used. For more context on `Nullable<T>` is, see the below appendix.

## Registering services (`ServiceDescriptor`s) with `IServiceCollection`

So far, we know how to represent services (dependencies) as `ServiceDescriptor`s. We'll now learn how to create a managing container and how to register our services with said container.

An instance of type [`IServiceCollection`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection?view=dotnet-plat-ext-6.0) is what will represent our managing container. From its interface definition, we can see that `IServiceCollection` is an interface to a collection of `ServiceDescriptor`s. (So, `IServiceCollection` is interpreted as "`I{ServiceCollection}`", which means "interface to a collection of services", not "`{IService}Collection`", which would mean "collection of interfaces to services"!).

```c#
using Microsoft.Extensions.DependencyInjection;
using System.Collections.Generic;
public interface IServiceCollection : ICollection<ServiceDescriptor>, IEnumerable<ServiceDescriptor>, IList<ServiceDescriptor> { }
```

Microsoft provides an implementation of `IServiceCollection` for us- the [`ServiceCollection`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollection?view=dotnet-plat-ext-6.0) class from the `Microsoft.Extensions.DependencyInjection` namespace- so we don't have to take care of the implementation ourselves.

### Service registration via extension methods to `IServiceCollection`

In order to store services in an `IServiceCollection`, we need to enable access to some *extension methods to* `IServiceCollection`. 

(An extension method is an instance method of a class that is added to the class *after* the class is defined. Confusingly, you *can* add extension methods, which are non-abstract methods, to an interface. To learn more about extension methods, see the below appendix).

To obtain access to the extension methods we need, just include a `using Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions` statement at the top of the file.

Some important extension methods (with the parameter for the extended class, `IServiceCollection`, omitted) added by `ServiceCollectionServiceExtensions` are:

```c#
AddSingleton(Type serviceType, Type implementationType);
AddSingleton(Type serviceType); // is the above with implementationType = serviceType
```

Being extension methods to `IServiceCollection`, these methods are invoked on an instance of type `IServiceCollection` in the same way that usual instance methods are. For example, if `services` has type `IServiceCollection`, then we would call the above methods by writing

```c#
services.AddSingleton(serviceType, implementationType);
services.AddSingleton(serviceType);
```

You can probably surmise that these two methods add a `ServiceDescriptor` of lifetime `Singleton` that has the specified `serviceType` and `implementationType` to the `IServiceCollection`. 

Since the arguments of the above methods are types, one could say it is more proper to specify those arguments as formal type parameters. There are versions of the above methods that cater to this preference:

```c#
AddSingleton<TService,TImplementation>()
AddSingleton<TService>(); // is the above with TImplementation = TService
AddSingleton<TService,TImplementation>(Func<IServiceProvider,TImplementation> implementationFactory)
```

And, of course, for every method whose name is `AddSingleton`, there will be corresponding methods with names of `AddTransient` and `AddScoped` that perform the same task for services of `Transient` and `Scoped` lifetimes, respectively.

Though, these two versions of `AddSingleton`, which specify the instance that is to be wrapped by the singleton service, don't have `AddTransient` or `AddScoped` counterparts, because it wouldn't make sense to specify only a single instance to `AddTransient` or `AddScoped`:

```c#
AddSingleton(Type serviceType, object implementationInstance);
AddSingleton<TService>(TService instance);
```

## Resolving services at runtime with `IServiceProvider`

At this point, we know how to we know how to represent services (dependencies) as `ServiceDescriptor`s, how to create how to create a managing container, and how to register our services with the container. The last item we need to address is that of configuring the resolving of services at runtime (i.e. configuring the injection of dependencies at runtime).

Let's suppose that `services` is an `IServiceCollection` (i.e. a managing container) that contains some `ServiceDescriptor`s (i.e. "services", or dependencies).

To grab services from a managing container named `services` at runtime, we will first obtain an instance of type `IServiceProvider` from the managing container* by storing the return value of `services.BuildServiceProvider()` . Then, we grab a particular service by using the single abstract method specified in the `IServiceProvider` interface:

```c#
public object? GetService(Type serviceType)
```

\* `services.BuildServiceProvider` returns an instance of `Microsoft.Extensions.DependencyInjection.ServiceProvider`, which implements `IServiceProvider`. 

# Somewhat random: creating `IServiceCollection`s by using `IServiceProvider`s

This section is pretty optional.

If you already have one `IServiceCollection` instance and corresponding `IServiceProvider`, and you want to create another `IServiceCollection` instance by making use of dependencies stored in the first `IServiceCollection` instance, you can use these extension methods to `IServiceCollection`:

```c#
AddSingleton(Type serviceType, Func<IServiceProvider, object> factory);
AddSingleton<TService,TImplementation>(Func<IServiceProvider,TImplementation> implementationFactory);
AddSingleton<TService>(Func<IServiceProvider, TService> implementationFactory); // is the above with TImplementation = TService
```

# Appendix

This appendix documents some less commonly known language features of the C# language.

## `?` and nullable reference types

A *non-nullable* type is a type for which compiler errors are thrown when a variable of that type with `null` value is attempted to be used . Contrastingly, a *nullable* type is a type for which compiler errors are not thrown in said situation. You can still get runtime errors with nullable types, of course! The whole point of non-nullable types is to avoid runtime errors by catching them at compilation.

According to the [Microsoft documentation](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references#:~:text=Prior to C%23 8.0%2C all,NullReferenceException), all reference types were nullable prior to C# 8.0. Nowadays (i.e. after C# 8.0), all reference types are non-nullable by default.

You can still use nullable types if you really want, though. For any type `T`, the type`Nullable<T>` is nullable. `T?` is shorthand for `Nullable<T>`.

## Extension methods

In C#, it is possible to define instance methods outside of the corresponding class definition. Methods defined in this way are called *extension methods*.

Extension methods must be defined in a `static` class, and must use the `this` keyword in the following way:

```c#
public class Cls { ... }

public static class Extension
{
     public static int extensionMethod1(this Cls cls)
     { int someValue = 0; return someValue; }       
     
     public static int extensionMethod2(this Cls cls, int arg)
     { int someValue = 0; return someValue; } 
}
```

Extension methods are called in the same way as regular instance methods: to call the above defined extension methods on an instance `cls` of `Cls`, you would write `cls.extensionMethod()` or `cls.extensionMethod2(arg)`, respectively.

### Extension methods to interfaces

Somewhat confusingly, it is possible to define extension methods- in exactly the same way as above- for interfaces. To me this possibility runs contradictory to the intent of "interface"- interfaces are not supposed to be associated with actual implementations of methods. But you *can* do it. It is also in fact impossible to add something like an "abstract extension method" to an interface. The C# standard library unfortunately makes much use of implementing interfaces via extension methods. Oh well.

# References

I referenced the following two articles in developing my understanding of `IServiceCollection`: [(1)](https://www.stevejgordon.co.uk/aspnet-core-dependency-injection-what-is-the-iserviceprovider-and-how-is-it-built), [(2)](https://www.stevejgordon.co.uk/aspnet-core-dependency-injection-what-is-the-iservicecollection).
