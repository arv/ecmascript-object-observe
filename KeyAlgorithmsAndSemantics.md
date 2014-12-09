# Object.observe: Key Algorithms and Semantics









## ChangeObserver and PendingChangeRecords

Conceptually, at the heart of the EMCAScript observation mechanism is two structures:

  * [[harmony:observe_internals#changeObservers|ChangeObservers]]: the set of functions which are presently registered as observers of changes which take place to given object.

  * [[harmony:observe_internals#pendingchangerecords|PendingChangeRecords]]: the sequence of change records which have been enqueued for delivery to the an observer, but not yet delivered.



[[harmony:observe_public_api#object.observe|Object.observe]] enters a function into an object's associated set of ChangeObservers (while [[harmony:observe_public_api#object.unobserve|Object.unobserve]] removes it).

When a change takes place to object, a change record [[harmony:observe_overview#enqueuing_changes_to_observers|may]] be appended to the PendingChangeRecords of the functions which are registered in its associated ChangeObservers.

An observer will receive the changes, either [[harmony:observe_overview#end-of-microtask_delivery_of_change_records|asynchronously, at the end of the current turn]], or synchronously, if it calls [[harmony:observe_public_api#object.deliverchangerecords|Object.deliverChangeRecords()]] with itself as an argument. In either case, IFF the function has pending changes, it will be invoked with the contents of its PendingChangeRecords as the only argument.





## Observable changes to objects

The vocabulary of observable changes to objects hews close to the object model and reflective APIs of ECMAScript.
  - When properties are [[harmony:observe_spec_changes#validateandapplypropertydescriptor|added or reconfigured]] on an object, observers may receive change records of type: "add", "reconfigure" respectively.
  - When the [[harmony:observe_spec_changes#validateandapplypropertydescriptor|value of data properties change]], observers may receive "update" change records.
  - When properties are [[harmony:observe_spec_changes#delete|deleted]], observers may receive records of type "delete".
  - When an object's [[harmony:observe_spec_changes#setprototypeof|prototype]] or [[harmony:observe_spec_changes#preventextensions|extensibility]] is changed, observers may receive "setPrototype", or "preventExtensions" change records.
  - When certain [[harmony:observe_spec_changes#changes_to_array_methods|mutations to Arrays]] take place which may affect multiple index properties, observers may receive "splice" change records.
  - The [[harmony:observe_internals#notifierprototype_.notify|notify()]] method of an object's associated [[harmony:observe_internals#notifier|Notifier]] may be used to cause observers to receive custom ("synthetic") change records.

Change records are enqueued to observers via the internal [[harmony:observe_internals#enqueuechangerecord|EnqueueChangeRecord]] algorithm.






## The Notifier object

Every object has an associated [[harmony:observe_internals#Notifier]] object which can be retrieved via [[harmony:observe_public_api#object.getnotifier|Object.getNotifier()]] as long as the object is not frozen. The Notifier has internal properties which reference the object's associated set of observers (ChangeObservers) as well as the set of changes which are currently being performed on the object (ActiveChanges).

The Notifier has public API (notify() and performChange()) which can be used to deliver "synthetic" change records to observers and [[harmony:observe_overview#activechanges_and_higher-level_change_types|describe higher-level changes]].

## Accepted change types

An observer must only receive change records whose type matches one provided in the accept list, which is passed as the (optional) third argument to [[harmony:observe_public_api#object.observe|Object.observe]] (if the argument is not provided, the list defaults to the "intrinsic" set of change types).

An observation is conceptually a tuple: <observed object, observer function, accepted change types>. Object.observe ensures that this tuple is represented as an observerRecord in the associated Notifier's ChangeObservers list.


## ActiveChanges and higher-level change types

The [[harmony:observe_internals#notifierprototype_.performchange|performChange()]] method of an object's associated Notifier provides a mechanism to describe a set of changes as a single (more compact) higher-level change type.

The set of change types being performed on an object is represented in the set of properties of the Notifier's `[[ActiveChanges]]` map whose value is positive. The count of active change types in ActiveChanges is affected by performChange() running [[harmony:observe_internals#beginchange|BeginChange]] before, and [[harmony:observe_internals#endchange|EndChange]] after invoking the provided changeFn function.

## Enqueuing changes to observers

When a change is to be enqueued to an object's associated observers, the internal [[harmony:observe_internals#enqueuechangerecord|EnqueueChangeRecord]] algorithm tests whether each observer should receive the change by invoking [[harmony:observe_internals#shoulddelivertoobserver|ShouldDeliverToObserver]].

ShouldDeliverToObserver returns true IFF an observer accepts the candidate change record's type, but accepts none of the change types currently being performed on the object (as represented in the ActiveChanges).


## End-of-turn Delivery of Change Records

At the end of the current processing turn (or "Microtask"), [[harmony:observe_internals#deliverallchangerecords|DeliverAllChangeRecords]] is invoked, which continuously delivers pending change records to observers until there are none remaining.

The order in which observers are delivered to is maintained in the [[harmony:observe_internals#observercallbacks|ObserverCallbacks]] list. The first time a function is used as an observer by provided it as an argument to Object.observe, it is appended to the end of ObserverCallbacks.

Delivery takes place by repeatedly iterating front-to-back through ObserverCallbacks, [[harmony:observe_internals#deliverchangerecords|delivering]] to any observer who has pending change records, until no observer have pending change records.
