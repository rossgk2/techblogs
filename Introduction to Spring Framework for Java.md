# Introduction to Spring Framework for Java

This article walks through the basics of using Spring Framework for Java. 

From a very, very high level point of view, Spring Framework infers a program's runtime behavior from labels that the programmer attaches to pieces of code. There are many different groupings of labels, and each grouping of labels provides an interface to the configuration of some behind-the-scenes process. 

Since a simple-looking Spring Framework program with just a few labels can have quite a lot going on behind the scenes, learning Spring Framework can seem a little overwhelming to the beginner. Learning is made even more difficult by the fact that most online resources documenting Spring Framework haphazardly walk through different types of labels instead of building a fundamental ground-up understanding. 

This article intends to fill this gap and provide a ground up understanding. We will start with vanilla Java, and then, one programming design pattern at a time, we will augment on an understanding of how that design pattern is configured in Spring Framework with labels.

Before you begin reading this article in earnest...

- You should have a firm grasp of how object oriented programming is achieved in Java. So, you should be familiar with concepts such as: "pass reference by value" and "pass by reference", classes, constructors, fields, access modifiers (`public` and `private`), methods, static methods, class instances/objects, getter and setter methods, inheritance, method overloading, runtime polymorphism, interfaces and abstract methods, etc.

- You should know about the [basics of Java annotations](https://docs.oracle.com/javase/tutorial/java/annotations/basics.html). The "labels" spoken of above are really annotations.
- You should be aware of [Macro Behler's excellent introduction to Spring Framework](https://www.marcobehler.com/guides/spring-framework). I've mentioned that most online resources on Spring Framework are disorganized and disappointing; his article is one of the few exceptions. It's always good to have multiple readings to pull from when learning a topic, and so I encourage you to read this article as a supplement to this.

# **Dependency injection**

Above, our "very, very high level" point of view was that Spring Framework infers a program's runtime behavior from Java annotations. This is an accurate surface-level description of Spring Framework, but it isn't a good characterization from which to grow a fundamental understanding.

We will begin our understanding of Spring Framework with a different characterization. At its core, Spring Framework is a tool for implementing the design pattern called *dependency injection*.

In [*dependency injection*](https://blogs.perficient.com/2021/09/22/an-abstract-take-on-the-dependency-injection-pattern/), the object instances on which a Java class `Cls` depends are "injected" into an instance `obj` of `Cls` by a container that has a reference to `obj`. Since the container, rather than `obj`, controls when `obj`'s dependencies are injected, it is often referred to as an *inversion of control container*. Dependency injection is also sometimes called "inversion of control" for this reason.

What's the point of dependency injection? Well, one of the advantages is that *i**t allows us to avoid instantiating unnecessary copies of a dependency.*

Suppose that multiple classes require a reference to an object that represents a connection to a particular database. Since this reference is a dependency, we can easily share a single database connection among all of the class instances by making use of the dependency injection technique described above. This is much better than wastefully giving each class instance its own database connection.

# **Spring beans**

When using Spring Framework, we will spend most of our time dealing with and thinking about "Spring beans," which are the dependencies that are managed by the Spring IoC container. 

To be more specific, a Spring bean is a not-necessarily-`Serializable` object that...

- is created at runtime by the *Spring IoC container* (IoC stands for inversion of control)
- has references to other objects or beans ("dependencies") injected into it at runtime by the Spring IoC container
- is otherwise controlled at runtime by the Spring IoC container.

Spring beans can be configured via XML files or by using Java annotations within Java classes. Using annotations is the modern approach. This article will use annotations only- no XML code.

Note that a Spring bean is different than a JavaBean. JavaBeans are part of the core Java language, while Spring beans are obviously not. (Specifically, a JavaBean is a `Serializable` class with a public no argument constructor that has all fields `private`. JavaBeans are also not managed by the Spring IoC container).

# **Configuring Spring beans**

In the above, we said that Spring beans are created at runtime by the Spring IoC container. But how does the Spring IoC container know what sort of beans to create? Well, you, the programmer, must write "configuration code" that specifies which Java classes should be instantiated as Spring beans at runtime.

There are two main ways to do this: either use the `@Bean` annotation or the `@Component` annotation. (You can also use an annotation derived from `@Component`).

## First configuration method: `@Bean`ed methods that return instances

To specify that `Cls` should be instantiated as a bean at runtime, annotate a method that returns an instance of `Cls` with `@Bean`, and place that annotated method within a class that is itself annotated with `@Configuration`:

```java
@Configuration
public class Config
{
     @Bean
     public Cls createABeanWrappingAClsInstance(Object args) { return new Cls(args); }
}
```

## Second configuration method: use `@Component` and `@ComponentScan`

Another way to specify that a class `Cls` should be instantiated as a bean at runtime is to annotate `Cls`'s class declaration (i.e. `public class Cls`) with `@Component`, while also annotating some other class, say `Config`, that is "at or above below" the level of `Cls` in the directory hierarchy with `@Configuration` and `@ComponentScan`:

```java
@Configuration
@ComponentScan
/* Config must be at or above Cls in the directory hierarchy. */
public class Config { }

@Component
public class Cls { ... }
```

(When you create later create an `ApplicationContext` in the `main()` method of your application, you will have to pass `ApplicationContext.class` to the `ApplicationContext` constructor. But, if you are using Spring Boot, the class containing the `main()` method is already secretly annotated with `@Configuration` and `@ComponentScan`, so you don't need to do anything other than annotate `Cls` with `@Component`). 

Specifically, using `@ComponentScan` in this way specifies that, at runtime, if a class "at or below" the level of `Cls` in the directory hierarchy is annotated by `@Component` or by an annotation whose parent annotation* is `@Component`, then that class will be used to construct a Spring bean. Notably, the annotations `@Service`, `@Repository`, and `@Controller` all have `@Component` as a parent annotation.

*One annotation is considered to be the "child annotation" of another annotation if it is [meta-annotated](https://dzone.com/articles/what-are-meta-annotations-in-java) by that other annotation.

### Sidenote: annotation "inheritance" in Spring

As is noted in [this Stack Overflow answer](https://stackoverflow.com/a/18585833), Spring Framework's [`AnnotationUtils` class](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/annotation/AnnotationUtils.html) has a method that tests whether an annotation is equal to or is annotated with another annotation. I'm making an educated guess that Spring uses this sort of inheritance testing for annotations all over the place.

### Differences between `@Service`, `@Repository`, and `@Controller`

`@Service`, `@Repository`, and `@Controller` are similar in that they are child annotations of `@Component` (i.e. they are all meta-annotated by `@Component`). What are some differences?

- `@Service` indicates to the programmer that the class it annotates contains "business logic". Other than that, it doesn't enable any behind-the-scenes behavior. The Spring devs may change this some day.
- `@Repository` "is a marker for any class that fulfils the role or stereotype of a repository (also known as Data Access Object or DAO). Among the uses of this marker is the automatic translation of exceptions [from implementation exceptions to a Spring exception]" (from [here](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-stereotype-annotations)).
- `@Controller` must annotate a class if we want to use annotations from Spring Web MVC of the form `@<*request type*>Mapping`. These annotations are used for setting up HTTP API endpoints.

Extra readings: [@Component vs. @Service vs. @Repository vs. @Controller](https://stackoverflow.com/questions/6827752/whats-the-difference-between-component-repository-service-annotations-in), [@Component vs. @Bean](https://stackoverflow.com/questions/10604298/spring-component-versus-bean).

# Implementing dependency injection

We now know how to configure Spring beans, but don't yet know anything about how to actually dependency-inject Spring beans into other Spring beans. We describe how to do so in this section.

Though, before we describe how to do so, there is a little more prerequisite knowledge we should cover.

## More prerequisite knowledge

### Convention: "bean definition"

For the rest of this document, the term *bean definition* will refer to a method annotated with `@Bean` that returns an object instance or a class annotated with `@Component`.

### Bean scopes

Every bean definition has an associated "scope".

The default (and most important) scope is `singleton`. If a bean is of `singleton` scope, all references to that bean access the same Java object. `singleton` scope is used to achieve dependency sharing, which, if you recall the above "Dependency injection" section, is one of the key advantages of using an IoC container.

The second most important scope is `prototype`. If a bean is of `prototype` scope, different references to that bean are references to different Java objects. 

The four other scopes, `request`, `session`, `application`, and `websocket`, can only be used in a "web-aware application context," and are less commonly used. Don't worry about these ones.

### Terminology: "plain old Java objects" ("POJOs")

A "plain old Java class" is a class that does not depend on an application framework such as Spring. Basically, since most Spring features are handled with annotations, a plain old Java class is a class without any Spring annotations. 

Unfortunately, people say "plain old Java object" instead of "plain old Java class", so we speak of POJOs instead of POJCs. 

POJOs are often used in Spring apps in combination with not-POJOs to represent "more concrete" objects (such as an `Employee`, etc.).

Extra reading: http://www.shaunabram.com/beans-vs-pojos/.

## Implementing somewhat-manual dependency injection

Now, we are actually ready to use Spring Framework to implement the dependency injection design pattern.

Suppose we've configured a Spring bean named `Cls1` that has a reference to a Spring bean `Cls2`:

```java
@Component
public class Cls1
{
     private Cls2 cls2;
     public Cls2 getCls2() { return cls2; }
     public void setCls2(Cls2 cls2) { this.cls2 = cls2;}
}

@Component
public class Cls2 { ... }
```

We want to inject an instance of the `Cls2` into our `Cls1` bean at runtime. To do so, we need a reference to the Spring IoC container.

The interfaces `BeanFactory` and `ApplicationContext` both represent the IoC container. Since `ApplicationContext` extends `BeanFactory`, and therefore has more functionality, `ApplicationContext` should be used in most situations. 

We perform dependency injection by using `ApplicationContext` as follows:

```java
public class Application
{
   public static void main(String[] args)
  {
       /* <package> is the package inside which to look for @Configuration classes and in which to perform @ComponentScan. For example,    
       <package> might be "com.perficient.techbootcamp.*" */
       ApplicationContext ctx = new annotationConfigApplicationContext("<package>");
       
       /* The below assumes that a class named Cls has been configured as a bean (recall, this is done
       by using @Component and @ComponentScan or by using @Bean). */
       Cls1 cls1 = ctx.getBean(Cls1.class);
       
       /* Perform dependency injection: inject an instance of cls2 into the bean cls1. */
       Cls2 cls2 = new Cls2();
       cls1.setCls2(cls2);
  }
}
```

The above code is adapted from [Macro Behler's article](https://www.marcobehler.com/guides/spring-framework#spring-ioc-dependency-container).

## Implementing dependency injection with `@Autowired`

In Spring Framework, one typically uses annotations that execute the effect of the above dependency injection behind the scenes. Specifically, one uses the `@Autowired` annotation. When `@Autowired` is present on a bean's field, an instance of that field's type will be injected into that field at runtime.

So, if we want to replicate the functionality of the above, we would write the following:

```java
@Component
public class Cls2 { ... }

@Component
public class Cls1
{
     @Autowired
     private Cls2 cls2;

     public Cls2 getCls2() { return cls2; }
     // Notice, no setter necessary.
}

public class Application
{
     public static void main(String[] args)
     {     
           ApplicationContext ctx = new annotationConfigApplicationContext("<package>");
       
           /* The below code has been commented out because it is unnecessary.
              The above @Autowired annotation tells Spring Framework to inject
              a reference to the Cls2 bean into the Cls1 bean at runtime. */
       
           // Cls1 cls1 = ctx.getBean(Cls1.class);
           // Cls2 cls2 = new Cls2();
           // cls1.setCls2(cls2);
   }
}
```

### Field injection with `@Autowired`

You may wonder how it is possible to inject an instance of `Cls2` into `cls1` when `Cls1` has no `setCls2()` method. After thinking about it for a second, you might suspect that injection is done by using `Cls1`'s constructor. This is actually not the case. (In the above code, `Cls1` doesn't have a with-args constructor!). When `@Autowired` annotates a bean's field, then, at runtime, the IoC container uses this [Java reflection technique](https://stackoverflow.com/questions/32716952/set-private-field-value-with-reflection) to modify the field, even if it's `private`.

Placing `@Autowired` on a field thus constitutes *field injection*.

#### Using `@Autowired` on fields is bad practice

According to [this article](https://blog.marcnuri.com/field-injection-is-not-recommended/), using field injection is bad practice because it disallows you from marking fields as `final` . (You want to be able to mark fields as `final` when appropriate because doing so prevents you from getting into a circular dependency situation).

More reasons why field injection is bad: https://dzone.com/articles/spring-di-patterns-the-good-the-bad-and-the-ugly.

### Using `@Autowired` on constructors and setters

`@Autowired` can also be used on constructors or setters to inject a parameter into a constructor or setter at runtime.

### The `@Qualifier` annotation

Because a bean could have an `@Autowired` field whose type is an interface, and because multiple classes may implement the same interface, it can be necessary to specify which implementation of the interface is meant to be dependency-injected. This is done with the `@Qualifier` annotation, as follows:

```java
public interface Intf { ... }

@Qualifier("impl1")
@Component
public class Impl1 extends Intf { ... }

@Qualifier("impl2")
@Component
public class Impl2 extends Intf { ... }

public class Cls
{
     @Autowired
     @Qualifier("impl1")
     private Cls1 cls1Instance; // at runtime, cls1Instance will be set to a Cls1 instance
   
     @Autowired
     @Qualifier("impl2")
     private Cls2 cls2Instance; // at runtime, cls1Instance will be set to a Cls1 instance
}
```

Here are the specifics of how field-names are matched to bean-names:

- Define the *qualifier-name* of a bean definition or field to be: (1) the argument of the `@Qualifier` annotation attached to said bean definition or field, if the bean definition or field is indeed annotated with `@Qualifier`, and (2) the name of the class associated with the bean definition, if the bean definition or field is not annotated with `@Qualifier`.
- When no `@Qualifier` annotation is present on a field, then the class whose case-agnostic qualifier name is equal to the case-agnostic name of the field is what is dependency-injected into the field. ("Case agnostic" means "ignore case").

# End

This concludes my introduction to Spring Framework for Java. I hope you've gained a sense as to how Spring Framework allows you to implement dependency injection!