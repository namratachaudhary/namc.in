---
title: "Foldable and Traversable in Haskell"
tags: [Haskell]
date: 2018-01-24 12:07:52 +0530
keywords: ["foldable", "traversable", "haskell"]
---

## Foldable

Foldable represents data structure type class that

* provides a generalisation of list folding (foldr and friends).
* provides operations derived from list foldings to arbitrary data structures

You can use a Foldable where you would have to traverse a dataset and reduce it to a single result.

* Get the product of a list
* Get the max path value in a tree

In short, `fold` can be understood as function to reduce a large structure into a single result.

### Foldable Class

```hs
class Foldable t where
    foldMap 	:: Monoid m => (a -> m) -> t a -> m
    fold 	:: Monoid m => t m -> m

    -- following have default implementations:
    foldr 	:: (a -> b -> b) -> b -> t a -> b
    foldr' 	:: (a -> b -> b) -> b -> t a -> b
    foldl 	:: (b -> a -> b) -> b -> t a -> b
    foldl' 	:: (b -> a -> b) -> b -> t a -> b
    foldr1 	:: (a -> a -> a) -> t a -> a
    foldl1 	:: (a -> a -> a) -> t a -> a

    toList 	:: t a -> [a]
    null 	:: t a -> Bool
    length 	:: t a -> Int
    elem 	:: Eq a => a -> t a -> Bool
    maximum 	:: Ord a => t a -> a
    minimum 	:: Ord a => t a -> a
    sum 	:: Num a => t a -> a
    product 	:: Num a => t a -> a
```

Foldable is a great example of how monoids can help formulating good abstractions as we can see `fold` and `foldMap` require the elements of the Foldable to be Monoids.

Monoids simply define a zero element via `mempty` and an associative operation mappend for combining two Monoids into one. `mconcat` makes use of `mappend` and `mempty` in it's default implementation.

```hs
class Monoid a where
    mempty  :: a
    mappend :: a -> a -> a
    mconcat :: [a] -> a

---
mconcat :: [a] -> a
mconcat = foldr mappend mempty
```

Let's look at some examples :

```hs
> length [1..10]
6

-- sum all elements of the list

-- just using map, without Foldable.
> summ xs = let ys = 0 : map (\(a,b)->a+b) (zip xs ys) in last ys
> summ [1..10]
55

-- using foldr
> foldr (+) 0 [1..10]
55

-- using sum
> sum [1..10]
55

-- check if element exists
> elem 4 [1..10]
True
```

Instead of thinking of `sum` as a function which is fmapped across a list and accumulating its elements with (+), Foldable and foldMap help us think of it as function which queries each element for its value and summarises the results using `Sum` monoid.

Monoidal summary perspective is important while using folds as it separates the details of data structure from the expected results.

### fold and foldMap

Foldable has no laws of its own and they are mostly general-purpose, however, fold and foldMap use monoid homomorphism.

In the example above `tList` was a list of integers `[Int]` , where the `[]` is a Foldable. `Int` is not a Monoid. So as long as we do not use any function from Foldable that requires Monoid, it will work fine. But if you try to use `fold` or `foldMap` on tList, it will throw an error.

```hs
> foldr1 (+) [1..10]
55

> fold [1..10]
<interactive>:1:1:
    No instance for (Monoid a0) arising from a use of ‘it’
```

In order to solve this problem, we can wrap the integers in a Monoid, such as Sum or Product, and fold them.

And for that, we can use `foldMap`

```hs
> foldMap Sum [1..10]
Sum {getSum = 55}
```

Thus, fold can be implemented in terms of foldMap, since while using foldMap, we need to provide a function to convert each item in list to a Monoid.

```hs
fold :: Monoid m => t m -> m
fold xs = foldMap id xs
```

### toList - List like folds

Any Foldable data structure can be converted to List using

```hs
toList :: Foldable t => t a -> [a]
```

.. which is a part of Foldable. If you use toList , folding the resulting list will produce the same result than folding the original structure directly. We can use `foldMap` to define toList.

```hs
toList = foldMap (\x -> [x])
```

Also, lists are free monoid for Haskell types. This means, any value can be given to the monoid such that the information is neither added nor erased from the original data.

```hs
-- Given a list xs :: [a]

xsFoldMap :: Monoid m => (a -> m) -> m
xsFoldMap = \f -> foldMap f xs
```

Since lists are free monoid, we can recover the original list xs by supplying `(\x->[x])` to xsFoldMap.

Since we know folding with Foldable operations will cause in some loss of information if the data structure is complex, using `toList` to implement folds make it possible to reconstruct the original structure.

### Examples

```hs
> import qualified Data.Set as S

> let testSet = S.fromList [1..10]
> testSet
fromList [1,2,3,4,5,6,7,8,9,10]

-- using toList to define data structure as free monoid
> import Data.Foldable
> toList testSet
[1,2,3,4,5,6,7,8,9,10]

-- using foldMap on list above result
> foldMap show testSet
"12345678910"

> foldMap Sum tSet
55
```

