# Object.observe: New Internal Properties, Objects and Algorithms

## [[ObserverCallbacks]]

There is now an ordered list, %%[[ObserverCallbacks]]%% which is shared per event queue. It is initially empty.

Note: This list is used to provide a deterministic ordering in which callbacks are called.


## [[PendingChangeRecords]]

Every function now has a %%[[PendingChangeRecords]]%% internal property which is an ordered list of ChangeRecords.
It is initially empty.

Note: This list gets populated with change records as the objects that this function is observing are mutated. It gets emptied when the change records are delivered.



### [[Notifier]]

Every object //O// now has a %%[[Notifier]]%% internal property which is initially **''undefined''**.

Note: This gets lazily initialized to a notifier object which is an object with the %%[[NotifierPrototype]]%% as its %%[[Prototype]]%%.



### [[NotifierPrototype]]

The %%[[NotifierPrototype]]%% is an ordinary object which is used as the %%[[Prototype]]%% of all notifiers returned by ''Object.getNotifier(O)''. It has two properties which are defined below.



### [[NotifierPrototype]].notify

The notify function of the %%[[NotifierPrototype]]%% is defined as follows:

Let //notifyFunction// be a function, which when invoked does the following:
  - Let //changeRecord// be the first argument to the function.
  - Let //notifier// be %%[[This]]%%.
  - If Type(//notifier//) is not Object, throw a **''TypeError''** exception.
  - If //notifier// does not have an internal property %%[[Target]]%% return.
  - Let //type// be the result of calling the %%[[Get]]%% internal method of //changeRecord// with **''"type"''**.
  - If Type(//type//) is not string, throw a **''TypeError''** exception.
  - Let //target// be %%[[Target]]%% of //notifier//.
  - Let //newRecord// be the result of the abstract operation ObjectCreate (15.12).
  - Call the %%[[DefineOwnProperty]]%% internal method of //newRecord// with arguments **''"object"''**, the Property Descriptor {%%[[Value]]%%: //target//, %%[[Writable]]%%: **''false''**, %%[[Enumerable]]%%: **''true''**, %%[[Configurable]]%%: **''false''**}, and **''true''**.
  - For each enumerable property name //N// in //changeRecord//,
    - If //N// is not **''"object"''**, then
      - Let //value// be the result of calling the %%[[Get]]%% internal method of //changeRecord// with //N//.
      - Call the %%[[DefineOwnProperty]]%% internal method of //newRecord// with arguments //N//, the Property Descriptor {%%[[Value]]%%: //value//, %%[[Writable]]%%: **''false''**, %%[[Enumerable]]%%: **''true''**, %%[[Configurable]]%%: **''false''**}, and **''true''**.
  - Set the %%[[Extensible]]%% internal property of //newRecord// to **''false''**.
  - Call the %%[[EnqueueChangeRecord]]%%, passing //target// and //newRecord// as arguments.



### [[NotifierPrototype]].performChange

The performChange function of the %%[[NotifierPrototype]]%% is defined as follows:

Let //performChange// be a function, which when invoked does the following:
  - Let //changeType// be the first argument to the function.
  - Let //changeFn// be the second argument to the function.
  - Let //notifier// be %%[[This]]%%.
  - If Type(//notifier//) is not Object, throw a **''TypeError''** exception.
  - If //notifier// does not have an internal property %%[[Target]]%%, return.
  - Let //target// be the internal property %%[[Target]]%% of //notifier//
  - If Type(//changeType//) is not string, throw a **''TypeError''** exception.
  - If IsCallable(//changeFn//) is not **true**, throw a **''TypeError''** exception.
  - Call the %%[[BeginChange]]%% internal method with //target// and //changeType//.
  - Let //changeRecord// be the return value of calling the %%[[Call]]%% internal method of //changeFn//, passing **undefined** as the //this// parameter with no arguments. Let //error// be any thrown exception.
  - Call the %%[[EndChange]]%% internal method with //target// and //changeType//.
  - If //error// is defined, throw //error//.
  - Let //changeObservers// be the result of getting the internal property %%[[ChangeObservers]]%% of //notifier//.
  - If //changeObservers// is empty, return.
  - Let //target// be %%[[Target]]%% of //notifier//.
  - Let //newRecord// be the result of the abstract operation ObjectCreate (15.12).
  - Call the %%[[DefineOwnProperty]]%% internal method of //newRecord// with arguments **''"object"''**, the Property Descriptor {%%[[Value]]%%: //target//, %%[[Writable]]%%: **''false''**, %%[[Enumerable]]%%: **''true''**, %%[[Configurable]]%%: **''false''**}, and **''true''**.
  - Call the %%[[DefineOwnProperty]]%% internal method of //newRecord// with arguments **''"type"''**, the Property Descriptor {%%[[Value]]%%: //changeType//, %%[[Writable]]%%: **''false''**, %%[[Enumerable]]%%: **''true''**, %%[[Configurable]]%%: **''false''**}, and **''true''**.
  - For each enumerable property name //N// in //changeRecord//,
    - If //N// is not **''"object"''** and //N// is not **''"type"''**, then
      - Let //value// be the result of calling the %%[[Get]]%% internal method of //changeRecord// with //N//.
      - Call the %%[[DefineOwnProperty]]%% internal method of //newRecord// with arguments //N//, the Property Descriptor {%%[[Value]]%%: //value//, %%[[Writable]]%%: **''false''**, %%[[Enumerable]]%%: **''true''**, %%[[Configurable]]%%: **''false''**}, and **''true''**.
  - Set the %%[[Extensible]]%% internal property of //newRecord// to **''false''**.
  - Call the %%[[EnqueueChangeRecord]]%%, passing //target// and //newRecord// as arguments.


### [[GetNotifier]]

There is now a %%[[GetNotifier]]%% internal algorithm:

?.??.?? %%[[GetNotifier]]%% (O)

  - Let //notifier// be %%[[Notifier]]%% of //O//.
  - If //notifier// is **undefined**
    - Let //notifier// be the result the abstract operation ObjectCreate (15.12).
    - Set the %%[[Prototype]]%% of //notifier// to %%[[NotifierPrototype]]%%.
    - Set the internal property %%[[Target]]%% of //notifier// to be //O//.
    - Set the internal property %%[[ChangeObservers]]%% of //notifier// to be a new empty list.
    - Set the internal property %%[[ActiveChanges]]%% of //notifier// to be the result of ObjectCreate(null).
    - Set %%[[Notifier]]%% of //O// to //notifier//.
  - return //notifier//.


### [[BeginChange]]

There is now a %%[[BeginChange]]%% internal algorithm:

?.??.?? %%[[BeginChange]]%% (O, changeType)

  - Let //notifier// be the result of calling %%[[GetNotifier]]%% passing //O//.
  - Let //activeChanges// be %%[[ActiveChanges]]%% of //notifier//.
  - Let //changeCount// be Get(//activeChanges//, //changeType//).
  - If //changeCount// is **undefined**, let //changeCount// be 1.
  - Else, let //changeCount// be //changeCount// + 1.
  - Perform CreateOwnDataProperty(//activeChanges//, //changeType//, //changeCount//).



### [[EndChange]]

There is now a %%[[EndChange]]%% internal algorithm:

?.??.?? %%[[EndChange]]%% (O, changeType)

  - Let //notifier// be the result of calling %%[[GetNotifier]]%% passing //O//.
  - Let //activeChanges// be %%[[ActiveChanges]]%% of //notifier//.
  - Let //changeCount// be Get(//activeChanges//, //changeType//).
  - Assert: //changeCount// > 0.
  - Let //changeCount// be //changeCount// - 1.
  - Perform CreateOwnDataProperty(//activeChanges//, //changeType//, //changeCount//).







### [[ShouldDeliverToObserver]]

There is now an abstract %%[[ShouldDeliverToObserver]]%% internal algorithm:

?.??.?? %%[[ShouldDeliverToObserver]]%% (activeChanges, acceptList, changeType)

When the %%[[ShouldDeliverToObserver]]%% internal algorithm is called, the following steps are taken:

  - Let //doesAccept// be **false**
  - For each //accept// in //acceptList//
    - If //activeChanges//[//accept//] > 0, return **false**
    - If //accept// == //changeType//
      - Set //doesAccept// to **true**  
  - return //doesAccept//.




### [[EnqueueChangeRecord]]

There is now an abstract %%[[EnqueueChangeRecord]]%% internal algorithm:

?.??.?? %%[[EnqueueChangeRecord]]%% (O, changeRecord)

When the %%[[EnqueueChangeRecord]]%% internal algorithm is called, the following steps are taken:

  - Let //notifier// be the result of calling %%[[GetNotifier]]%%, passing //O//.
  - Let //changeType// be the result of calling Get(//changeRecord//, "type").
  - Let //activeChanges// be %%[[ActiveChanges]]%% of //notifier//.
  - Let //changeObservers// be %%[[ChangeObservers]]%% of //notifier//.
  - For each //observerRecord// in //changeObservers//:
    - Let //acceptList// be the result of calling Get(//observerRecord//, “accept”).
    - Let //deliver// be the result of calling %%[[ShouldDeliverToObserver]]%% with //activeChanges//, //acceptList// and //changeType//.
    - If //deliver// is **false**, continue.
    - Otherwise, Let //observer// be Get(//observerRecord//, "callback").
    - Let //pendingRecords// be the result of getting %%[[PendingChangeRecords]]%% for //observer//.
    - Append //changeRecord// to the end of //pendingRecords//.

### [[DeliverChangeRecords]]

There is an abstract %%[[DeliverChangeRecords]%% internal algorithm.

?.??.?? %%[[DeliverChangeRecords]]%% (C)

When the %%[[DeliverChangeRecords]]%% internal algorithm is called with callback //C//, the following steps are taken:

  - Let //changeRecords// be %%[[PendingChangeRecords]]%% of //C//.
  - Clear the %%[[PendingChangeRecords]]%% of //C//.
  - Let //array// be the result of the abstraction operation ArrayCreate (15.4) with argument 0.
  - Let //n// be 0.
  - For each //record// in //changeRecords//, do:
    - Call the %%[[DefineOwnProperty]]%% internal method of //array// with arguments ToString(//n//), the PropertyDescriptor {%%[[Value]]%%: //record//, %%[[Writable]]%%: **true**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **true**}, and **false**.
    - Increment //n// by 1.
  - If //array// is empty, return false.
  - Call the %%[[Call]]%% internal method, (silently ignoring any thrown exception or return value) of //C//, passing undefined as the //this// parameter, and a single argument, //array//.
  - Return true.

Note: The user facing function ''Object.deliverChangeRecords'' returns **''undefined''** to prevent detection if anything was delivered or not.




### [[DeliverAllChangeRecords]]

There is an abstract %%[[DeliverAllChangeRecords]]%% internal algorithm:

?.??.?? %%[[DeliverAllChangeRecords]]%%

When the %%[[DeliverAllChangeRecords]]%% internal algorithm is called, the following steps are taken:

  - Let //observers// be the result of getting %%[[ObserverCallbacks]]%%
  - Let //anyWorkDone// be false.
  - For each //observer// in //observers//, do:
    - Let //result// be the result of calling %%[[DeliverChangeRecords]]%% with //observer//.
    - If //result// is **''true''**, set //anyWorkDone// to **''true''**.
  - Return //anyWorkDone//.

Note: It is the intention that the embedder will call this internal algorithm when it is time to deliver the change records.



### [[CreateChangeRecord]]

There is now an abstract operation %%[[CreateChangeRecord]]%%:

?.??.?? %%[[CreateChangeRecord]]%% (type, object, name, oldDesc, newDesc)

When the abstract operation CreateChangeRecord is called with the arguments: //type//, //object//, //name// and //oldDesc//, the following steps are taken:

  - Let //changeRecord// be the result of the abstraction operation ObjectCreate (15.2).
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"type"''**, Property Descriptor {%%[[Value]]%%: //type//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"object"''**, Property Descriptor {%%[[Value]]%%: //object//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - If Type(//name//) is string, Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"name"''**, Property Descriptor {%%[[Value]]%%: //name//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - If IsDataDescriptor(//oldDesc//) is true:
    - If IsDataDescritor(//newDesc//) is false or SameValue(oldDesc.%%[[Value]]%%, newDesc.%%[[Value]]%%) is false
      - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"oldValue"''**, Property Descriptor {%%[[Value]]%%: //oldDesc//.%%[Value]]%%, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Set the %%[[Extensible]]%% internal property of //changeRecord// to **false**.
  - Return //changeRecord//.


### [[CreateSpliceChangeRecord]]

There is now an abstract operation %%[[CreateSpliceChangeRecord]]%%:

?.??.?? %%[[CreateSpliceChangeRecord]]%% (object, index, removed, addedCount)

When the abstract operation CreateSpliceChangeRecord is called with the arguments: //object//, //index//, //removed//, and //addedCount//, the following steps are taken:

  - Let //changeRecord// be the result of the abstraction operation ObjectCreate (15.2).
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"type"''**, Property Descriptor {%%[[Value]]%%: "splice", %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"object"''**, Property Descriptor {%%[[Value]]%%: //object//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"index"''**, Property Descriptor {%%[[Value]]%%: //index//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"removed"''**, Property Descriptor {%%[[Value]]%%: //removed//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Call the %%[[DefineOwnProperty]]%% internal method of //changeRecord// with arguments **''"addedCount"''**, Property Descriptor {%%[[Value]]%%: //addedCount//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.
  - Set the %%[[Extensible]]%% internal property of //changeRecord// to **false**.
  - Return //changeRecord//.
