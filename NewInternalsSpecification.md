# Object.observe: New Internal Properties, Objects and Algorithms

## [[ObserverCallbacks]]

There is now an ordered list, [[ObserverCallbacks]] which is shared per event queue. It is initially empty.

Note: This list is used to provide a deterministic ordering in which callbacks are called.


## [[PendingChangeRecords]]

Every function now has a [[PendingChangeRecords]] internal slot which is an ordered list of ChangeRecords.
It is initially empty.

Note: This list gets populated with change records as the objects that this function is observing are mutated. It gets emptied when the change records are delivered.



### [[Notifier]]

Every object _O_ now has a [[Notifier]] internal slot which is initially **''undefined''**.

Note: This gets lazily initialized to a notifier object which is an object with the %NotifierPrototype% as its [[Prototype]].

## Notifier Objects

A Notifier Object is an object returned from `Object.getNotifier`. There is not a named constructor for Notifier Objects.

### Properties of Notifier Prototype

### The %NotifierPrototype% Object 

All Notifier Objects inherit properties from the %NotifierPrototype% intrinsic object. The %NotifierPrototype% intrinsic object is an ordinary object and its [[Prototype]] internal slot is the %ObjectPrototype% intrinsic object (19.1.3). In addition, %NotifierPrototype% has the following properties:

### %NotifierPrototype%.notify(_changeRecord_)

  1. Let _O_ be the **this** value.
  1. If Type(_O_) is not Object, throw a **TypeError** exception.
  1. If _O_ does not have a [[Target]] internal slot return.
  1. Let _type_ be Get(_changeRecord_, `"type"`).
  1. ReturnIfAbrubt(_type_).
  1. If Type(_type_) is not string, throw a **TypeError** exception.
  1. Let _target_ be the value of _O_'s [[Target]] internal slot.
  1. Let _newRecord_ be ObjectCreate(%ObjectPrototype%).
  1. Let _desc_ be the PropertyDescriptor{[[Value]]: _target_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
  1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _newRecord_ passing `"object"` and _desc_ as arguments.
  1. Assert: _success_ is **true**.
  1. Let _names_ be the result of calling the [[Enumerate]] internal method of _changeRecord_ with no arguments.
  1. Repeat for each element _N_ of _names_ in List order,
    1. If _N_ is not `"object"`, then
      1. Let _value_ be Get(_changeRecord_, _N_).
      1. ReturnIfAbrubt(_value_).
      1. Let _desc_ be the PropertyDescriptor{[[Value]]: _value_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
      1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _newRecord_ passing _N_ and _desc_ as arguments.
      1. Assert: _success_ is **true**.
  1. Set the value of the [[Extensible]] internal slot of _newRecord_ to **false**.
  1. Call EnqueueChangeRecord(_target_, _newRecord_).


### %NotifierPrototype%.performChange(changeType, changeFn)

  1. Let _O_ be the **this** value.
  1. If Type(_O_) is not Object, throw a **TypeError** exception.
  1. If _O_ does not have a [[Target]] internal slot return.
  1. 1. Let _target_ be the value of _O_'s [[Target]] internal slot.
  1. If Type(_changeType_) is not string, throw a **TypeError** exception.
  1. If IsCallable(_changeFn_) is **false**, throw a **TypeError** exception.
  1. Call BeginChange(_target_, _changeType_).
  1. Let _changeRecord_ be the result of calling the [[Call]] internal method of _changeFn_, with **undefined** as _thisArgument_ and an empty List as _argumentsList_.
  1. Call EndChange(_target_, _changeType_).
  1. ReturnIfAbrubt(_changeRecord_).
  1. Let _changeObservers_ be the value of _O_'s [[ChangeObservers]] internal slot.
  1. If _changeObservers_ is empty, return.
  1. Let _target_ be the value of _O_'s [[Target]] internal slot.
  1. Let _newRecord_ be ObjectCreate(%ObjectPrototype%).
  1. Let _desc_ be the PropertyDescriptor{[[Value]]: _target_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
  1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _newRecord_ passing `"object"` and _desc_ as arguments.
  1. Assert: _success_ is **true**.
  1. Let _desc_ be the PropertyDescriptor{[[Value]]: _changeType_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
  1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _newRecord_ passing `"type"` and _desc_ as arguments.
  1. Assert: _success_ is **true**.
  1. Let _names_ be the result of calling the [[Enumerate]] internal method of _changeRecord_ with no arguments.
  1. Repeat for each element _N_ of _names_ in List order,
    1. If _N_ is not `"object"` and _N_ is not `"type"`, then
      1. Let _value_ be Get(_changeRecord_, _N_).
      1. ReturnIfAbrubt(_value_).
      1. Let _desc_ be the PropertyDescriptor{[[Value]]: _value_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
      1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _newRecord_ passing _N_ and _desc_ as arguments.
      1. Assert: _success_ is **true**.
  1. Set the value of the [[Extensible]] internal slot of _newRecord_ to **false**.
  1. Call EnqueueChangeRecord(_target_, _newRecord_).


### GetNotifier(O)

When the abstract operation GetNotifier is called with Object _O_ the following steps are taken:

  1. Let _notifier_ be the value of _O_'s [[Notifier]] internal slot.
  1. If _notifier_ is **undefined**, then
    1. Let _notifier_ be ObjectCreate(%NotifierPrototype%, ([[Target]], [[ChangeObservers]], [[ActiveChanges]])).
    1. Set _notifier_'s [[Target]] internal slot to _O_.
    1. Set _notifier_'s [[ChangeObservers]] internal slot to a new empty List.
    1. Set _notifier_'s [[ActiveChanges]] internal slot to ObjectCreate(**null**).
    1. Set _O_'s [[Notifier]] internal slot to _notifier_.
  1. return _notifier_.


### BeginChange(O, changeType)

When the abstract operation BeginChange is called with Object _O_ and string _changeType_ the following steps are taken:

  1. Assert: Type(_O_) is Object.
  1. Assert: Type(_changeType_) is String.
  1. Let _notifier_ be GetNotifier(_O_).
  1. Let _activeChanges_ be the value of _notifier_'s [[ActiveChanges]] internal slot.
  1. Let _changeCount_ be Get(_activeChanges_, _changeType_).
  1. ReturnIfAbrubt(_changeCount_).
  1. If _changeCount_ is **undefined**, let _changeCount_ be 1.
  1. Else, let _changeCount_ be _changeCount_ + 1.
  1. Let _success_ be CreateDataProperty(_activeChanges_, _changeType_, _changeCount_).
  1. Assert: _success_ is **true**.



### EndChange(O, changeType)

When the abstract operation EndChange is called with Object _O_ and string _changeType_ the following steps are taken:

  1. Assert: Type(_O_) is Object.
  1. Assert: Type(_changeType_) is String.
  1. Let _notifier_ be GetNotifier(_O_).
  1. Let _activeChanges_ be the value of _notifier_'s [[ActiveChanges]] internal slot.
  1. Let _changeCount_ be Get(_activeChanges_, _changeType_).
  1. ReturnIfAbrubt(_changeCount_).
  1. Assert: _changeCount_ > 0.
  1. Let _changeCount_ be _changeCount_ - 1.
  1. Let _success_ be CreateDataProperty(_activeChanges_, _changeType_, _changeCount_).
  1. Assert: _success_ is **true**.







### ShouldDeliverToObserver(activeChanges, acceptList, changeType)

When the abstract operation ShouldDeliverToObserver is called with Object _activeChanges_, List _acceptList_ and string _changeType_ the following steps are taken:

  1. Let _doesAccept_ be **false**.
  1. For each _accept_ in _acceptList_, do
    1. Let _activeChangeCount_ be Get(_activeChanges_, _accept_).
    1. ReturnIfAbrupt(_activeChangeCount_).
    1. If _activeChangeCount_ > 0, return **false**.
    1. If _accept_ is same string as _changeType_, then
      1. Let _doesAccept_ be **true**  .
  1. return _doesAccept_.




### EnqueueChangeRecord(O, changeRecord)

When the abstract operation EnqueueChangeRecord is called with Object _O_ and change record _changeRecord_ the following steps are taken:

  1. Assert: Type(_O_) is Object.
  1. Let _notifier_ be GetNotifier(_O_).
  1. Let _changeType_ be  Get(_changeRecord_, `"type"`).
  1. ReturnIfAbrupt(_changeType_).
  1. Let _activeChanges_ be the value of _notfier_'s [[ActiveChanges]] internal slot.
  1. Let _changeObservers_ be the value of _notifier_'s [[ChangeObservers]] internal slot.
  1. For each _observerRecord_ in _changeObservers_, do
    1. Let _skip_ be Get(_observerRecord_, `"skip"`).
    1. ReturnIfAbrupt(_skip_).
    1. If _skip_ is **true**, continue.
    1. Let _acceptList_ be Get(_observerRecord_, `"accept"`).
    1. ReturnIfAbrupt(_acceptList_).
    1. Let _deliver_ be ShouldDeliverToObserver(_activeChanges_, _acceptList_, _changeType_).
    1. If _deliver_ is **false**, continue.
    1. Otherwise, let _observer_ be Get(_observerRecord_, `"callback"`).
    1. ReturnIfAbrupt(_observer_).
    1. Let _pendingRecords_ be the value of _observer_'s [[PendingChangeRecords]] internal slot.
    1. Append _changeRecord_ to the end of _pendingRecords_.


### DeliverChangeRecords(C)

When the abstract operation DeliverChangeRecords is called with callback _C_, the following steps are taken:

  1. Let _changeRecords_ be the value of _C_'s [[PendingChangeRecords]] internal slot.
  1. Set _C_'s [[PendingChangeRecords]] internal slot to a new empty List.
  1. Let _array_ be ArrayCreate(0).
  1. Let _n_ be 0.
  1. For each _record_ in _changeRecords_, do:
    1. Let _status_ be the result of CreateDataProperty(_array_, ToString(_n_), _record).
    1. Assert: _status_ is **true**.
    1. Increment _n_ by 1.
  1. If _array_ is empty, return **false**.
  1. Let _status_ be Call(_C_, **undefined**, «_array_»).
  1. _result_ is intentionall ignored here since. An implementation might choose to log abrubt completions here.
  1. Return **true**.

Note: The user facing function ''Object.deliverChangeRecords'' returns **''undefined''** to prevent detection if anything was delivered or not.




### [[DeliverAllChangeRecords]]

There is an abstract [[DeliverAllChangeRecords]] internal algorithm:

?.??.?? [[DeliverAllChangeRecords]]

When the [[DeliverAllChangeRecords]] internal algorithm is called, the following steps are taken:

  - Let _observers_ be the result of getting [[ObserverCallbacks]]
  - Let _anyWorkDone_ be false.
  - For each _observer_ in _observers_, do:
    - Let _result_ be the result of calling [[DeliverChangeRecords]] with _observer_.
    - If _result_ is **''true''**, set _anyWorkDone_ to **''true''**.
  - Return _anyWorkDone_.

Note: It is the intention that the embedder will call this internal algorithm when it is time to deliver the change records.


### CreateAndEnqueueChangeRecord(type, object, name, oldDesc, newDesc)

When the abstract operation CreateChangeRecord is called with string _type_, Object _object_, _name_, Object _oldDesc_ and Object _newDesc_ the following steps are taken:

  1. Let _record_ be CreateChangeRecord(_type_, _object_, _name_, _oldDesc_, _newDes_).
  1. Call EnqueueChangeRecord(_object_, _record_).


### CreateChangeRecord(type, object, name, oldDesc, newDesc)

When the abstract operation CreateChangeRecord is called with string _type_, Object _object_, _name_, Object _oldDesc_ and Object _newDesc_ the following steps are taken:

  1. Let _changeRecord_ be ObjectCreate(%ObjectPrototype%).
  1. Let _desc_ be the PropertyDescriptor{[[Value]]: _type_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
  1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _changeRecord_ passing `"type"` and _desc_ as arguments.
  1. Assert: _success_ is **true**.

  1. Let _desc_ be the PropertyDescriptor{[[Value]]: _object_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
  1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _changeRecord_ passing `"object"` and _desc_ as arguments.
  1. Assert: _success_ is **true**.
  1. If IsPropertyKey(_name_) is true, then
    1. Let _desc_ be the PropertyDescriptor{[[Value]]: _name_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
    1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _changeRecord_ passing `"name"` and _desc_ as arguments.
    1. Assert: _success_ is **true**.
  1. If IsDataDescriptor(_oldDesc_) is **true**, then
    1. If IsDataDescritor(_newDesc_) is **false** or SameValue(_oldDesc_.[[Value]], _newDesc_.[[Value]]) is **false**, then
      1. Let _desc_ be the PropertyDescriptor{[[Value]]: _oldDesc_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}.
      1. Let _success_ be the result of calling the [[DefineOwnProperty]] internal method of _changeRecord_ passing `"oldValue"` and _desc_ as arguments.
      1. Assert: _success_ is **true**.
  1. Set the value of the [[Extensible]] internal slot of _changeRecord_ to **false**.
  1. Return _changeRecord_.


### [[CreateSpliceChangeRecord]]

There is now an abstract operation [[CreateSpliceChangeRecord]]:

?.??.?? [[CreateSpliceChangeRecord]] (object, index, removed, addedCount)

When the abstract operation CreateSpliceChangeRecord is called with the arguments: _object_, _index_, _removed_, and _addedCount_, the following steps are taken:

  - Let _changeRecord_ be the result of the abstraction operation ObjectCreate (15.2).
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"type"''**, Property Descriptor {[[Value]]: "splice", [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"object"''**, Property Descriptor {[[Value]]: _object_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"index"''**, Property Descriptor {[[Value]]: _index_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"removed"''**, Property Descriptor {[[Value]]: _removed_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"addedCount"''**, Property Descriptor {[[Value]]: _addedCount_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Set the value of the [[Extensible]] internal slot of _changeRecord_ to **false**.
  - Return _changeRecord_.
