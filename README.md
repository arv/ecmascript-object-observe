# Object.observe
 


## Updates

[Updates](Updates.md)



## Problem
Declarative programming techniques provide leverage and power to developers. The web platform has pioneered many -- not the least of which is HTML. As the practice of application development evolves, it is important that primitives exist in the platform which allow developers to create and evolve robust declarative mechanisms independent of those provided directly by the platform.

A class of declarative mechanisms depends on discovering that ECMAScript values have changed. For example, UI frameworks often want to provide an ability to databind objects in a datamodel to UI elements. Likewise, domain objects, application logic and persistence strategies can often best be described in terms of constraints and relationships on data.

Today, ECMAScript code wishing to observe changes to objects typically either creates objects wrapping the real data or employs dirty-checking strategies for discovering changes. The first approach requires objects being observed to buy into the strategy, making the user model more complex and eroding composability of concerns. The second approach has poor algorithmic behavior, requiring work proportional to the number of objects observed to discover if any change has taken place. Both require increased working sets.



## Goals

The desired characteristics of a solution are:

  * No wrapper or proxy objects needed, providing memory efficiency and object identity 
  * Change notifications on add/delete of a property on an object
  * Change notifications on modifications to property descriptor of properties on an object
  * The ability for an object to manually indicate when an accessor property has changed
  * Efficiently implementable in engines
  * Simple, targeted, extension to current ES
  * Asynchronous notification of changes, but allow synchronous fetching of changes pending delivery


## Solution

Object.observe allows for the direct observation of changes to ECMAScript objects. It allows an observer to receive a time-ordered sequence of change records which describe the set of changes which took place to the set of observed objects.

Changes to objects are directly observable and are described in terms of

  * Changes in the value of data properties
  * The addition, deletion, and reconfiguration of all properties visible via the reflection APIs
  * Changes to the prototype and extensibility of the object itself.

A flexible system is provided via a "notifier" object associated with every observable object, which allows for:

  * Decoupling an object's implementation from its observable API.
  * Allowing an object to efficiently describe multiple lower-level changes as a single higher-level change.

Lastly, Array.observe allows for the efficient description of certain changes which may affect many index-valued properties as single "splice" change record.




## Cross-cutting Concerns and Potential Implementation Challenges

Object.observe is a relatively significant and cross-cutting feature, as it

  * Affects the core object model of ECMAScript
  * Makes previously unobservable behavior observable
  * Requires changes to central algorithms (e.g. mutating the properties of an object)
  * Must not allow for the creation of a side-channel of communication between otherwise isolated object graphs.

## API Usage Examples

[Usage Examples](UsageExamples.md)



## Key Algorithms and Semantics

[Key Algorithms and Semantics](KeyAlgorithmsAndSemantics.md)

## New Internal Properties, Objects and Algorithms

[New internals specification](NewInternalsSpecification.md)

## New Public API

[Public API Specification](PublicApiSpecification.md)



## Modifications to Existing Internal Algorithms

[Changes to the existing specification][ChangesToTtheExistingSpecification.md]
