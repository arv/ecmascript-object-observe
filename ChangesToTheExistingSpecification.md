# Object.observe: Modifications to existing internal algorithms




### [[ValidateAndApplyPropertyDescriptor]]

Modify the ValidateAndApplyPropertyDescriptor  algorithm as __indicated__:

9.1.6.3 ValidateAndApplyPropertyDescriptor (O, P, extensible, Desc, current)

When the abstract operation ValidateAndApplyPropertyDescriptor is called with Object O, property key P, Boolean value extensible, and property descriptors Desc, and current the following steps are taken:

This algorithm contains steps that test various fields of the Property Descriptor Desc for specific values. The fields that are tested in this manner need not actually exist in Desc. If a field is absent then its value is considered to be false.

NOTE: If undefined is passed as the O argument only validation is performed and no object updates are performed.


  - Assert: If O is not undefined then P is a valid property key.
  - __Let changeType be a string, initially set to “reconfigure”.__
  - If current is undefined, then
    - If extensible is false, then return false.
    - Assert: extensible is true.
    - If IsGenericDescriptor(Desc) or IsDataDescriptor(Desc) is true, then
      - If O is not undefined, then create an own data property named P of object O whose %%[[Value]]%%, %%[[Writable]]%%, %%[[Enumerable]]%% and %%[[Configurable]]%% attribute values are described by Desc. If the value of an attribute field of Desc is absent, the attribute of the newly created property is set to its default value.
    - Else Desc must be an accessor Property Descriptor,
      - If O is not undefined, then create an own accessor property named P of object O whose %%[[Get]]%%, %%[[Set]]%%, %%[[Enumerable]]%% and %%[[Configurable]]%% attribute values are described by Desc. If the value of an attribute field of Desc is absent, the attribute of the newly created property is set to its default value.
    - __Let R be the result of calling %%[[CreateChangeRecord]]%% with arguments: "add", O, P, current and Desc.__
    - __Call %%[[EnqueueChangeRecord]]%% passing O and R__
    - Return true.
  - Return true, if every field in Desc is absent.
  - Return true, if every field in Desc also occurs in current and the value of every field in Desc is the same value as the corresponding field in current when compared using the SameValue algorithm.
  - If the %%[[Configurable]]%% field of current is false then
    - Return false, if the %%[[Configurable]]%% field of Desc is true.
    - Return false, if the %%[[Enumerable]]%% field of Desc is present and the %%[[Enumerable]]%% fields of current and Desc are the Boolean negation of each other.
  - If IsGenericDescriptor(Desc) is true, then no further validation is required.
  - Else if IsDataDescriptor(current) and IsDataDescriptor(Desc) have different results, then
    - Return false, if the %%[[Configurable]]%% field of current is false.
    - If IsDataDescriptor(current) is true, then
      - If O is not undefined, then convert the property named P of object O from a data property to an accessor property. Preserve the existing values of the converted property’s %%[[Configurable]]%% and %%[[Enumerable]]%% attributes and set the rest of the property’s attributes to their default values.
    - Else,
      - If O is not undefined, then convert the property named P of object O from an accessor property to a data property. Preserve the existing values of the converted property’s %%[[Configurable]]%% and %%[[Enumerable]]%% attributes and set the rest of the property’s attributes to their default values.
  - Else if IsDataDescriptor(current) and IsDataDescriptor(Desc) are both true, then
    - If the %%[[Configurable]]%% field of current is false, then
      - Return false, if the %%[[Writable]]%% field of current is false and the %%[[Writable]]%% field of Desc is true.
      - If the %%[[Writable]]%% field of current is false, then
        - Return false, if the %%[[Value]]%% field of Desc is present and SameValue(Desc.%%[[Value]]%%, current.%%[[Value]]%%) is false.
    - else the %%[[Configurable]]%% field of current is true, so any change is acceptable.
    - __If %%[[Configurable]]%% is not in Desc or SameValue(Desc.%%[[Configurable]]%%, current.%%[[Configurable]]%%) is true and %%[[Enumerable]]%% is not in Desc or SameValue(Desc.%%[[Enumerable]]%%,current.%%[[Enumerable]]%%) is true and %%[[Writable]]%% is not  in Desc or SameValue(Desc.%%[[Writable]]%%, current.%%[[Writable]]%%) is true, then__
      - __Set changeType to be the string "update".__
  - Else IsAccessorDescriptor(current) and IsAccessorDescriptor(Desc) are both true,
    - If the %%[[Configurable]]%% field of current is false, then
      - Return false, if the %%[[Set]]%% field of Desc is present and SameValue(Desc.%%[[Set]]%%, current.%%[[Set]]%%) is false.
      - Return false, if the %%[[Get]]%% field of Desc is present and SameValue(Desc.%%[[Get]]%%, current.%%[[Get]]%%) is false.
  - If O is not undefined, then
    - For each attribute field of Desc that is present, set the correspondingly named attribute of the property named P of object O to the value of the field.
  - __Let R be the result of calling %%[[CreateChangeRecord]]%% with arguments: changeType, O, P, current and Desc.__
  - __Call %%[[EnqueueChangeRecord]]%% passing O and R__
  - Return true.





