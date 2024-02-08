# conditional_dispatch
Decorator for dispatching a function's implementation according to registered conditional statements. While using a similar syntax to python's standard library's [singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch), it supports arbitrary conditional statements on single or multiple input arguments. 

**Simple Example**
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

**Multidispatching**
```
  @conditional_dispatch
  def my_function(arg0, arg1):
    print("DEFAULT")

  @my_function.register(lambda arg0, arg1: isinstance(arg0, tuple) && isinstance(arg1, list)
  def _(argument):
    print("TUPLE & LIST)

  my_function((1, 0), [0, 1])
"TUPLE & LIST"

  my_function((1, 0), 0)
"DEFAULT"
```

**Dispatching Class Methods**
```
  class AnimalFactory:
    """
    Cool Animal Factory
    """

    @classmethod
    @conditional_dispatch
    def cool_cat_creator(instructions):
      return "BASIC COOL CAT"

    @staticmethod
    @cool_cat_creator.register(lambda instructions: instructions == "calico")
    def calico_creator(instructions):
      return "COOL CALICO CAT"
```
