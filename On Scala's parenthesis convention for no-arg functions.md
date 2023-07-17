One might be confused or even angered when they learn about Scala's convention regarding parenthesis usage for no-arg functions.

The convention is this: given a no-arg function, you use put parentheses next to the function call *only if* the function has side effects. So, you would invoke a function named `printCurrentState` by writing `printCurrentState()`, since `printCurrentState` has the side effect of printing output to the console. On the other hand, you would invoke a function named `getCurrentState` by simply writing `getCurrentState`, since `getCurrentState` presumably just returns a value and does nothing else.

But why? Here, I'll present a quick analysis to convince you of why this convention makes sense.

Let's consider an arbitrary no-arg function. An arbitrary no-arg function falls into one of the following categories:

- `void` methods 
  - with side effects ("no-arg subroutine")
  - with no side effects ("useless method")
- methods with a return value
  - with no side effects ("a method that philosophically represents a variable", i.e. a "getter method")
  - with side effects ("variable retrieval and subroutine").

If we ignore the possibility of "useless methods", then we can rearrange the remaining three options to see that any given no-arg method is either

- a `void` method with side effects
- a method with a return value and no side effects
- a method with a return value and side effects.

That is, every no-arg method is in practice either

- a method with side effects
- a method with a return value and no side effects, i.e., a "getter method."

"Getter methods" in a philosophical sense "almost variables" because it is not completely inaccurate to think of their invocations as peeks into the state of reality rather than as a value returned by work done behind-the-scenes.

Since every no-arg method is either a "getter method" or not, it is syntactically unambiguous to establish the convention of invoking "getter methods" without parentheses (). More importantly, it is pleasing, as **the removal of parentheses emphasizes the interpretation of "getter methods" as being variables**.