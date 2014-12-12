# Public API Specification

## Object.observe(O, callback, accept = undefined)

When the `observe` method is called with arguments Object _O_, function _callback_ and _accept_ the following steps are taken:

  1. If Type(_O_) is not Object, throw a **TypeError** exception.
  1. If IsCallable(_callback_) is **false**, throw a **TypeError** exception.
  1. If TestIntegrityLevel(_callback_, `"frozen"`) is **true**, throw a **TypeError** exception.
  1. If _accept_ is **undefined**, then
    1. Let _acceptList_ be a new List containing `"add"`, `"update"`, `"delete"`, `"reconfigure"`, `"setPrototype"` and `"preventExtensions"`.
  1. Else
    1. Let _acceptList_ be a new List.
    1. If Type(_accept_) is not Object, throw a **TypeError** exception.
    1. Let _lenValue_ be the result of Get(_accept_, `"length"`)
    1. ReturnIfAbrubt(_lenValue_).
    1. Let _len_ be ToLength(_lenValue_).
    1. ReturnIfAbrupt(_len_).
    1. Let _nextIndex_ be 0.
    1. While _nextIndex_ < _len_, repeat
      1. Let _next_ be the result of Get(_accept_, ToString(_nextIndex_)).
      1. ReturnIfAbrupt(_next_).
      1. Let _nextString_ be ToString(_next_).
      1. ReturnIfAbrupt(_nextString_).
      1. Append _nextString_ to _acceptList_.
      1. Let _nextIndex_ be _nextIndex_ + 1.
  1. Let _notifier_ be GetNotifier(_O_).
  1. Let _changeObservers_ be the value of _notifier_`s [[ChangeObservers]] internal slot.
  1. If _changeObservers_ already contains a _record_ whose _callback_ property is the same as _callback_, then
    1. Call CreateDataProperty(_record_, `"accept"`, _acceptList_).
    1. Return _O_.
  1. Let _observerRecord_ be ObjectCreate(%ObjectPrototype%).
  1. Perform CreateDataProperty(_observerRecord_, `"callback"`, _callback_).
  1. Perform CreateDataProperty(_observerRecord_, `"accept"`, _acceptList_).
  1. Append _observerRecord_ to the end of the _changeObservers_ list.
  1. Let _observerCallbacks_ be [[ObserverCallbacks]].
  1. If _observerCallbacks_ already contains _callback_, return _O_.
  1. Append _callback_ to the end of the _observerCallbacks_ list.
  1. Return _O_.



## Object.unobserve(O, callback)

When the `unobserve` method is called with arguments Object _O_ and function _callback_ the following steps are taken:

  1. If Type(_O_) is not Object, throw a **TypeError** exception.
  1. If IsCallable(_callback_) is **false**, throw a **TypeError** exception.
  1. Let _notifier_ be GetNotifier(_O_).
  1. Let _changeObservers_ be the value of _notifier_'s [[ChangeObservers]] internal slot.
  1. If _changeObservers_ does not contain a record whose _callback_ property is the same as _callback_, return _O_.
  1. Remove the record whose _callback_ property is the same as _callback_ from the _changeObservers_ list.
  1. Return _O_.




## Array.observe

A new function Array.observe(O, callback) is added, which is equivalent to

```js
function(O, callback) {
  return Object.observe(O, callback, ["add", "update", "delete", "splice"]);
}
```



## Array.unobserve

A new function Array.unobserve(o, callback) is added, which is equivalent to

```js
function(O, callback) {
  return Object.unobserve(O, callback);
}
```


## Object.deliverChangeRecords

When the `deliverChangeRecords` method is called with argument _callback_, the following steps are taken:

  1. If IsCallable(_callback_) is **false**, throw a **TypeError** exception.
  1. Repeat
    1. Let _result_ be DeliverChangeRecords(_callback_).
    1. If _result_ is **false** return.


## Object.getNotifier

When the `getNotifier` method is called with argument _O_, the following steps are taken:

  1. If Type(_O_) is not Object, throw a **TypeError** exception.
  1. If _O_ is frozen, return **null**.
  1. Return the result of GetNotifier(_O_).
