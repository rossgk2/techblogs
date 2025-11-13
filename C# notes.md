# C# features by category and version

## Properties

| Feature                                                                 | Description, if not self-explanatory | Version     |
|-------------------------------------------------------------------------|-------------|-------------|
| Properties, [automatically implemented properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties), [default values for properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) | Syntactic sugar for getter and setter methods | 1.0, 3.0, 6.0 |
| `init` accessor for properties                                          |            | 9           |
| Implicit backing of a property by a field with `field` keyword |  | 14 |

## Defining types

| Feature                                                      | Description, if not self-explanatory                         | Version |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------- |
| Inline constructors                                          | e.g. `public class Cls(string field1, int field2)`)          | 12      |
| [Extension methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) | Extension methods are instance methods that can't access `private` or `protected` members defined outside of an already-defined type. They are implemented as syntactic sugar for `static` methods that take the already-defined type as argument | 3.0     |
| `record class` (or `record`) and `record struct`             | Immutable types with field-based equality. `record class` defines a reference type and `record struct` defines a value type. | 9       |

## Using types

| Feature                                                      | Description, if not self-explanatory | Version |
| ------------------------------------------------------------ | ------------------------------------ | ------- |
| Type-parameters (generics)                                   |                                      | 2.0     |
| Implicitly typed variables with `var`                        |                                      | 3.0     |
| Dynamically typed variables with `dynamic`                   | E.g. `dynamic x = 1; x = "hi";`      | 4.0     |
| [Make value types (`int`, `bool`, `struct`) reference types with `ref`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations#reference-variables) |                                      | 7.0     |
| Nullable value types via `?`                                 |                                      | 2.0     |
| Nullable reference types via `?`                             |                                      | 8.0     |
| [Nullish ternary operator `??`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) |                                      | 8.0     |
| [Non-null conditional access with `.` and `.[]`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) |                                      | 5.0     |

## Intuitive key-value support

| Feature                                                      | Version                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Instantiation of existing classes](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers) with key-value syntax | 3.0                                                          |
| [Instantiation of anonymous classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types) with key-value syntax | 3.0                                                          |
| `Tuple`                                                      | A reference type version of a tuple. (See the additional notes for a reminder on what a tuple is.) |
| `ValueTuple`s (e.g. `(int, string) x = (1, "hi");`)          | A value type version of a tuple. Note, all "tuple literals" are `ValueTuple`s, *not* `Tuple`s. (See the additional notes for a reminder on what a tuple is.) |

## Methods

| Feature                                                                 | Description, if not self-explanatory | Version     |
|-------------------------------------------------------------------------|-------------|-------------|
| Anonymous methods ("lambda expressions")                       |          | 3.0         |
| [=> syntax for no-arg methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members), and for all method-like members |    | 6.0, 7.0   |
| `in` keyword for method parameters | Basically `readonly` for method parameters | 7.2 |
| `out` keyword for method parameters | Basically "must be written to" for method parameters | 1.0 |
| [All functions automatically support named arguments](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments), all methods automatically support named and/or positional arguments |    | 4.0, 7.2   |

## Asynchronous programming

| Feature           | Version |
|-------------------|---------|
| `async` and `await` | 5.0     |
| `Main()` can be `async` | 7.1     |

---

## Miscellaneous