### [[Delete]]

Modify the %%[[Delete]]%% algorithm as __indicated__:

9.1.10 %%[[Delete]]%% (P)

When the %%[[Delete]]%% internal method of //O// is called with property key //P// the following steps are taken:

  - Assert: IsPropertyKey(P) is true.
  - Let //desc// be the result of calling the %%[GetOwnProperty]]%% internal method of //O// with argument //P//.
  - If //desc// is undefined, then return true.
  - __Let //notifier// be the result of calling %%[[GetNotifier]]%%, passing //O//.__
  - If desc.%%[[Configurable]]%% is true, then
    - Remove the own property with name //P// from //O//.
    - __Let //R// be the result of calling %%[[CreateChangeRecord]]%% with arguments: “delete”, //O//, //P// and //desc//.__
    - __Call %%[[EnqueueChangeRecord]]%% passing //O// and //R//.__
    - Return true.
  - Return false.









### [[PreventExtensions]]

9.1.4 %%[[PreventExtensions]]%% ()

When the %%[[PreventExtensions]]%% internal method of //O// is called the following steps are taken:

  - __Let //wasExtensible// be the value of the %%[[Extensible]]%% internal data property of //O//.__
  - Set the value of the %%[[Extensible]]%% internal data property of O to false.
  - If //wasExtensible// is false, return true.
  - __Let //notifier// be the result of calling %%[[GetNotifier]]%%, passing //O//.__
  - __Let //R// be the result of calling %%[[CreateChangeRecord]]%% with arguments: “preventExtensions”, //O//.__
  - __Call %%[[EnqueueChangeRecord]]%% passing //O// and //R//.__
  - Return true.


### [[SetPrototypeOf]]

9.1.2 %%[[SetPrototypeOf]]%% (V)

