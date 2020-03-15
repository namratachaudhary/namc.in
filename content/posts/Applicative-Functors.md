---
title: "Haskell's Applicative Functors"
tags: [Haskell]
date: 2018-01-12 18:20:12 +0530
keywords: ["applicative functors", "functors", "monads", "haskell"]
---

Understanding Applicative functors would require some understanding about Functors and Monads. We shall try to look at brief definitions for the scope of this article.

### Functor

According to [Haskell docs](https://hackage.haskell.org/package/base-4.8.1.0/docs/Data-Functor.html), a functor is simply something that can be mapped over.

In other words, it is an abstraction for a context with the ability to apply a function to all the things inside the context. The context can be defined as container, computation etc. E.g.; a list or sequence is a container of homogenous elements. You can apply a function to each of the elements, which produces a new sequence of elements transformed by the function.

Let’s create a new functor, implementing `myfmap` such that `Data.Map` is an instance of the new functor typeclass. `myfmap` applies a function `f` to the value(s) inside functor’s context while preserving the context.

```haskell
import Data.Map as DataMap
import Data.List as DataList

class MyFunctor f where
  myfmap :: (a -> b) -> f a -> f b

instance (Ord k) => MyFunctor (DataMap.Map k) where
  myfmap f x = DataMap.fromList $
                     DataList.map (\(p,q) ->(p,f q)) $
                     DataMap.toList x
```

### Applicative Functor

In computer science, applicative is an abstraction for a context, and it has the ability to apply functions in the same type of context to all elements in the context. For example, a sequence A which has homogenous elements, and sequence B which consists of functions that can be applied to sequence A , which produce a new sequence of elements transformed by all the functions. (Super confusing? Sorry 😕 . We will clear it up)

The Applicative typeclass in Haskell is located in Control.Applicative module and defines pure and `(<*>)` .

```haskell
class (Functor f) => Applicative f where
    pure :: a -> f a
    (<*>) : f (a -> b) -> f a -> f b
```

* An Applicative has to be Functor. This is a class constraint.

* `pure` takes any context and returns an Applicative the value of context inside it.

* `<*>` is a representation of `fmap` , where `<*>` takes a functor that has a function in it and another functor and run the function from first functor and maps it over the second functor. Whereas `fmap` which takes a function and functor and applies the function inside the functor.

—-

### List an an Applicative!

List also instantiates an applicative typeclass, with implementation as :

```haskell
instance Applicative [] where
    pure x = [x]
    fs <*> xs = [f x | f <- fs, x <- xs]
```

Here, the implementation of `(<*>)` is basically a list comprehension, where every function is applied to every value. For example :

```shell
ghci> (*) <$> [2, 3] <*> [4, 5]
[8, 10, 12, 15]
```

If you want to apply each function in first list to the respective value in second list, the [ZipList typeclass](https://hackage.haskell.org/package/base-4.8.1.0/docs/Control-Applicative.html#t:ZipList) is very handy.

--

### Curious case of [Either](https://hackage.haskell.org/package/base-4.10.0.0/docs/Data-Either.html)

The base **Monad** instance for `Either` is defined as follows.

```hs
instance Monad (Either e) where
  return = Right
  Left e  >>= _ = Left e
  Right a >>= f = f a
```

This instance has inherent short-circuiting. But in case you would like to collect error messages which occur anywhere in the above computation, it goes against `(>>=)` and `lazy-evaluation/short-circuiting` .

* Lazy evaluation is when we proceed from left to right, when a single computation “fails” into the Left then all the rest do as well.

* `(>>=)` takes a function, maps it over an instance of a monad and then flattens the result.

```hs
(>>=) :: m a -> (a -> m b) -> m b
```

* `(>>=)` produces `m b` from `m a` so long as it can run `(a -> m b)` . This demands that the value of a should ideally exists during the time of computation, and this is impossible for `Either` .

So let’s try to solve the above problem by defining a **Functor** instance of Either .

```hs
instance Functor (Either a) where
    fmap f (Left x) = Left x
    fmap f (Right y) = Right (f y)
```

Things we can understand from the above definition :

* We know the definition of `fmap :: (c -> d) -> f c -> f d` .

* If we replace `f` with `Either a` , we get `fmap :: (c -> d) -> Either a c -> Either a d`

* The problem with this implementation of `Either` is that we cannot map over Left .

😱 Why?
To understand that, let `Either a b` computation, which may succeed and return b or fail with error a , similar to monad instance. So the functor instance does not map over `Left` values since you would want to map over the computation, if it fails, there is nothing to manipulate.

### Implementing an Applicative instance

Applicative Monad instance cannot have a corresponding `Monad`.

As we saw the definition of Applicative, we will define pure and `(<*>)` . Defining `pure` is rather simple, as we want it return the Right element. Implementation of `(<*>)` is little tricky . The following cases need to be considered for defining `(<*>)` .

```hs
instance Applicative (Either e) where
    pure                 =  Right
    Right f <*> Right a  = Right (f a)
    Left  e  <*> Right _ = Left e
    Right _  <*> Left  e = Left e
    Left e1  <*> Left e2 = Left (e1 <> e2)
```


* The first statement is the pure statement.

* `(<*>)` allows evaluation in parallel instead of necessarily needing results from previous computation to compute present values.

* Thus, we can use our purely `Applicative Either` to collect errors, ignoring Right if any Left exist in the sequence.

* As soon as it hits a Left, it aborts and returns that Left.

```shell
ghci> (++) <$> Left "Hello" <*> undefined
Left "Hello"                              -- not undefined

ghci> (++) <$> Right "Hello" <*> undefined
*** Exception: Prelude.undefined          -- undefined

ghci> (++)  <$> Right "Hello" <*> Left " World"
Left " World"

ghci> (++)  <$> Right "Hello" <*> Right " World"
Right "Hello World"
```


### Limitations

There’s some limitations to using purely applicative functor. As we saw in the definition of `(>>=) :: m a -> (a -> m b) -> m b` ; which means that without `(>>=)` you can’t pick “what to do next based on what came before”.

Also, if you take a generally pure function and feed the applicative arguments to it, like

```shell
> f :: a -> b -> c
> f <$> getLine <*> getProcessID "chrome" <*> getFreeMemory
```

All the arguments will get evaluated, no matter what. That is, you cannot express — “if the second argument exits, abort the rest of the computation”. Due to this, anything recursive, as well as most interactive programs do not use Applicatives. Monads, on the other hand are a good choice in that case.


--

More Reading :

[LearnYouAHaskell - Functors, Applicative Functors and Monads](http://learnyouahaskell.com/functors-applicative-functors-and-monoids)

[Adit.io - Functors, Applicative Functors and Monads in pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

[Hackage - Control Applicative](https://hackage.haskell.org/package/base-4.8.1.0/docs/Control-Applicative.html)

[StackOverfow - Monad vs Applicative](https://stackoverflow.com/questions/17409260/what-advantage-does-monad-give-us-over-an-applicative)

