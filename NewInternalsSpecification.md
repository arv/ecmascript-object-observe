# Object.observe: New Internal Properties, Objects and Algorithms

## [[ObserverCallbacks]]

There is now an ordered list, [[ObserverCallbacks]] which is shared per event queue. It is initially empty.

Note: This list is used to provide a deterministic ordering in which callbacks are called.


## [[PendingChangeRecords]]

Every function now has a [[PendingChangeRecords]] internal property which is an ordered list of ChangeRecords.
It is initially empty.

Note: This list gets populated with change records as the objects that this function is observing are mutated. It gets emptied when the change records are delivered.



### [[Notifier]]

Every object _O_ now has a [[Notifier]] internal property which is initially **''undefined''**.

Note: This gets lazily initialized to a notifier object which is an object with the [[NotifierPrototype]] as its [[Prototype]].



### [[NotifierPrototype]]

The [[NotifierPrototype]] is an ordinary object which is used as the [[Prototype]] of all notifiers returned by ''Object.getNotifier(O)''. It has two properties which are defined below.



### [[NotifierPrototype]].notify

The notify function of the [[NotifierPrototype]] is defined as follows:

Let _notifyFunction_ be a function, which when invoked does the following:
  - Let _changeRecord_ be the first argument to the function.
  - Let _notifier_ be [[This]].
  - If Type(_notifier_) is not Object, throw a **''TypeError''** exception.
  - If _notifier_ does not have an internal property [[Target]] return.
  - Let _type_ be the result of calling the [[Get]] internal method of _changeRecord_ with **''"type"''**.
  - If Type(_type_) is not string, throw a **''TypeError''** exception.
  - Let _target_ be [[Target]] of _notifier_.
  - Let _newRecord_ be the result of the abstract operation ObjectCreate (15.12).
  - Call the [[DefineOwnProperty]] internal method of _newRecord_ with arguments **''"object"''**, the Property Descriptor {[[Value]]: _target_, [[Writable]]: **''false''**, [[Enumerable]]: **''true''**, [[Configurable]]: **''false''**}, and **''true''**.
  - For each enumerable property name _N_ in _changeRecord_,
    - If _N_ is not **''"object"''**, then
      - Let _value_ be the result of calling the [[Get]] internal method of _changeRecord_ with _N_.
      - Call the [[DefineOwnProperty]] internal method of _newRecord_ with arguments _N_, the Property Descriptor {[[Value]]: _value_, [[Writable]]: **''false''**, [[Enumerable]]: **''true''**, [[Configurable]]: **''false''**}, and **''true''**.
  - Set the [[Extensible]] internal property of _newRecord_ to **''false''**.
  - Call the [[EnqueueChangeRecord]], passing _target_ and _newRecord_ as arguments.



### [[NotifierPrototype]].performChange

The performChange function of the [[NotifierPrototype]] is defined as follows:

