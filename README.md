# conditional_dispatch
Simple little decorator to create generic functions with conditional dispatching. This is useful for breaking up a single function into
multiple implementations based on different conditions, without having to write a bunch of if-elif-else statements.
I hate those. You can also use it to implement a multiple-argument type dispatcher, which is pretty cool.
Feel free to copy/paste in your own projects! Uses syntax like python's standard library's [singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch) and is able to be used with class and instance methods.


## Examples

```
"""
Example of variable argument dispatch
"""

@conditional_dispatch
def my_function(*args: Any) -> None
  ...

@my_function.register(lambda *args: len(args) == 0)
def _(*args) -> None:
  print("One argument")

@my_function.register(lambda *args: len(args) > 0)
def _(*args) -> None:
  print("Multiple arguments")
```


```
"""
Example of multiple type dispatch
"""

@conditional_dispatch
def my_function(a: Any, b: Any) -> str:
  raise TypeError(f"Dispatching error: no support for {type(input)}")


@my_function.register(lambda a, b: isinstance(a, int) and isinstance(b, int))
def _(a: int, b: int) -> str:
  return "Both are ints"


@my_function.register(lambda a, b: isinstance(a, int) and a >= 0 and isinstance(b, str))
def _(a: int, b: int) -> str:
  return "a is an int and b is str"


@my_function.register(lambda a, b: isinstance(a, str) and isinstance(b, int))
def _(a: str, b: int) -> str:
  return "a is a str, b is an int"

@my_function.register(lambda a, b: isinstance(a, str) and isinstance(b, str))
def _(a: str, b: str) -> str:
  return "Both are strs"
```