## Traversable

### Derivation

Consider the following Functor and Foldable instances for lists:

```hs
instance Functor [] where
    fmap _ []     = []
    fmap f (x:xs) = f x : fmap f xs

instance Foldable [] where
    foldMap _ []     = mempty
    foldMap f (x:xs) = f x <> foldMap f xs
```

Both `fmap f` and `foldMap f` walks across the list and apply `f` to each element. However, **fmap f** collects the result by rebuilding the list, and **foldMap f** collects the result by combining them with `mappend`.

But if we have to add a condition to our traversal, we can add it as a `Maybe`:

```hs
deleteTens :: (Num a, Ord a) => a -> Maybe a
deleteTens x = if (mod x 10 == 0) then Nothing else Just x

-- ghci
-- Here, fmap is the Functor, and deleteTens is our foldable
-- This results in [Maybe a]
> fmap deleteTens [0..20]
```

Why can't we use a direct `Foldable` ? Because Foldable would replace the structure of the original list with that of whatever Monoid we pick for folding, and we will not be able to get back the original list.

But now we need a way to convert `Maybe` to list. To do that, we can combine the Maybe contexts of the values and recreate the list structure withing the combined context.

To do that, we make use of a type class which combines Functor context : `Applicatives` . This leads us to

```hs
-- sequenceA :: Applicative f => [f a] -> f [a]
instance Traversable [] where
    sequenceA []     = pure []
    sequenceA (u:us) = (:) <$> u <*> sequenceA us

-- equivalently:
instance Traversable [] where
    sequenceA us = foldr (\u v -> (:) <$> u <*> v) (pure []) us
```

* Traversable is to Applicative contexts what Foldable is to Monoid values.
* Similary, sequenceA is analogous to fold as it creates a summary of context within a structure, and rebuild structure with new context.

So how do we get the original list type back from `fmap deleteTens [0..20]`

```hs
> let seqWithoutTens = sequenceA . fmap deleteTens
> :t seqWithoutTens
seqWithoutTens
  :: (Num a, Ord a, Traversable t) => t a -> Maybe (t a)

> seqWithoutTens [0..10]
Nothing

> rejectWithNegatives [0..5]
Just [0,1,2,3,4,5]
```

### Traversable class

`Traversable` is another type classes in the Prelude that can be used for data structure manipulation. A Traversable represents data structure which can be traversed, collecting results at each stop.

However, Traversable does not provide us with a way to change the data.

Let's look at the typeclass

```hs
class (Functor t, Foldable t) => Traversable t where
    traverse  :: Applicative f => (a -> f b) -> t a -> f (t b)
    sequenceA :: Applicative f => t (f a) -> f (t a)

    -- These methods have default definitions.
    -- They are merely specialised versions of the other two.
    mapM      :: Monad m => (a -> m b) -> t a -> m (t b)
    sequence  :: Monad m => t (m a) -> m (t a)
```

We will pretty much be using `traverse` and `sequenceA` for most of our operations.

And so, rewriting our Traversable instance to incorporate traverse and sequenceA :

```hs
instance Traversable [] where
	-- traverse
    traverse _ []     = pure []
	traverse f (x:xs) = (:) <$> f x <*> traverse f xs

	-- sequenceA
	sequenceA [] = pure []
	sequenceA (x:xs) = (:) <$> x <*> sequenceA xs
```

And if sequenceA is analogous to fold, traverse is analogous to foldMap.
They can be defined in terms of each other, and therefore a minimal implementation of Traversable would look like:

```hs
class (Functor t, Foldable t) => Traversable t where
    traverse :: Applicative f => (a -> f b) -> t a -> f (t b)
    traverse f = sequenceA . fmap f

    sequenceA :: Applicative f => t (f a) -> f (t a)
    sequenceA = traverse id
```

**sequenceA**

```hs
> sequenceA [["a", "b"], ["c", "d"]]
[["a","c"],["a","d"],["b","c"],["b","d"]]
```

sequenceA works similar to `sequence :: Monad m => [m a] -> m [a]` from Control.Monad , it just additionally takes the applicative effects, runs them on the list, and then gives us the result.

In above example, our resultant list is of 4 elements, each consisting of 2 elements, which is exactly the kind of output you would expect from combinging with `(<*>)` by combining a 2x2 list.

sequence :: Monad m => [m a] -> m [a] from Control.Monad and sequenceA are doing the same thing. It simply takes the Applicative effects, runs them and pulls them out of the list.

**traverse**

The type of traverse, along with an example :

```hs
traverse :: (Applicative f, Traversable t) => (a -> f b) -> t a -> f (t b)

-- ghci

> traverse (\x -> Just (x + 1)) [0..4]
Just [1,2,3,4,5]
```

traverse type resembles mapping functions. traverse allows traversals which add an extra layer of context on top of the structure. It is a slightly modified version of `(>>=)` . The structure below new layer matches the original original structure, but the values are changed obviously.

As you can see in example above, the result maintains the structure of `[0..4]`