Let _performChange_ be a function, which when invoked does the following:
  - Let _changeType_ be the first argument to the function.
  - Let _changeFn_ be the second argument to the function.
  - Let _notifier_ be [[This]].
  - If Type(_notifier_) is not Object, throw a **''TypeError''** exception.
  - If _notifier_ does not have an internal property [[Target]], return.
  - Let _target_ be the internal property [[Target]] of _notifier_
  - If Type(_changeType_) is not string, throw a **''TypeError''** exception.
  - If IsCallable(_changeFn_) is not **true**, throw a **''TypeError''** exception.
  - Call the [[BeginChange]] internal method with _target_ and _changeType_.
  - Let _changeRecord_ be the return value of calling the [[Call]] internal method of _changeFn_, passing **undefined** as the _this_ parameter with no arguments. Let _error_ be any thrown exception.
  - Call the [[EndChange]] internal method with _target_ and _changeType_.
  - If _error_ is defined, throw _error_.
  - Let _changeObservers_ be the result of getting the internal property [[ChangeObservers]] of _notifier_.
  - If _changeObservers_ is empty, return.
  - Let _target_ be [[Target]] of _notifier_.
  - Let _newRecord_ be the result of the abstract operation ObjectCreate (15.12).
  - Call the [[DefineOwnProperty]] internal method of _newRecord_ with arguments **''"object"''**, the Property Descriptor {[[Value]]: _target_, [[Writable]]: **''false''**, [[Enumerable]]: **''true''**, [[Configurable]]: **''false''**}, and **''true''**.
  - Call the [[DefineOwnProperty]] internal method of _newRecord_ with arguments **''"type"''**, the Property Descriptor {[[Value]]: _changeType_, [[Writable]]: **''false''**, [[Enumerable]]: **''true''**, [[Configurable]]: **''false''**}, and **''true''**.
  - For each enumerable property name _N_ in _changeRecord_,
    - If _N_ is not **''"object"''** and _N_ is not **''"type"''**, then
      - Let _value_ be the result of calling the [[Get]] internal method of _changeRecord_ with _N_.
      - Call the [[DefineOwnProperty]] internal method of _newRecord_ with arguments _N_, the Property Descriptor {[[Value]]: _value_, [[Writable]]: **''false''**, [[Enumerable]]: **''true''**, [[Configurable]]: **''false''**}, and **''true''**.
  - Set the [[Extensible]] internal property of _newRecord_ to **''false''**.
  - Call the [[EnqueueChangeRecord]], passing _target_ and _newRecord_ as arguments.


### GetNotifier(O)

When the abstract operation GetNotifier is called with Object _O_ the following steps are taken:

  1. Let _notifier_ be the value of _O_'s [[Notifier]] internal slot.
  1. If _notifier_ is **undefined**, then
    1. Let _notifier_ be ObjectCreate(%ObjectPrototype%, ([[Target]], [[ChangeObservers]], [[ActiveChanges]])).
    1. Set the [[Prototype]] of _notifier_ to [[NotifierPrototype]]
    1. Set _notifier_'s [[Target]] internal slot to _O_.
    1. Set _notifier_'s [[ChangeObservers]] internal slot to a new empty list.
    1. Set _notifier_'s [[ActiveChanges]] internal slot to ObjectCreate(**null**).
    1. Set _O_'s [[Notifier]] internal slot to _notifier_.
  1. return _notifier_.


### [[BeginChange]]

There is now a [[BeginChange]] internal algorithm:

?.??.?? [[BeginChange]] (O, changeType)

  - Let _notifier_ be the result of calling [[GetNotifier]] passing _O_.
  - Let _activeChanges_ be [[ActiveChanges]] of _notifier_.
  - Let _changeCount_ be Get(_activeChanges_, _changeType_).
  - If _changeCount_ is **undefined**, let _changeCount_ be 1.
  - Else, let _changeCount_ be _changeCount_ + 1.
  - Perform CreateOwnDataProperty(_activeChanges_, _changeType_, _changeCount_).



### [[EndChange]]

There is now a [[EndChange]] internal algorithm:

?.??.?? [[EndChange]] (O, changeType)

  - Let _notifier_ be the result of calling [[GetNotifier]] passing _O_.
  - Let _activeChanges_ be [[ActiveChanges]] of _notifier_.
  - Let _changeCount_ be Get(_activeChanges_, _changeType_).
  - Assert: _changeCount_ > 0.
  - Let _changeCount_ be _changeCount_ - 1.
  - Perform CreateOwnDataProperty(_activeChanges_, _changeType_, _changeCount_).







### ShouldDeliverToObserver(activeChanges, acceptList, changeType)

