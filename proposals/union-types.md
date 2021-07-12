# Union types

* **Type**: Design proposal
* **Authors**: Mikhail Glukhikh, Evgeny Bukreev
* **Contributors**: Roman Elizarov
* **Status**: Proposed
* **Prototype**: In Progress
* **Related issues**: [KT-13108](https://youtrack.jetbrains.com/issue/KT-13108)
* **Discussion**: TODO

## Summary

Support *union types* and *multi-catch block*.

## Motivation / use cases

- When writing Kotlin APIs that can succeed or fail, you’d use a sealed class as a result, providing a type-safe listing of all possible types of errors that a function can fail with. 
  This can become quite verbose too
- Sometimes it is necessary to write a Kotlin API for a set of types without taking out specific information from them
- When working with exception-heavy Java APIs in Kotlin, you cannot catch multiple exception types in a single catch() clause. 
  You’ll have to write multiple catch() clauses or use a `when` expression to test the exception type
- >106 votes on [KT-13108](https://youtrack.jetbrains.com/issue/KT-13108): Denotable union and intersection types
- >466 votes on [KT-7128](https://youtrack.jetbrains.com/issue/KT-7128): Multi catch block

## Description

We propose to add denotable *union types* to the Kotlin type system. 
The *union type* will be created by listing the nested types with `|`. 
For example:
```kotlin
val x: Int | Float | Double = ...
```

`when` with a *union type* check will be exhaustiveness, because this is a frequent use case:
```kotlin
fun getSize(x: Int | String | Boolean): Int {
    return when (x) {
        is Int -> x
        is String -> x.length
        is Boolean -> 1
    }
}
```

*Union types* also provides a syntax for multi-catch when working with legacy Java APIs:
```kotlin
fun loadData(path: String) {
   try {
       findDataLoader(path).loadXmlData()
   } catch (e: IOException | JDOMException) {
       println("Failed")
   }
}
```
## Resolution

### Subtyping
<code>
type A = a<sub>1</sub> | a<sub>2</sub> | ... | a<sub>i</sub> (i can be 1)
</code>
<br>
<code>
type B = b<sub>1</sub> | b<sub>2</sub> | ... | b<sub>j</sub> (j can be 1)
</code>
<br>
<code>
A <: B &hArr; &forall; a &isin; A &exist; b &isin; B : a <: b
</code>

### Common super type
Common super type is the lower type which is the supertype for all nested types in a union type.
Common super type can be an intersection type if all nested types implement some interface, which does not implement the common super class.

### Resolution algorithm
For cases:
```kotlin
var x: Foo | Bar
```
```kotlin
fun foo(x: Foo | Bar)
```
```kotlin
fun foo(): Foo | Bar = x
```
and similar x can take values, associated with a given type by subtyping.

To invoke some function, on a union type, it's need to look for this function in a common super type.

## Open questions

- Allowed places for union types: catch parameter, value parameter, return type, local variable type
- Forbidden places for union types: smart-casts, as/is, extension function receiver, supertypes / type parameter bounds
- Functional types in union types
- Storing meta information in bytecode for function signature
