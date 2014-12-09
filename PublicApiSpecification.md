# Object.observe: Public API Specification

## Object.observe

A new function Object.observe(O, callback, accept) is added, which behaves as follows:
  - If Type(//O//) is not Object, throw a TypeError exception.
  - If IsCallable(//callback//) is **false**, throw a TypeError exception.
  - If IsFrozen(//callback//) is **true**, throw a TypeError exception.
  - If //accept// is **undefined**
    - Let //acceptList// be a new List containing ["add", "update", "delete", "reconfigure", "setPrototype", "preventExtensions"]
  - Else:
    - Let //acceptList// be a new List.
    - If Type(//accept//) is not Object, throw a TypeError exception.
    - Let //lenValue// be the result of Get(//accept//, "length")
    - Let //len// be ToLength(//lenValue//).
    - ReturnIfAbrupt(//len//).
    - Let //nextIndex// be 0.
    - Repeat while //nextIndex// < //len//
      - Let //next// be the result of Get(//accept//, ToString(//nextIndex//)).
      - Let //nextString// be ToString(//next//).
      - ReturnIfAbrupt(//nextString//).
      - Append //nextString// to //acceptList//.
      - Let //nextIndex// be //nextIndex// + 1.
  - Let //notifier// be the result of calling %%[[GetNotifier]]%%, passing //O//.
  - Let //changeObservers// be %%[[ChangeObservers]]%% of //notifier//.
  - If //changeObservers// already contains a //record// whose //callback// property is the same as //callback//, then
    - Perform CreateOwnDataProperty(//record//, "accept", //acceptList//) and
    - Return //O//.
  - Let //observerRecord// be the result of performing ObjectCreate().
  - Perform CreateOwnDataProperty(//observerRecord//, "callback", //callback//).
  - Perform CreateOwnDataProperty(//observerRecord//, "accept", //acceptList//).
  - Append //observerRecord// to the end of the //changeObservers// list.
  - Let //observerCallbacks// be %%[[ObserverCallbacks]]%%.
  - If //observerCallbacks// already contains //callback//, return //O//.
  - Append //callback// to the end of the //observerCallbacks// list.
  - Return //O//.



## Object.unobserve

A new function Object.unobserve(O, callback) is added, which behaves as follows:
  - If Type(//O//) is not Object, throw a TypeError exception.
  - If IsCallable(//callback//) is not **true**, throw a TypeError exception.
  - Let //notifier// be the result of calling %%[[GetNotifier]]%%, passing //O//.
  - Let //changeObservers// be %%[[ChangeObservers]]%% of //notifier//.
  - If //changeObservers// does not contain a record whose //callback// property is the same as //callback//, return //O//.
  - Remove the record whose //callback// property is the same as //callback// from the //changeObservers// list.
  - Return //O//.




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

A new function Object.deliverChangeRecords(callback) is added, which behaves as follows:
  - If IsCallable(//callback//) is not **true**, throw a TypeError exception.
  - Call %%[[DeliverChangeRecords]]%% with arguments: //callback// repeatedly until it returns false (no records were pending for delivery and callback no invoked).
  - Return.


## Object.getNotifier

A new function Object.getNotifier(O) is added, which behaves as follows:

  - If Type(//O//) is not Object, throw a TypeError exception.
  - If //O// is frozen, return **''null''**.
  - Return the result of calling %%[[GetNotifier]]%%, passing //O//.