When the %%[[SetPrototypeOf]]%% internal method of //O// is called with argument //V// the following steps are taken:

  - Assert: Either Type(//V//) is Object or Type(//V//) is Null.
  - Let //extensible// be the value of the %%[[Extensible]]%% internal data property of //O//.
  - Let //current// be the value of the %%[[Prototype]]%% internal data property of //O//.
  - If SameValue(//V//, //current//), then return true.
  - If //extensible// is false, then return false.
  - If //V// is not null, then
    - Let //p// be //V//.
    - Repeat, while //p// is not null
      - If SameValue(//p//, //O//) is true, then return false.
      - Let //nextp// be the result of calling the %%[[GetPrototypeOf]]%% internal method of //p// with no arguments.
      - ReturnIfAbrupt(//nextp//).
      - Let //p// be //nextp//.
  - Set the value of the %%[[Prototype]]%% internal data property of //O// to //V//.
  - __Let //notifier// be the result of calling %%[[GetNotifier]]%%, passing //O//.__
  - __Let //R// be the result of the abstraction operation ObjectCreate (15.2).__
  - __Call the %%[[DefineOwnProperty]]%% internal method of //R// with arguments **''"type"''**, Property Descriptor {%%[[Value]]%%: **''"setPrototype"''**, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.__
  - __Call the %%[[DefineOwnProperty]]%% internal method of //R// with arguments **''"object"''**, Property Descriptor {%%[[Value]]%%: //O//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.__
  - __Call the %%[[DefineOwnProperty]]%% internal method of //R// with arguments **''"oldValue"''**, Property Descriptor {%%[[Value]]%%: //current//, %%[[Writable]]%%: **false**, %%[[Enumerable]]%%: **true**, %%[[Configurable]]%%: **false**}, and **false**.__
  - __Set the %%[[Extensible]]%% internal property of //R// to false.__
  - __Call %%[[EnqueueChangeRecord]]%% on passing //O// and //R//.__
  - Return true.


## Changes to Array methods





### Array.prototype.pop

22.1.3.16 Array.prototype.pop ()

The last element of the array is removed from the array and returned.

  - Let O be the result of calling ToObject passing the this value as the argument.
  - ReturnIfAbrupt(O).
  - Let lenVal be the result of Get(O, "length").
  - Let len be ToUint32(lenVal).
  - ReturnIfAbrupt(len).
  - If len is zero, 
    - Let putStatus be the result of Put(O, "length", 0, true).
    - ReturnIfAbrupt(putStatus).
    - Return undefined.
  - Else, len > 0
    - Let newLen be len–1.
    - Let indx be ToString(newLen).
    - __Call the %%[[BeginChange]]%% internal passing O and "splice".__
    - Let element be the result of Get(O, indx).
    - ReturnIfAbrupt(element).
    - Let deleteStatus be the result of DeletePropertyOrThrow(O, indx).
    - __If deleteStatus.%%[[type]]%% is normal, let putStatus be the result of Put(O, "length", newLen, true).__
    - __Call the %%[[EndChange]]%% internal passing O and "splice".__
    - __Let elements be a new List.__
    - __Append element to the end of elements.__
    - __Let removed be the result of CreateArrayFromList(elements).__
    - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: O, newLen, removed and 0.__
    - __Call %%[[EnqueueChangeRecord]]%% passing O and R__
    - ReturnIfAbrupt(deleteStatus)
    - ReturnIfAbrupt(putStatus)
    - Return element.





### Array.prototype.push

22.1.3.17 Array.prototype.push ( [ item1 [ , item2 [ , … ] ] ] )

The arguments are appended to the end of the array, in the order in which they appear. The new length of the  array is returned as the result of the call.

When the push method is called with zero or more arguments item1, item2, etc., the following steps are taken:

  - Let O be the result of calling ToObject passing the this value as the argument.
  - ReturnIfAbrupt(O).
  - Let lenVal be the result of Get(O, "length").
  - Let n be ToLength(lenVal).
  - ReturnIfAbrupt(n).
  - __Let putStatus be n.__
  - Let items be an internal List whose elements are, in left to right order, the arguments that were passed to this function invocation.
  - __Let //numAdded// be the length of the //items// list.__
  - __Call the %%[[BeginChange]]%% internal method passing O and "splice".__
  - __Repeat, while items is not empty and status.%%[[type]]%% is normal__
    - Remove the first element from items and let E be the value of the element.
    - Let putStatus be the result of Put(O, ToString(n), E, true).
    - __If putStatus.%%[[type]]%% is normal, increase n by 1.__
  - __If putStatus.%%[[type]]%% is normal, let putStatus be the result of Put(O, "length", n, true).__
  - __Call the %%[[EndChange]]%% internal method passing O and "splice".__
  - __Let removed be the result of ArrayCreate(0).__
  - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: O, lenVal, removed and numAdded.__
  - __Call %%[[EnqueueChangeRecord]]%% passing O and R__
  - ReturnIfAbrupt(putStatus)
  - Return n.




### Array.prototype.shift

22.1.3.21 Array.prototype.shift ( )

The first element of the array is removed from the array and returned.

  - Let O be the result of calling ToObject passing the this value as the argument.
  - ReturnIfAbrupt(O).
  - Let lenVal be the result of Get(O, "length").
  - Let len be ToLength(lenVal).
  - ReturnIfAbrupt(len).
  - If len is zero, then
    - Let putStatus be the result of Put(O, "length", 0, true).
    - ReturnIfAbrupt(putStatus).
    - Return undefined.
  - __Call the %%[[BeginChange]]%% internal method passing O and "splice".__
  - Let first be the result of Get(O, "0").
  - ReturnIfAbrupt(first).
  - __Let status be first__
  - Let k be 1.
  - __Repeat, while status.%%[[type]]%% is normal and k < len__
    - Let from be ToString(k).
    - Let to be ToString(k–1).
    - Let fromPresent be the result of HasProperty(O, from).
    - Let status be fromPresent.
    - __If status.%%[[type]]%% is normal__
      - If fromPresent is true, then
        - Let fromVal be the result of Get(O, from).
        - __Let status be fromVal__.
        - __If status.%%[[type]]%% is normal, let putStatus be the result of Put(O, to, fromVal, true).__
      - Else, fromPresent is false
        - Let status be the result of DeletePropertyOrThrow(O, to).
      - Increase k by 1.
  - __If status.%%[[type]]%% is normal, let status be the result of DeletePropertyOrThrow(O, ToString(len–1)).__
  - __If status.%%[[type]]%% is normal, let status be the result of Put(O, "length", len–1, true).__
  - __Call the %%[[EndChange]]%% internal method passing O and "splice".__
  - __Let elements be a new List.__
  - __Append first to the end of elements.__
  - __Let removed be the result of CreateArrayFromList(elements).__
  - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: O, 0, removed and 0.__
  - __Call %%[[EnqueueChangeRecord]]%% passing O and R.__
  - ReturnIfAbrupt(status).
  - Return first.









### Array.prototype.splice

22.1.3.25 Array.prototype.splice (start, deleteCount [ , item1 [ , item2 [ , … ] ] ] )

When the splice method is called with two or more arguments start, deleteCount and (optionally) item1, item2, etc., the deleteCount elements of the array starting at array index start are replaced by the arguments item1, item2, etc. An Array object containing the deleted elements (if any) is returned. The following steps are taken:

  - Let O be the result of calling ToObject passing the this value as the argument.
  - ReturnIfAbrupt(O).
  - Let lenVal be the result of Get(O, "length")
  - Let len be ToLength(lenVal).
  - ReturnIfAbrupt(len).
  - Let relativeStart be ToInteger(start).
  - ReturnIfAbrupt(relativeStart).
  - If relativeStart is negative, let actualStart be max( (len + relativeStart), 0); else let actualStart be min(relativeStart, len).
  - If start is not present, then
    - Let actualDeleteCount be 0.
  - Else if deleteCount is not present, then
    - Let actualDeleteCount be len - actualStart
  - Else,
    - Let dc be ToInteger(deleteCount).
    - ReturnIfAbrupt(dc).
    - Let actualDeleteCount be min(max(dc, 0), len - actualStart).
  - Let A be undefined.
  - If O is an exotic Array object, then
    - Let C be the result of Get(O, "constructor").
    - ReturnIfAbrupt(C).
    - If IsConstructor(C) is true, then
      - Let thisRealm be the running execution context's Realm.
      - If thisRealm and the value of C's %%[[Realm]]%% internal slot are the same value, then
        - Let A be the result of calling the %%[[Construct]]%% internal method of C with argument (actualDeleteCount).
  - If A is undefined, then
    - Let A be the result of the abstract operation ArrayCreate with the argument actualDeleteCount.
  - ReturnIfAbrupt(A).
  - Let k be 0.
  - __Call the %%[[BeginChange]]%% internal method passing O and "splice".__
  - __Let deletedItems be a new List.__
  - __Let status be relativeStart.__
  - __Repeat, while status.%%[[type]]%% is normal and k < actualDeleteCount__
    - Let from be ToString(actualStart+k).
    - Let fromPresent be the result of HasProperty(O, from).
    - __Let status be fromPresent.__
    - __If status.%%[[type]]%% is normal, then__
      - If fromPresent is true, then
        - Let fromValue be the result of Get(O, from).
        - __Let status be fromValue.__
        - __If status.%%[[type]]%% is normal, then__
          - __Let status be the result of CreateDataPropertyOrThrow(A, ToString(k), fromValue).__
          - __Append fromValue to the end of deletedItems.__
       - Increment k by 1.
  - __If status.%%[[type]]%% is not throw, let status be the result of Put(A, "length", actualDeleteCount, true).__
  - __If status.%%[[type]]%% is not throw, then__
    - Let items be an internal List whose elements are, in left to right order, the portion of the actual argument list starting with item1. The list will be empty if no such items are present.
    - Let itemCount be the number of elements in items.
  - If status.%%[[type]]%% is not throw and itemCount < actualDeleteCount, then
    - Let k be actualStart.
    - __Repeat, while status.%%[[type]]%% is normal and k < (len – actualDeleteCount)__
      - Let from be ToString(k+actualDeleteCount).
      - Let to be ToString(k+itemCount).
      - Let fromPresent be the result of HasProperty(O, from).
      - __Let status be fromPresent.__
      - __If status.%%[[type]]%% is normal, then__
        - If fromPresent is true, then
          - Let fromValue be the result of Get(O, from).
          - __Let status be fromValue.__
          - __If status.%%[[type]]%% is normal, let status be the result of Put(O, to, fromValue, true).__
        - Else, fromPresent is false
          - Let status be the result of DeletePropertyOrThrow(O, to).
        - __If status.%%[[type]]%% is normal, Increase k by 1.__
    - __If status.%%[[type]]%% is normal, Let k be len.__
    - __Repeat, while status.%%[[type]]%% is normal and k > (len – actualDeleteCount + itemCount)__
      - Let status be the result of DeletePropertyOrThrow(O, ToString(k–1)).
      - __If status.%%[[type]]%% is normal, decrease k by 1.__
  - __Else if status.%%[[type]]%% is normal and itemCount > actualDeleteCount, then__
    - Let k be (len – actualDeleteCount).
    - __Repeat, while status.%%[[type]]%% is normal and k > actualStart__
      - Let from be ToString(k + actualDeleteCount – 1).
      - Let to be ToString(k + itemCount – 1)
      - Let fromPresent be the result of HasProperty(O, from).
      - __Let status be fromPresent.__
      - __If status.%%[[type]]%% is normal, then__
        - If fromPresent is true, then
          - Let fromValue be the result of Get(O, from).
          - __Let status be fromValue.__
          - __If status.%%[[type]]%% is normal, let status be the result of Put(O, to, fromValue, true).__
        - Else fromPresent is false,
          - Let status be the result of DeletePropertyOrThrow(O, to).
        - __If status.%%[[type]]%% is normal, decrease k by 1.__
  - __If status.%%[[type]]%% is normal, then__
    - Let k be actualStart.
    - __Repeat, while status.%%[[type]]%% is normal and items is not empty__
      - Remove the first element from items and let E be the value of that element.
      - Let status be the result of Put(O, ToString(k), E, true).
      - __If status.%%[[type]]%% is normal, increase k by 1.__
    - Let status be the result of Put(O, "length", len – actualDeleteCount + itemCount, true).
  - __Call the %%[[EndChange]]%% internal method passing O and "splice".__
  - __Let  removed be CreateArrayFromList(deletedItems).__
  - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: O, actualStart, removed and itemCount.__
  - __Call %%[[EnqueueChangeRecord]]%% passing O and R__
  - ReturnIfAbrupt(status)
  - Return A.


### Array.prototype.unshift

22.1.3.28 Array.prototype.unshift ( [ item1 [ , item2 [ , … ] ] ] )

The arguments are prepended to the start of the array, such that their order within the array is the same as the order in which they appear in the argument list.

When the unshift method is called with zero or more arguments item1, item2, etc., the following steps are taken:

  - Let O be the result of calling ToObject passing the this value as the argument.
  - ReturnIfAbrupt(O).
  - Let lenVal be the result of Get(O, "length")
  - Let len be ToLength(lenVal).
  - ReturnIfAbrupt(len).
  - Let argCount be the number of actual arguments.
  - Let k be len.
  - __Let status be len.__
  - __Call the %%[[BeginChange]]%% internal method passing O and "splice".__
  - __Repeat, while status.%%[[type]]%% is normal and k > 0,__
    - Let from be ToString(k–1).
    - Let to be ToString(k+argCount –1).
    - Let fromPresent be the result of HasProperty(O, from).
    - __Let status be fromPresent.__
    - __If status.%%[[type]]%% is normal, then__
      - If fromPresent is true, then
        - Let fromValue be the result of Get(O, from).
        - __Let status be fromValue.__
        - __If status.%%[[type]]%% is normal, let putStatus be the result of Put(O, to, fromValue, true).__
        - __Let status be putStatus.__
      - Else, fromPresent is false
        - Let deleteStatus be the result of DeletePropertyOrThrow(O, to).
        - __Let status be deleteStatus.__
      - If status.%%[[type]]%% is not throw, decrease k by 1.
  - __If status.%%[[type]]%% is normal, then__
    - Let j be 0.
    - Let items be a List whose elements are, in left to right order, the arguments that were passed to this function invocation.
  - __Repeat, while status.%%[[type]]%% is normal and items is not empty__
    - Remove the first element from items and let E be the value of that element.
    - Let status be the result of Put(O, ToString(j), E, true).
    - __If status.%%[[type]]%% is normal, increase j by 1.__
  - Let status be the result of Put(O, "length", len+argCount, true).
  - __Call the %%[[EndChange]]%% internal method passing O and "splice".__
  - __Let removed be the result of ArrayCreate(0).__
  - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: O, 0, removed and argCount.__
  - __Call %%[[EnqueueChangeRecord]]%% passing O and R.__
  - ReturnIfAbrupt(status)
  - Return len+argCount.


## Array Exotic Objects


### [[DefineOwnProperty]] (P, Desc)

9.4.2.1 %%[[DefineOwnProperty]]%% (P, Desc)

When the %%[[DefineOwnProperty]]%% internal method of an exotic Array object A is called with property P, and Property Descriptor Desc the following steps are taken:

  - Assert: IsPropertyKey(P) is true.
  - If P is "length", then
    - Return the result of calling ArraySetLength with arguments A, and Desc.
  - Else if P is an array index, then
    - Let oldLenDesc be the result of calling the %%[[GetOwnProperty]]%% internal method of A passing "length" as the argument. The result will never be undefined or an accessor descriptor because Array objects are created with a length data property that cannot be deleted or reconfigured.
    - Let oldLen be oldLenDesc.%%[[Value]]%%.
    - Let index be ToUint32(P).
    - Assert: index will never be an abrupt completion.
    - If index ≥ oldLen and oldLenDesc.%%[[Writable]]%% is false, then return false.
    - Let succeeded be the result of calling OrdinaryDefineOwnProperty passing A, P, and Desc as arguments.
    - ReturnIfAbrupt(succeeded).
    - If succeeded is false, then return false.
    - If index ≥ oldLen
      - Set oldLenDesc.%%[[Value]]%% to index + 1.
      - Let succeeded be the result of calling OrdinaryDefineOwnProperty passing A, "length", and oldLenDesc as arguments.
      - ReturnIfAbrupt(succeeded).
      - __Call the %%[[BeginChange]]%% internal method passing A and "splice".__
      - __Call the %%[[EndChange]]%% internal method passing A and "splice".__
      - __Let removed be the result of ArrayCreate(0).__
      - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: A, oldLen, removed and (index + 1 - oldLen).__
      - __Call %%[[EnqueueChangeRecord]]%% passing A and R__
    - Return true.
  - Return the result of calling OrdinaryDefineOwnProperty passing A, P, and Desc as arguments.


### ArraySetLength Abstract Operation

9.4.2.2 ArraySetLength Abstract Operation

When the abstract operation ArraySetLength is called with an exotic Array object A, and Property Descriptor Desc the following steps are taken:

  - If the %%[[Value]]%% field of Desc is absent, then
    - Return the result of calling OrdinaryDefineOwnProperty passing A, "length", and Desc as arguments.
  - Let newLenDesc be a copy of Desc.
  - Let newLen be ToUint32(Desc.%%[[Value]]%%).
  - If newLen is not equal to ToNumber( Desc.%%[[Value]]%%), throw a RangeError exception.
  - Set newLenDesc.%%[[Value]]%% to newLen.
  - Let oldLenDesc be the result of calling the %%[[GetOwnProperty]]%% internal method of A passing "length" as the argument. The result will never be undefined or an accessor descriptor because Array objects are created with a length data property that cannot be deleted or reconfigured.
  - Let oldLen be oldLenDesc.%%[[Value]]%%.
  - If newLen ≥oldLen, then
    - Return the result of calling OrdinaryDefineOwnProperty passing A, "length", and newLenDesc as arguments.
  - If oldLenDesc.%%[[Writable]]%% is false, then return false.
  - If newLenDesc.%%[[Writable]]%% is absent or has the value true, let newWritable be true.
  - Else,
    - Need to defer setting the %%[[Writable]]%% attribute to false in case any elements cannot be deleted.
    - Let newWritable be false.
    - Set newLenDesc.%%[[Writable]]%% to true.
  - Let succeeded be the result of calling OrdinaryDefineOwnProperty passing A, "length", and newLenDesc as arguments.
  - ReturnIfAbrupt(succeeded).
  - If succeeded is false, return false.
  - __Call the %%[[BeginChange]]%% internal method passing A and "splice".__
  - __Let removedList be a new List.__
  - __Let status be succeeded.__
  - __Let numAdded be newLen - oldLen.__
  - __If numAdded < 0, then let numAdded be 0.__
  - __While status.%%[[type]]%% is normal and newLen < oldLen repeat,__
    - Set oldLen to oldLen – 1.
    - __Let oldValue be the result of Get(A, ToString(oldLen)).__
    - __Let status be oldValue.__
    - __If status.%%[[type]]%% is normal, then__
      - __Add oldValue to the front of removedList.__
      - Let deleteSucceeded be the result of calling the %%[[Delete]]%% internal method of A passing ToString(oldLen).
      - __Let status be deleteSucceeded.__
      - If deleteSucceeded is false, then
        - Set newLenDesc.%%[[Value]]%% to oldLen+1.
        - If newWritable is false, set newLenDesc.%%[[Writable]]%% to false.
        - __Let status be the result of calling OrdinaryDefineOwnProperty passing A, "length", and newLenDesc as arguments.__
        - __If status.%%[[type]]%% is normal, then__
          - Return false.
  - __If status.%%[[type]]%% is normal and newWritable is false, then__
    - Call OrdinaryDefineOwnProperty passing A, "length", and Property Descriptor{%%[[Writable]]%%: false} as arguments. This call will always return true.
  - __Call the %%[[EndChange]]%% internal passing A and "splice".__
  - __Let removed be the result of ArrayCreateFromList(removedList).__
  - __Let R be the result of calling %%[[CreateSpliceChangeRecord]]%% with arguments: A, newLen, removed and numAdded.__
  - __Call %%[[EnqueueChangeRecord]]%% passing A and R.__
  - ReturnIfAbrupt(status)
  - Return true.