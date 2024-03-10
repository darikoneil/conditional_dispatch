# conditional_dispatch
Simple little ecorator for dispatching a function's implementation according to registered conditional statements. While using a similar syntax to python's standard library's [singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch), it supports arbitrary conditional statements on single or multiple input arguments. 

**Dispatch using value example**
```
  @conditional_dispatch
  def my_function(argument):
    print("DEFAULT")

  @my_function.register(lambda arg: arg > 10)
  def _(argument):
    print("GREATER THAN TEN")

  @my_function.register(lambda arg: arg < 10)
  def _(argument):
    print("LESS THAN TEN)

  my_function(1)
"LESS THAN TEN"

  my_function(11)
"GREATER THAN TEN"

  my_function(10)
"DEFAULT"
```

**Dispatch using number of arguments example**
```
@conditional_dispatch
def my_function(*args):
  return "A"

@my_function.register(lambda args: len(args) == 1
def _(*args):
  return "B"

@my_function.register(lambda args: len(args) == 2
def _(*args):
  return "C"

  my_function()
"A"
  my_function(1)
"B"
  my_function(1,2)
"C"
```

**Multidispatching**
```
  @conditional_dispatch
  def my_function(arg0, arg1):
    print("DEFAULT")

  @my_function.register(lambda arg0, arg1: isinstance(arg0, tuple) and isinstance(arg1, list))
  def _(argument):
    print("TUPLE & LIST)

  my_function((1, 0), [0, 1])
"TUPLE & LIST"

  my_function((1, 0), 0)
"DEFAULT"
```