**Conclusive Properties**

This enables us to define a few properties of lens :

* Traversals traverse all elements and each element is traversed only once
* Traversals do not involve any skips or repetitions
* the structure of the original structure is retained, though the values may change

### Traversable Laws

#### Identity Law

Traversing with `Identity` constructor wraps the structure with Indentity, which changes nothing and the original structure can be recovered by `runIdentity` . Thus, Identity constructor is the identity traversal.

```hs
newtype Identity a = Identity { runIdentity :: a }

instance Functor Identity where
    fmap f (Identity x) = Identity (f x)

instance Applicative Identity where
    pure x = Identity x
    Identity f <*> Identity x = Identity (f x)

-- formulating
traverse Identity = Identity
sequenceA . fmap Identity = Identity
```

#### Composition Law

`Compose` is used to form composition of functors, but if we compose two Applicatives, we get an Applicative as result.

As per composition law, it does not matter whether the two traversals are performed separately or if they are composed in order to walk the structure only once.

The composition law is stated in terms of the Compose functor:

```hs
newtype Compose f g a = Compose { getCompose :: f (g a) }

instance (Functor f, Functor g) => Functor (Compose f g) where
    fmap f (Compose x) = Compose (fmap (fmap f) x)

instance (Applicative f, Applicative g) => Applicative (Compose f g) where
    pure x = Compose (pure (pure x))
    Compose f <*> Compose x = Compose ((<*>) <$> f <*> x)

-- formulating
traverse (Compose . fmap g . f) = Compose . fmap (traverse g) . traverse f
sequenceA . fmap Compose = Compose . fmap sequenceA . sequenceA
```

#### Applicative Homomorphism Law

An applicative homomorphism is analogoues to [Monoid homomorphisms](https://en.wikipedia.org/wiki/Monoid#Monoid_homomorphisms), and it is a function which preserves the Applicative operations, such that:

```hs
-- Given a choice of f and g, and for any a,
t :: (Applicative f, Applicative g) => f a -> g a

t (pure x) = pure x
t (x <*> y) = t x <*> t y

-- formulating
t . traverse f = traverse (t . f)
t . sequenceA = sequenceA . fmap t
```

## Implementation for Foldable using Traversable

Traversable allows us to define Functor and Foldable, i.e. implement both `fmap` and `foldMap` using `traverse` as long as Traversable instance follows the laws.

#### fmap

Comparing fmap with traverse :

```hs
fmap 	 :: Functor f => (a -> b) -> f a -> f b
traverse :: (Traversable t, Applicative f) => (a -> f b) -> t a -> f (t b)
```

In `traverse` the function that is passed returns a value wrapped in Applicative context, and the result obtained from traverse is also wrapped.

We can Identity functor to wrap the value after we apply the functor and then unwrap it at the end (using `runIdentity :: Identity a -> a`), to get the exact same type as fmap. Thus, using Idenity to make traversal out of an arbitrary function, we get :

```hs
fmap f = runIdentity . traverse (Identity . f)
```

#### foldMap

Comparing foldMap with traverse :

```hs
foldMap  :: (Traversable t, Monoid m) => (a -> m) -> t a -> m
traverse :: (Traversable t, Applicative f) => (a -> f b) -> t a -> f (t b)
```

We need a way to convert each element of Traversable to a Monoid. To do this, we will use `Const` from Control.Applicative.

`Const` is a constant functor such that, the value of type `Const a b` holds value `a` which is unaffected by `fmap` . To define `foldMap` , `a` value would be an Applicative, such that :

```hs
instance Monoid a => Applicative (Const a) where
    pure _ = Const mempty
    Const x <*> Const y = Const (x `mappend` y)

-- (<*>) combines the values in each context with mappend
```

In order to make a traversal out of any `Monoid m => a -> m` function, we can define foldMap as :

```hs
foldMap f = getConst . traverse (Const . f)
```

#### Foldable

After defining `fmap` and `foldMap`, we can use them to define a Functor and a Foldable instance for any Traversable.

For example, let's take a simple list type :

```hs
-- our definitions

fmap' f    = runIdentity . traverse (Identity . f)
foldMap' f = getConst . traverse (Const . f)

-- defining Functor and Foldable instance for List

data List a = Nil
            | Cons a (List a)
            deriving Show

instance Functor List where
    fmap = fmap'

instance Foldable List where
    foldMap = foldMap'

instance Traversable List where
    traverse _ Nil = pure Nil
    traverse f (Cons x xs) = fmap Cons (f x) <*> traverse f xs
```

Checking how it works on ghci :

```hs
> traverse (\x -> Just (x + 1)) (Cons 1 (Cons 2 (Cons 3 Nil)))
Just (Cons 2 (Cons 3 (Cons 4 Nil)))

> fold (Cons "hello" (Cons "haskell" Nil))
"hellohaskell"

> fmap (+1) (Cons 1 (Cons 2 (Cons 3 Nil)))
Cons 2 (Cons 3 (Cons 4 Nil))
```

