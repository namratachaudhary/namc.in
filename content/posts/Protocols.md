---
title: "Structured Clojure: Protocols"
description: "Clojure Protocols - Definition, implementation, extension, reification"
tags: [Clojure]
date: 2018-01-04 10:20:20 +0530
keywords: ["protocols", "clojure"]
---

Protocols in clojure are a way to define functions that operate in different ways on different types. As someone [mentioned](https://stackoverflow.com/questions/4509782/simple-explanation-of-clojure-protocols#4513556), Clojure protocols help in solving the expression problem.

### Expression Problem

Your program is a combination of a datatype and operations over it.

Expression problem calls for an implementation of the program such that —

* new functionalities and data-structures can be added that play well with existing functionality

* the above is done whilst maintaining type safety; i.e. without recompiling old modules, and without casts/runtime type checks.

For languages like Javascript or Clojure, even if you cannot statically check type-safety, you would still expect that level of consistency. Although, it is easy to add new procedures and functions, but it is relatively tougher to add new data types , esp where switch , pattern-matching (case , cond) etc are involved. You might have to modify your code to accommodate these changes.

So we make use of Protocols in Clojure (Implicits in Scala, Typeclasses in Haskell, Interface in Java) to solve the Expression Problem

—-

Rough idea of what follows
We’re going to define a bunch of protocols and geometrical shapes to demonstrate how protocols work, and are defined. Then we will try to extend and reify them.

--

### Definition

Since we’re talking about geometrical shapes, let’s assume they are closed, and have an area. So that becomes a protocol.

```clojure
user> (defprotocol Name
        "This interface states `name` method should be implemented"
        (name [shape] "name of the shape"))

user> (defprotocol Area
        (area [shape] "calculates area"))
```

Now we define an immutable persistent maps, which will implement the protocol.

```clojure
user> (defrecord Square [size])
user> (defrecord Rectangle [a b])
```

### Implementation

We have defined protocols, and records on which those protocols will be implemented. However, we need to define how those protocols shall be defined for respective records.

```clojure
user> (extend-type Square
        Name                         ; enforces the protocol Shape
        (name [shape]                ; implements `name` associated
          "Square")
        Area                         ; enforces the protocol Area
        (area [shape]                ; implements `area` associated
        (Math/pow (:side shape) 2)))
```

Let’s extend the protocols to other record.

```clojure
user> (extend-type Rectangle
        Name
        (name [shape]
          "Rectangle")
        Area
        (area [shape]
        (* (:a shape) (:b shape))))
```

As you can see, we are not trying to change the values of the `map` . In fact we are using the existing properties to define how the protocol should be implemented, since `area` and `shape` differs for each record.

### Extension

The purpose of extending a protocol is to provide several implementations of the same protocol all at once. A protocol can be extended using another protocol.

Let’s define another protocol for the `shapes`, but there is a constraint. We need to extend the protocol such that this new protocol does not alter the existing code. Anyway we cannot change the definition of `defrecords` (they’re immutable).

So is this possible? Yes. Let’s define another property for the shape, `Perimeter`.

```clojure
user> (defprotocol Perimeter
        "Returns the perimeter of the shape"
        (perimeter [shape] "perimeter of the shape"))

user> (extend-protocol Perimeter
        Square
          (perimeter [shape] (* (:side shape) 4))
        Rectangle
          (perimeter [shape] (* (+ (:a shape) (:b shape)) 2)))
```

Great!

But what if we have a new shape?

```clojure
user> (defrecord Triangle [a b c])
```

Can we extend our protocols to the new record without changing the existing code? Sure, let’s try — 

(FYI we did this above)

```clojure
user> (extend-type Triangle
        Name
        (name [shape]
          "Triangle")
        Area
        (area [{:keys [a b c]}]                ; this can be sketchy
        (let [s (/ (+ a b c)) 2]               ; Heron's formula
          (Math/sqrt (* s (- s a) (- s b) (- s c))))))
```

### Reification

reify is for the case where you want to create an anonymous implementation of a protocol, which is useful for taking advantage of local context e.g. creating local overrides.

```clojure
;;; 'reduce-unit-area' is the function returing a 'reify' of area.
;;; This is when someone computes an area, the area of some shapes
;;; is reduced by 1 unit.
user> (defn reduce-unit-area []
        (reify Area
          (area [_]
            (- 1 (area shape)))))
```

You can try this code in REPL and see the results for yourself.