| Feature                                                      |                                                              | Version |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------- |
| [String interpolation, like  `$"The value of x is: {x}"`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated) |                                                              | 6.0     |
| [`static import` imports only static members](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-directive) |                                                              | 6.0     |
| [`catch when` for exception filtering](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/when) |                                                              | 6.0     |
| Nested functions                                             |                                                              | 7.0     |
| `throw` expressions                                          | Prior to this, `throw` was only a *statement*, not an expression that could be used inside other statements. | 7.0     |
| Range-indexing operator `..`                                 |                                                              | 8.0     |
| [Top level statements - programs don't need a `Main()` method](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history) |                                                              | 9       |
| `required` members                                           |                                                              | 11      |

# Examples

## Queries

To execute a query, you can either use a "query expression":

```
IEnumerable<int> highScoresQuery =
    from score in scores
    where score > 80
    orderby score descending
    select score;
```

or you can use the LINQ (language-integrated query) extension methods defined on `IEnumerable`:

```
IEnumerable<int> highScoresQuery = scores
    .Where(score => score > 80)
    .OrderByDescending(score => score);
```

## Examples of key-value support

Instantiation of an existing class:


```c#
Cls obj = new Cls { field1 = 10, field2 = "hi" };
Cls obj = new Cls("hi"){ field1 = 10 };
```

Instantiation of an anonymous class:


```c#
var obj = new { field1 = 10, field2 = "hi" };
```

Note, trying to specify types for `field1` and `field2` will result in a compilation error.

This JSON

```json
{
    "a1": 1,
    "a2": [
        {
            "c3": "hi",
            "c4": "bye"
        },
        2
    ]
}
```

could be represented by instantiating an anonymous class like this:

```c#
var obj = new
{
    a1 = 1,
    a2 = new object[]
    {
        new { c3 = "hi", c4 = "bye" },
        2
    }
};
```

# Additional notes

## Tuples vs. lists

I find that Python version of things highlights the core ideas best. In Python, lists and tuples can both contain objects of multiple types. The core difference is that lists are mutable, while tuples are immutable.

This is relevant for understanding the purpose of `Tuple` and `ValueTuple`. A `Tuple` is a reference type version of the tuple type. A `ValueTuple` is a value type version of the tuple type.

Note that even while tuples are language-agnostically immutable lists, they do not implement `List` or `List<T>` in C#. They are their own separate type. 

## Classes vs. structs

I used to think that in the most abstract, language-agnostic sense, structs only specified state, while classes specify both state and behavior. This is not true. In C, which is where the word "struct" comes from, structs can contain function pointers, and thus in spirit have instance methods. And in C++, structs can simply straightforwardly have instance methods.

The real difference is that "struct" is synonymous with "value type", and "class" is synonymous with "reference type". As a secondary concern, structs do not support inheritance, while classes do. 

This is relevant for understanding the difference between `record class` and  `record struct`. The

## Properties

We like to think of properties as being fields plus syntactical sugar; technically, they are not. You might think that you could use `public readonly string field { get; init; }` in a class, but you cannot, since `readonly` can only be applied to fields and `init` can only be used in properties.

## Immutability constructs: `const`, `readonly`, `init`, `record class` and `record struct`,  `sealed`



## `await foreach` vs. `yield`

## Access and immutability modifiers

Unfortunately, the Venn diagram of modifiers that can be applied to fields and modifiers that can be applied to function arguments doesn't have that much of an intersection.

| Modifier           | Description                                                  | Field   | Property | Function argument | Method  | Local Variable            |
| ------------------ | ------------------------------------------------------------ | ------- | -------- | ----------------- | ------- | ------------------------- |
| public             | Accessible from any other code                               | **Yes** | **Yes**  | No                | **Yes** | No                        |
| private            | Accessible only within the containing class or struct        | **Yes** | **Yes**  | No                | **Yes** | No                        |
| protected          | Accessible within the containing class and derived classes   | **Yes** | **Yes**  | No                | **Yes** | No                        |
| internal           | Accessible within the same assembly                          | **Yes** | **Yes**  | No                | **Yes** | No                        |
| protected internal | Accessible within same assembly or derived classes           | **Yes** | **Yes**  | No                | **Yes** | No                        |
| private protected  | Accessible within class or derived classes in same assembly  | **Yes** | **Yes**  | No                | **Yes** | No                        |
| static             | Belongs to the type itself rather than an instance           | **Yes** | **Yes**  | No                | **Yes** | **Yes**                   |
| readonly           | Can only be assigned during declaration or in constructor    | **Yes** | No       | No                | No      | No                        |
| const              | Compile-time constant                                        | **Yes** | No       | No                | No      | **Yes**                   |
| volatile           | Field may be modified by multiple threads                    | **Yes** | No       | No                | No      | No                        |
| new                | Hides a member inherited from a base class                   | **Yes** | **Yes**  | No                | **Yes** | No                        |
| abstract           | Must be overridden in derived class                          | No      | **Yes**  | No                | **Yes** | No                        |
| virtual            | Can be overridden in derived class                           | No      | **Yes**  | No                | **Yes** | No                        |
| override           | Overrides a base class member                                | No      | **Yes**  | No                | **Yes** | No                        |
| sealed             | Prevents further overriding                                  | No      | **Yes**  | No                | **Yes** | No                        |
| extern             | Declared externally (e.g., via DLL)                          | **Yes** | **Yes**  | No                | **Yes** | No                        |
| unsafe             | Allows use of pointer types                                  | **Yes** | **Yes**  | **Yes**           | **Yes** | **Yes**                   |
| params             | Allows a variable number of arguments                        | No      | No       | **Yes**           | No      | No                        |
| ref                | Passes argument by reference                                 | No      | No       | **Yes**           | No      | **Yes** (with ref return) |
| out                | Passes argument by reference, must be assigned in method     | No      | No       | **Yes**           | No      | No                        |
| in                 | Passes argument by reference, read-only                      | No      | No       | **Yes**           | No      | No                        |
| required           | Ensures property is set during object initialization (C# 11+) | No      | **Yes**  | No                | No      | No                        |