When the abstract operation ShouldDeliverToObserver is called with Object _activeChanges_, List _acceptList_ and string _changeType_ the following steps are taken:

  1. Let _doesAccept_ be **false**.
  1. For each _accept_ in _acceptList_, do
    1. Let _activeChangeCount_ be Get(_activeChanges_, _accept_).
    1. If _activeChangeCount_ > 0, return **false**.
    1. If _accept_ is same string as _changeType_, then
      1. Let _doesAccept_ be **true**  .
  1. return _doesAccept_.




### EnqueueChangeRecord(O, changeRecord)

When the abstract operation EnqueueChangeRecord is called with Object _O_ and change record _changeRecord_ the following steps are taken:

  1. Let _notifier_ be GetNotifier(_O_).
  1. Let _changeType_ be  Get(_changeRecord_, `"type"`).
  1. Let _activeChanges_ be the value of _notfier_'s [[ActiveChanges]] internal slot.
  1. Let _changeObservers_ be the value of _notifier_'s [[ChangeObservers]] internal slot.
  1. For each _observerRecord_ in _changeObservers_, do
    1. Let _acceptList_ be Get(_observerRecord_, `"accept"`).
    1. Let _deliver_ be ShouldDeliverToObserver(_activeChanges_, _acceptList_, _changeType_).
    1. If _deliver_ is **false**, continue.
    1. Otherwise, let _observer_ be Get(_observerRecord_, `"callback"`).
    1. Let _pendingRecords_ be the value of _observer_'s [[PendingChangeRecords]] internal slot.
    1. Append _changeRecord_ to the end of _pendingRecords_.


### [[DeliverChangeRecords]]

There is an abstract [[DeliverChangeRecords]` internal algorithm.

?.??.?? [[DeliverChangeRecords]] (C)

When the [[DeliverChangeRecords]] internal algorithm is called with callback _C_, the following steps are taken:

  - Let _changeRecords_ be [[PendingChangeRecords]] of _C_.
  - Clear the [[PendingChangeRecords]] of _C_.
  - Let _array_ be the result of the abstraction operation ArrayCreate (15.4) with argument 0.
  - Let _n_ be 0.
  - For each _record_ in _changeRecords_, do:
    - Call the [[DefineOwnProperty]] internal method of _array_ with arguments ToString(_n_), the PropertyDescriptor {[[Value]]: _record_, [[Writable]]: **true**, [[Enumerable]]: **true**, [[Configurable]]: **true**}, and **false**.
    - Increment _n_ by 1.
  - If _array_ is empty, return false.
  - Call the [[Call]] internal method, (silently ignoring any thrown exception or return value) of _C_, passing undefined as the _this_ parameter, and a single argument, _array_.
  - Return true.

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



### [[CreateChangeRecord]]

There is now an abstract operation [[CreateChangeRecord]]:

?.??.?? [[CreateChangeRecord]] (type, object, name, oldDesc, newDesc)

When the abstract operation CreateChangeRecord is called with the arguments: _type_, _object_, _name_ and _oldDesc_, the following steps are taken:

  - Let _changeRecord_ be the result of the abstraction operation ObjectCreate (15.2).
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"type"''**, Property Descriptor {[[Value]]: _type_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"object"''**, Property Descriptor {[[Value]]: _object_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - If Type(_name_) is string, Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"name"''**, Property Descriptor {[[Value]]: _name_, [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - If IsDataDescriptor(_oldDesc_) is true:
    - If IsDataDescritor(_newDesc_) is false or SameValue(oldDesc.[[Value]], newDesc.[[Value]]) is false
      - Call the [[DefineOwnProperty]] internal method of _changeRecord_ with arguments **''"oldValue"''**, Property Descriptor {[[Value]]: _oldDesc_.`[Value]], [[Writable]]: **false**, [[Enumerable]]: **true**, [[Configurable]]: **false**}, and **false**.
  - Set the [[Extensible]] internal property of _changeRecord_ to **false**.
  - Return _changeRecord_.


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
  - Set the [[Extensible]] internal property of _changeRecord_ to **false**.
  - Return _changeRecord_.
