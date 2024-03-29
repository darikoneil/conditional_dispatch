# conditional_dispatch
Simple little decorator for dispatching a function's implementation according to registered conditional statements. While using a similar syntax to python's standard library's [singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch), it supports arbitrary conditional statements on single or multiple input arguments. 

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

**Actual Example Use-Case**
```
class CallbackRequest:
    """
    This class requests the injection of the specified callback into the requesting instance.

    :param requesting_instance: The instance requesting the callback injection
    :param callback_key: The key of the callback to inject
    :raises CallbackError: Raised when the callback is called but was not injected
    :raises CallbackKeyWarning: Raised when the callback key is not present in the requesting instance, which could
        result in the requesting instance being unable to call the callback
    :raises DispatchError: Raised when the callback argument is neither a callback function, not present in the
        :class:`CallbackRegistry <probe.registries.CallbackRegistry>`, and not None Type or if dispatching
         otherwise fails

    .. seealso:: :class:`CallbackFunction <probe.extensions.CallbackFunction>`
    """
    def __init__(self, requesting_instance: Any, callback_key: str) -> None:
        """
        This class requests the injection of the specified callback into the requesting instance.

        :param requesting_instance: The instance requesting the callback injection
        :param callback_key: The key of the callback to inject
        :raises CallbackError: Raised when the callback is called but was not injected
        :raises CallbackKeyWarning: Raised when the callback key is not present in the requesting instance, which could
            result in the requesting instance being unable to call the callback
        :raises DispatchError: Raised when the callback argument is neither a callback function, not present in the
            :class:`CallbackRegistry <probe.registries.CallbackRegistry>`, and not None Type or if dispatching
            otherwise fails

        .. seealso:: :class:`CallbackFunction <probe.extensions.CallbackFunction>`
        """
        self.requesting_instance = requesting_instance
        self.callback_key = callback_key

    def __str__(self) -> str:
        return f"Callback request with key {self.callback_key}"

    @conditional_dispatch
    def inject(self, callback: Union[CallbackFunction, CallbackRegistry, None],
               providers: Optional[Union[dict, WeakValueDictionary]] = None) -> None:
        """
        Injects the requested callback into the requesting instance. If the retrieved callback is a bound method,
        then its associated instance or class is attached to the callback's arguments by iterating through the
        providers. This method is dispatched based on the type of the callback. If the callback is
        a :class:`CallbackRegistry <probe.registries.CallbackRegistry>`, then the callback is retrieved from the
        registry. If the callback is a :class:`CallbackFunction <probe.extensions.CallbackFunction>`, then the callback
        is directly injected. If the callback is None Type, then the callback is mocked.

        :param callback: The callback to inject or the callback registry containing the callback
        :param providers: A weak-valued dictionary of providers used to resolve bound methods. A standard dictionary
            can be used (e.g., testing) but during runtime the providers will be in a weak-valued dictionary after
            being collected by :class:`ProviderTraverse <probe.injection.ProviderTraverse>`
        :raises CallbackKeyWarning: Raised when the callback key is not present in the requesting instance, which could
        result in the requesting instance being unable to call the callback
        :raises DispatchError: Raised when the callback argument is neither a callback function, not present in the
            :class:`CallbackRegistry <probe.registries.CallbackRegistry>`, and not None Type or if dispatching
            otherwise fails
        """
        raise DispatchError(self)

    @inject.register(lambda self, callback, providers=None:
                     isinstance(callback, CallbackFunction) and not isinstance(callback, BoundFunction))
    def _(self, callback: CallbackFunction, providers: Optional[Union[dict, WeakValueDictionary]] = None) -> None:
        """
        Injects the requested callback function directly into the requesting instance
        """
        if not hasattr(self.requesting_instance, self.callback_key):
            warnings.warn(CallbackKeyWarning(self.requesting_instance.__class__.__name__, self.callback_key),
                          stacklevel=2)
        setattr(self.requesting_instance, self.callback_key, callback)

    @inject.register(lambda self, callback, providers=None:
                     isinstance(callback, (BoundFunction, CallbackFunction)))
    def _(self, callback: BoundFunction, providers: Optional[Union[dict, WeakValueDictionary]] = None) -> None:
        """
        Attached the bound instance or class to the callback's arguments by iterating through the providers and
        constructing a partial function,
        """
        if providers is None:
            raise DispatchError(self)

        bound_provider = callback.__qualname__.split(".")[0]
        provider = next((provider for provider in providers.values() if provider.__class__.__name__ == bound_provider),
                        None)
        callback = partial(callback, provider)
        self.inject(callback, providers)

    @inject.register(lambda self, callback, providers=None: isinstance(callback, CallbackRegistry))
    def _(self, callback: CallbackRegistry, providers: Optional[Union[dict, WeakValueDictionary]] = None) -> None:

        callback = callback.get(self.callback_key)
        self.inject(callback, providers)

    @inject.register(lambda self, callback, providers=None: EMPTY(callback))
    def _(self, callback: None, providers: Optional[Union[dict, WeakValueDictionary]] = None) -> None:
        warnings.warn(NoOptionalCallbackWarning(self.requesting_instance.__class__.__name__,
                                                self.callback_key), stacklevel=2)
        callback = rename_callback(self.callback_key)(lambda *args, **kwargs: None)
        self.inject(callback, providers)

    def __call__(self, *args, **kwargs) -> None:
        """
        Notifies the user the callback has not been injected

        :raises CallbackError: Raised when the callback is called but was not injected
        """
        raise CallbackError(self.requesting_instance.__class__.__name__,
                            self.callback_key)
```
