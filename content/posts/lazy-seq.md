---
title: "lazy-seq - Lazy Sequences in Clojure"
description: "lazy-seq - How? Why? And other heresies"
tags: [Clojure]
date: 2018-01-10 01:00:46 +0530
keywords: ["lazy-seq", "lazy sequences", "clojure"]
---

### Basic idea

> Evaluation of an expression is delayed until the value is needed.

There are two ways to achieve the above :

* use lambda functions at runtime

* use macros/special forms at compile time

With lazy eval techniques, it is possible to construct infinite data structures that are evaluated as consumed, using lambdas, closures and recursion. In clojure, they are generated using lazy-seq and cons forms.

Let’s take an example from the [Clojure docs for lazy-seq](http://clojuredocs.org/clojure.core/lazy-seq)

```clojure
user> (def fib-seq
 (lazy-cat [0 1] (map + (rest fib-seq) fib-seq)))
;; => #'user/fib-seq
```

`lazy-cat` is another macro to concatenate lazy sequences. We shall write the above expression to an alternate translation which uses `lazy-seq`

```clojure
user> (def fib-seq (concat (lazy-seq [0 1]) (lazy-seq (map + (rest fib-seq) fib-seq))))
;; => #'user/fib-seq
user> (doall (take 2000 fib-seq))
```

This lazy-seq evaluates for very large indexes without giving an `OutOfMemoryException/StackOverFlowError`. (Might throw ArithmeticException for extremely large integer, though)

How this works internally :

* `lazy-seq` executes the body once the first time it is accessed.

* the result is cached and used whenever the function is called in future.

Going back to the example;

`[0 1]` is the base case for the problem. In Fibonacci’s sequence, the first two values are 0 and 1. The consecutive computation of each value requires summation of previous value and the value before that one. Hence, we need at least two values to start the process.

```clojure
(map + (rest fib-seq) fib-seq) 
```

Here, rest will return the part of fib-seq without it’s head. map will combine two sequences using + and produce a next sequence. Let’s take the fibonacci sequence `fib-seq-temp`

```clojure
user> (def fib-seq-temp [0 1 1 2 3])
user> (map + (rest fib-seq-temp) fib-seq-temp)
;; => (1 2 3 5)
```


The sequences fed to map are part of same basic sequence . The first sequence `(rest [seq])` is the base seq without the head. The second sequence is the base seq, including the first value.

**What is the base sequence here?**

According to fibonacci number’s definition, `A(n) = A(n-1) + A(n-2)` . We need to be able to access previously computed values in order to extend the sequence. So, we use the recursive reference to fib-seq to access our previous results. The base seq is typically `[0 1]` here to begin with.

Let’s follow the steps mentioned above and run through the process :

* Take the first element of `fib-seq` (base case [0 1] ) — 0

* Take the second element of `fib-seq — 1`

* Take the third element of fib-seq — Stuck, aren’t we? 

Here, we use map to generate a sequence which is used as remaining values.

The sum of rest [0 1] which is 1, and first item of [0 1] , which is zero is 1 ; which is our result below.

```clojure
user> (map + (rest [0 1]) [0])
;; => (1)
```

Similarly, let’s repeat the process for generating fourth element.

We use map again for generating base seq. 

Compute the sum of second item of rest [0 1], whose value is (1) as computed above; and second item of [0 1]

```clojure
user> (map + (rest [0 1]) [1])
;; => (2)
```

Easy so far?

Now generate the fourth element. 

Sum the third element of rest [0 1] (which we computed as 1) and fourth item generated from[0 1] (which was 2 ). And we get 3 .

This will keep building up to match the desired definition for series. To understand this better, let’s add a print statement to our original definition.

```clojure
user> (def fib-seq
  (lazy-cat [0 1]
            (map
              (fn [a b]
                (println (format "%d + %d" a b)
                (+ a b))
            (rest fib-seq) fib-seq)))
;; => #'user/fib-seq
user> (doall (take 10 fib-seq))
1 + 0
1 + 1
2 + 1
3 + 2
5 + 3
8 + 5
13 + 8
21 + 13
;; => (0 1 1 2 3 5 8 13 21 34)
```

—-

### Working with different data-types

* `map` , `reduce` , `take` etc work in terms of `first` , `rest` and more .

* `first` (as well as others) ensure if the collection implements `ISeq` .

* If yes, then these functions are implemented directly. Else, a seq view of the collection is created to implement first / more / rest .

### lazy-seq macro

* The macro uses `thunk` to store the sequence-generating calculations.

* When an element/chunk of sequence is requested, next thunk is called to retrieve the values.

* The thunk creates next subroutine to represent the tail of the sequence (lose your head).

* These thunks implement sequence interface, each thunk is called just once, and then cached. So the realized portion is just a sequence of values.

### Holding onto the head

Take these two methods to generate infinite lazy seqs of randoms.

```clojure
user> (defn infinite[] (lazy-seq (cons (rand) (infinite))))
;; => #'user/infinite
```
--
```clojure
user> (defn infinite-1[] (lazy-seq (cons (rand) (infinite-1))))
;; => #'user/infinite-1
user> (def zyx (infinite-1))
```

If you try to run `(last (infinite))` ; it won’t crash, but will neither terminate. However, `(last zyx)` will crash quickly with an `OutOfMemoryError` .
This is because of “holding onto the head”.

In `(last (infinite))` , Clojure disposes the earlier members of sequence because it is able to determine that they are not of any use. Hence, memory is not lost. Whereas, in `(last zyx)` , the program is holding on to the definition , and hence, it is not able to clear the memory. This causes it to throw `OutOfMemoryError`.

### So what’s the point of lazy-seq ?

* lazy-seq is just one of many possible ways to create lazy sequences. And there are several other ways to do it in clojure.

* If a sequence is not lazy, it often holds onto it’s head, which consumes a lot of heap space.

* If it is lazy, it is computed, then discarded as it is not used for further computations.

* This is particularly useful when you are dealing with huge sequences, and only using a small part of them for sequential computation.

—-

More reading:

[Purely Functional - What are lazy sequences](https://purelyfunctional.tv/lesson/what-are-lazy-sequences/)

[Attic Light - Lazy Sequences in clojure](http://theatticlight.net/posts/Lazy-Sequences-in-Clojure/)

