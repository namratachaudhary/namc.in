---
title: "Haskell : Higher-Rank and Higher-Kinded Types"
tags: [Haskell]
date: 2020-06-15 12:20:12 +0530
keywords: ["polymorphism", "types", "kinds", "haskell"]
---

Hi! We're gonna look at higher-rank and higher-kinded types, specifically in Haskell but I will try to cover the concept as generically as possible. 

## Two types of "Polymorphism" in Haskell

* Parametric Polymorphism
* Ad-hoc Polymorphism (also known as typeclass) 

**A function is parametrically polymorphic if it behaves uniformly for all types, in at least one of its type parameters** 

Couple of examples could be - 

a. Simple swap function
```hs
swap :: (a, b) -> (b, a)
swap (x, y) = (y, x)
```

b. Not-so-simple len function
```hs
-- Maps each element to a 1, then sums up all the elements
len :: [a] -> Integer
len = sum . map (\_ -> 1)
```
The idea is, you want these functions to work for all values of `a` or `b`. The function `swap` will swap elements regardless of their types, and function `len` returns the length of a list no matter of what is in that list. 

This is also what we call _generics_ in Java or _templates_ in C++ - but it was first introduced in Ocaml, Standard ML and the kinds. 

Now what if we need a way to write an unique a function which accepts any type for which a computation/operation is possible and we need to provide a specific implementation for each case. This is **ad-hoc polymorphism** and we use _typelasses_ in Haskell to implement this.

Everyone loves demonstrating this with `area` function example so let's do that. Also, the `Show`, `Ord`, `Eq` typeclasses are inherited by free from parametric polymorphism. 

```hs
{-# LANGUAGE FlexibleInstances #-}
{-# LANGUAGE TypeSynonymInstances #-}
{-# LANGUAGE TypeFamilies #-}

class Area t where
  area :: t -> Float

instance Area Square where
  area (Square s) = s * s

instance Area Rectangle where
  area (Rectangle a b) = a * b
```

Conclusively - If your function does not care about input type, or need any context about them, then parametric polymorphism is the way to go, else use ad-hoc polymorphism and add implementation for each type. 

For this part, we only need to focus on **PARAMETRIC POLYMORPHISM** Moving on..

**(Not-so) Fun fact time**
Since Haskell's type-system is based on **[Hindley-Milner](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)** also commonly known as _let-bound polymorphism_ implies using `let` or `where` bound indentifiers are polymorphic, but lambda-bound identifiers are monomorphic. More info here - [Haskell - Type Pitfalls](https://www.cs.auckland.ac.nz/references/haskell/haskell-intro-html/pitfalls.html)

## Higher-rank types

In most languages, polymorphic functions are first-class values ; by defintion, can be stored in variables and passed to functions. But in Haskell, they are not. Higher rank types , or as they're called `RankNTypes` in Haskell are used to make polymorphic functions first-class, just like regular (monomorphic) functions.

[`RankNTypes`](https://wiki.haskell.org/Rank-N_types) does this by introducting `forall` argument to the function type.

```hs
--- Illegal Foo!
foo :: (Int, Char)
foo = (\f -> (f 1, f 'a')) id
```

The above function `foo` can be fixed by - 
* binding `f` with `let` or `where` (using let-bound polymorphism)
  ```hs
  foo :: (Int, Char)
  foo = let f = id
       in (f 1, f 'a')
  ```

* using `RankNTypes` extension
  ```hs
  {-# LANGUAGE RankNTypes #-}
  {-# LANGUAGE ScopedTypeVariables #-}

  foo :: (Int, Char)
  foo = (\(f :: forall a. a -> a) -> (f 1, f 'a')) id
  ```

To be honest, this does not usually come up in Haskell, or any statically typed language because supporting higher-rank types make type inference undecidable - [Typability and type checking in System F are equivalent and undecidable](https://www.sciencedirect.com/science/article/pii/S0168007298000475). Ryan Scott's blog does a great job at explaining why working with Higher-rank Kinds is a bit tedious, but it is also a great resource for someone wanting to explore this further. [The surprising rigidness of higher-rank kinds](https://ryanglscott.github.io/2019/07/10/the-surprising-rigidness-of-higher-rank-kinds/). 

## Higher-kinded types

Also known as type of types or _type operators_ .. but is it really? 

Higher-kinded types are really common in Haskell (vs higher-rank types). Commonly known as `Functor` or `Monad` - they are popular examples of higher-kinded polymorphism. 

As [Stephen Diehl mentions](http://dev.stephendiehl.com/fun/001_basics.html#higher-kinded-types) - kind of an ordinary type is usually written as `*`, and type constructors (unary type operators) have kind `* -> *`

To make it simpler to understand - 

* The kind `*` is also known as ground, or 0th order.
* Any kind of the form `* -> * -> ... -> *` with at least one arrow is first-order.
* A higher-order kind is one that has a "nested arrow on the left", e.g., `(* -> *) -> *`.

We can look at some commonly used defintions - 

* **Maybe** 
  ```hs
  Maybe : Type->Type
  ```
  Since, we cannot get a type from type constructor, we can get a `kind` . Therefore, 

  ```hs
  λ> :k Maybe
  Maybe :: * -> *
  ```
* **Shape** 
  ```hs
  data Shape f = Shape (f ())
  [(), (), ()] :: Shape List
  ```
  Which you can further divide into `Traversables` and break it down into further components. 

Next, we will look at a basic definition of class `Functor`

```hs
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

`fmap` changes the type parameter of an `f` from `a` to `b` but does not change `f`. 
If we use `fmap` over a list, we get a list (etc). These are compile-time guarantees (revisited later). 

Two questions here - 
* Can't we just use function overloading instead?
* What if we simply convert our types to `Seq` and do whatever we want with it?

Well, that's correct. Additionally if you have a type which supports conversion to and from `Seq` you get `map`  by reusing `Seq.map`. 

Why do we need a Functor class? First, functor classes allow you to implement fmap for types which do not support conversion to and from `Seq` like - IO actions, functions, etc. Therefore, making the concept of mapping very sequence-agnostic. 

As we have seen previously the two functor laws - 
* `fmap id xs == xs` : mapping with an identity/noop function is the same as doing nothing
* `fmap f (fmap g xs) = fmap (f . g) xs` : any result that you can produce by mapping twice, you can also produce by mapping once

Which is why, its important for `fmap` to maintain the static, compile time guarantees of preserving types. 

So if we try to define functor class over IO - we get :

```hs
instance Functor IO where
    fmap f action =
        do x <- action
           return (f x)

newtype Function a b = Function (a -> b)

instance Functor (Function a) where
    fmap f (Function g) = Function (f . g)
```

Another popular usage is seen in lambda calculus, where `Alg` has kind `(* -> *) -> * -> *` enabling us to write recursion schemes on top of datatypes.

```hs
data Alg f a = Alg (f a -> a)
```

In languages like Java, you cannot write

```java
class ClassExample<T, a> {
    T<a> function()
}
```

In Haskell `T` would have kind `*->*`, but a Java type (i.e. class) cannot have a type parameter of that kind, a higher-kinded type.

Personally, I feel these concepts are better understood from the perspective of lambda calculus, parametric polymorphism with higher-rank types as System F, and higher-kinded types as System λω. Hopefully we can cover that in future. 

Thanks! 