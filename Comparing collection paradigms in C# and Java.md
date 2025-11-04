# Comparing collection paradigms in C# and Java

## Abstract data types

| Abstract data type                    | Legacy C# interface | Modern C# interface | Legacy Java interface | Modern Java interface |
| ------------------------------------- | ------------------- | ------------------- | --------------------- | --------------------- |
| "Enumerable" or "iterable" collection | `IEnumerable`       | `IEnumerable<T>`    | `Iterable`            | `Iterable<T>`         |
| Collection                            | `ICollection`       | `ICollection<T>`    | `Collection`          | `Collection<T>`       |
| Dynamically-sized list                | `IList`             | `IList<T>`          | `List`                | `List<T>`             |

## Java's intuitive collection type hierarchy

In Java, collection type classes implement the interface that corresponds to their abstract data type. For instance, since `ArrayList<T>` and `LinkedList<T>` implement the same abstract data type- that of a dynamically sized list- they are both implementations of the `List<T>` interface. We have this inheritance graph:

- ```
  List<T>
  ```

  - `ArrayList<T>`
  - `LinkedList<T>`

This is intuitive and makes a lot of sense.

## C#'s unintuitive collection type hierarchy

Unfortunately, C# organizes its collection interface inheritance graph much less intuitively.

### `List<T>` is `ArrayList<T>` in spirit (even though `ArrayList<T>` doesn't actually exist)

In the C# standard library, the go-to implementation of `IList<T>`, called `List<T>`, is actually not abstract, like the name `List<T>` suggests, but is actually implemented by using a dynamically resizing array, just as is the `ArrayList<T>` of Java. Wouldn't it make more sense if the `List<T>`of C# were called `ArrayList<T>`?

Well, yes. And in fact, before type parameters came to C#, the `ArrayList` type *was* the go-to `IList` implementation. (`ArrayList` still hangs around today just to make this whole matter more confusing.) Unfortunately, there does *not exist* an `ArrayList<T>` type in C#, presumably because if there did then we would have the unintuitive fact that `ArrayList<object>` is not equal to `ArrayList`. (See the appendix for why this would happen.)

Given that the devs seemed to be okay with this inconvenience for `IEnumerable` and `ICollection`*, though, I'm not sure why they weren't okay with having the same inconvenience apply to `ArrayList`. I actually think it would be better if the inconvenience persisted for all collection types, not just some of them. That would be much more consistent and easier to remember!

Oh well. We are stuck with `List<T>`, an array implementation of `IList<T>`, being presented as being very generic when it's not.

\* `IEnumerable<T>` and `ICollection<T>` exist in spite of the fact that `IEnumerable<object>` and `ICollection<object>` are not equal to `IEnumerable` and `ICollection`.

### `LinkedList<T>` doesn't implement `List<T>`

What's more is that `LinkedList<T>`, which you would think is linked-list implementation of `IList<T>`, is in fact only an implementation of `ICollection<T>`.

Cleary, C# organizes the collection type hierarchy fundamentally differently than Java. Fundamentally, the rules for the collection type hierarchies are:

- (Java). Collection interfaces correspond to abstract data types.
- (C#). Each collection interface informally guarantees a performance standard associated with some implementation of an abstract data type. (And thus, each collection interface corresponds to some implementation of an abstract data type; interface is coupled to implementation. How unfortunate.)

In C#, `IList<T>` is associated with the performance standard of an array implementation of a list. Of course, a linked list implementation will not have the performance standard of an array implementation, so, according to the above rule for C#, `LinkedList<T>` must not implement `IList<T>`.

Of course, disregarding performance standards, a linked list could be used to implement an `IList<T>`, so it is guaranteed that a linked list could be used to implement any superinterface of `IList<T>`. This suggests an easy fix: we just make `LinkedList<T>` an implementor of `ICollection<T>`, which is the immediate superinterface of `IList<T>`.

That's how we end up with `LinkedList<T>` implementing `ICollection<T>` in C#.

## Summary of collection classes

### `IList` (C#), `List` interface (Java) implementations

| Implementation            | Notes           | Legacy C# class              | Modern C# class               | Legacy Java class                                   | Modern Java class                                            |
| ------------------------- | --------------- | ---------------------------- | ----------------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| Array                     | Feature-minimal | `Collection`                 | `Collection<T>`               | *(none)*                                            | (*none*)                                                     |
| Array                     | Feature-rich    | `ArrayList implements IList` | `List<T> implements IList<T>` | `ArrayList implements List`                         | `ArrayList<T> implements List<T>`                            |
| Bidirectional linked list |                 | *(none)*                     | *(none)*                      | `LinkedList implements List`, `Deque extends Queue` | `LinkedList<T> implements List<T>`, `Deque<T> extends Queue<T>` |

As noted above, the `LinkedList<T>` of C# does not implement `IList<T>`, since, in C#, `IList<T>`s informally guarantee the performance standard of an array-based implementation of a list.

### `IDictionary`  (C#), `Map` interface (Java) implementations

| Description                     | Legacy C# class                     | Modern C# class                                     | Legacy Java class                          | Modern Java class                                         |
| ------------------------------- | ----------------------------------- | --------------------------------------------------- | ------------------------------------------ | --------------------------------------------------------- |
| Dictionary                      | `Hashtable implements IDictionary`  | `Dictionary<K,V> implements IDictionary<K,V>`       | `HashMap implements Map`                   | `HashMap<K,V> implements Map<K,V>`                        |
| Sorted dictionary (tree-based)  | *(none)*                            | `SortedDictionary<K,V> implements IDictionary<K,V>` | `TreeMap implements SortedMap extends Map` | `TreeMap<K,V> implements SortedMap<K,V> extends Map<K,V>` |
| Sorted dictionary (array-based) | `SortedList implements IDictionary` | `SortedList<K,V> implements IDictionary<K,V>`       | *(none)*                                   | *(none)*                                                  |

Note how `SortedList` and `SortedList<K,V>`  are misleading names for what are really array-based implementations of `IDictionary` and `IDictionary<K,V>`.

### `ICollection` (C#), `Collection` interface (Java) implementations

| Description               | Legacy C# class                | Modern C# class                           | Legacy Java class                               | Modern Java class                                        |
| ------------------------- | ------------------------------ | ----------------------------------------- | ----------------------------------------------- | -------------------------------------------------------- |
| Bidirectional linked list | *(none)*                       | `LinkedList<T> implements ICollection<T>` | `Deque implements Queue implements Collection`  | `Deque<T> implements Queue<T> implements Collection<T>`  |
| Queue (FIFO)              | `Queue implements ICollection` | `Queue<T> implements ICollection<T>`      | `Queue implements Collection`                   | `Queue<T> implements Collection<T>`                      |
| Stack (LIFO)              | `Stack implements ICollection` | `Stack<T> implements ICollection<T>`      | `Deque implements Queue implements Collection`* | `Deque<T> implements Queue<T> implements Collection<T>`* |

\* Note, the Java documentation says that a double ended queue is Java's best implementation of a stack. Though there does exist a `Stack` type, using it is discouraged, since it inherits from the legacy `Vector` class.

### `ISet` (C#), `Set` interface (Java)

| Description           | Legacy C# | Modern C#                         | Legacy Java                    | Modern Java                          |
| --------------------- | --------- | --------------------------------- | ------------------------------ | ------------------------------------ |
| Hash set              | *(none)*  | `HashSet<T> implements ISet<T>`   | `HashSet implements Set`       | `HashSet<T> implements Set<T>`       |
| Sorted Set / Tree Set | *(none)*  | `SortedSet<T> implements ISet<T>` | `TreeSet implements SortedSet` | `TreeSet<T> implements SortedSet<T>` |

## Appendix: in C#, `T` is not equal to `T<object>` at runtime

Java uses *runtime type erasure* to implement type parameters. This has the implication that while `T<S1>` and `T<S2>` may be distinct at compile time, they will be both equal to `T`, and thus equal to each other, at runtime.

This is not the case in C#. The types  `T<S1>` and `T<S2>` are distinct at both compile time and runtime. While it *is* overall good that type parameters are properly represented at runtime in C#, there is the unfortunate side-effect that, since type parameters weren't an original language feature, the types `T` and `T<object>` are not equal at runtime. 

So, for instance, `IList` is unintuitively a different type than `IList<object>`.