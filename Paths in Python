# Paths in Python

As a result of its evolution over time, Python has a couple different paradigms for working with file paths.

## Path strings

Before Python 3.4, paths were primarily handled as `string`s with the `os.path` standard module. This provided platform agnostic path manipulation.

## `pathlib`: `PurePath` and `Path`

In Python 3.4, the `pathlib` standard module was introduced. It defines the `PurePath` and `Path` classes, and overloads the operator `/` so that it can be used to join paths together. * `PurePath`s are objects wrapping path strings that don't come with methods for filesystem access. Platform-specific path conventions are also represented as predefined `PurePath` instances. * `Path` is a subclass of `PurePath` with additional methods for filesystem access.

## The `os.PathLike` protocol

Until Python 3.6, many core standard library functions didn't support `pathlib` (i.e. `PurePath` or `Path`) because they only accepted `string`s as argument. If you wanted to use a `pathlib` representation of your path in a call to a standard library function, you'd often to cast the `pathlib` representation to a string, call the library function, and then convert the result back into a `pathlib` representation.

The `os.PathLike` protocol was introduced in Python 3.6 to solve this problem. Now, many standard library functions that were previously incompatible with `pathlib` have been rewritten to accept `os.PathLike` arguments (or `string`s, for backwards compatability). Since the `pathlib` classes `PurePath` and `Path`now implement this protocol, they can be passed to any standard library function taking in a `os.PathLike` argument.
