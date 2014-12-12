# Public API Specification

## Object.observe

When the `observe` method is called with arguments _O_, _callback_ and _accept_ the following steps are taken:

  - If Type(_O_) is not Object, throw a **TypeError** exception.
  - If IsCallable(_callback_) is **false**, throw a **TypeError** exception.
  - If IsFrozen(_callback_) is **true**, throw a **TypeError** exception.
  - If _accept_ is **undefined**
    - Let _acceptList_ be a new List containing ["add", "update", "delete", "reconfigure", "setPrototype", "preventExtensions"]
  - Else:
    - Let _acceptList_ be a new List.
    - If Type(_accept_) is not Object, throw a **TypeError** exception.
    - Let _lenValue_ be the result of Get(_accept_, "length")
    - Let _len_ be ToLength(_lenValue_).
    - ReturnIfAbrupt(_len_).
    - Let _nextIndex_ be 0.
    - Repeat while _nextIndex_ < _len_
      - Let _next_ be the result of Get(_accept_, ToString(_nextIndex_)).
      - Let _nextString_ be ToString(_next_).
      - ReturnIfAbrupt(_nextString_).
      - Append _nextString_ to _acceptList_.
      - Let _nextIndex_ be _nextIndex_ + 1.
  - Let _notifier_ be the result of calling [[GetNotifier]], passing _O_.
  - Let _changeObservers_ be [[ChangeObservers]] of _notifier_.
  - If _changeObservers_ already contains a _record_ whose _callback_ property is the same as _callback_, then
    - Perform CreateOwnDataProperty(_record_, "accept", _acceptList_) and
    - Return _O_.
  - Let _observerRecord_ be the result of performing ObjectCreate().
  - Perform CreateOwnDataProperty(_observerRecord_, "callback", _callback_).
  - Perform CreateOwnDataProperty(_observerRecord_, "accept", _acceptList_).
  - Append _observerRecord_ to the end of the _changeObservers_ list.
  - Let _observerCallbacks_ be [[ObserverCallbacks]].
  - If _observerCallbacks_ already contains _callback_, return _O_.
  - Append _callback_ to the end of the _observerCallbacks_ list.
  - Return _O_.



## Object.unobserve

When the `unobserve` method is called with arguments _O_ and _callback_ the following steps are taken:

  - If Type(_O_) is not Object, throw a **TypeError** exception.
  - If IsCallable(_callback_) is **false**, throw a **TypeError** exception.
  - Let _notifier_ be the result of calling [[GetNotifier]], passing _O_.
  - Let _changeObservers_ be the value of _notifier_'s [[ChangeObservers]] internal slot.
  - If _changeObservers_ does not contain a record whose _callback_ property is the same as _callback_, return _O_.
  - Remove the record whose _callback_ property is the same as _callback_ from the _changeObservers_ list.
  - Return _O_.




## Array.observe

A new function Array.observe(o, callback) is added, which is equivalent to

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

  - If IsCallable(_callback_) is not **true**, throw a **TypeError** exception.
  - Call [[DeliverChangeRecords]] with arguments: _callback_ repeatedly until it returns false (no records were pending for delivery and callback no invoked).
  - Return.


## Object.getNotifier

When the `getNotifier` method is called with argument _O_, the following steps are taken:

  - If Type(_O_) is not Object, throw a **TypeError** exception.
  - If _O_ is frozen, return **null**.
  - Return the result of calling [[GetNotifier]], passing _O_.
