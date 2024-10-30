# conditional_dispatch
Simple little decorator for dispatching a function's implementation according to registered conditional statements. Feel free to copy/paste. While using a similar syntax to python's standard library's [singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch), it supports arbitrary conditional statements on single or multiple input arguments. 

**Simple Example: dispatch using value example**
```
  @conditional_dispatch
  def my_function(argument):
    print("DEFAULT")

  @my_function.register(lambda argument: argument > 10)
  def _(argument):
    print("GREATER THAN TEN")

  @my_function.register(lambda argument: argument < 10)
  def _(argument):
    print("LESS THAN TEN)

  my_function(1)
"LESS THAN TEN"

  my_function(11)
"GREATER THAN TEN"

  my_function(10)
"DEFAULT"
```
