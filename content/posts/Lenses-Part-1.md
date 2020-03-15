---
title: "Haskell Lens - Part 1"
tags: [Haskell]
date: 2018-03-26 13:00:58 +0530
keywords: ["haskell", "lens"]
---

## Preface

`lens` are one of the most popular, yet confusing aspect of Haskell. To be fair, I could never really understand how they work. This series of posts is going to be my attempt to understand lens, the ideas and implementation details, and also the lens package. I hope I'll learn something in the process, and you will too (hopefully!).

Before we begin, I'd like to give a heads-up about few things, so you know what lies ahead.

1. I am assuming if you're here, you have a fair amount of idea about `Monoids`, `Monads`, `Functors`, `Applicatives`. If you're not very familiar with the concepts, here is a list of suggested reading.
    * [Applicative Monads](http://learnyouahaskell.com/functors-applicative-functors-and-monoids)
    * [Applicative Functors](https://namc.in/2018-01-12-applicative-functors)
    * [LYAH](http://learnyouahaskell.com/a-fistful-of-monads)
    * [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

2. In my opnion, you should probably get comfortable with those concepts before proceeding with lenses.

3. This is the most important part. I'm learning as I write this post. So if you feel I'm wrong about something, please do leave a note, and I'd be more than happy to correct myself.

---

## Introduction

The `lens` library allows us to query and update some value in deeply nested records. Well, that's the simplest problem it solves. So let's see some examples for better understanding.

The example I've selected is close to a project I did.

```hs
-- line.hs

{-# LANGUAGE TemplateHaskell #-}

data Line = Line { _start :: Point, _end :: Point} deriving (Show)

data Point = Point { _x :: Double, _y :: Double } deriving (Show)
```

Let's try to define a few getters and setters!

Keep in mind! We haven't _yet_ started using Lenses, so we will do it the old school way!

Setter =>

```hs
-- helper functions to make Point and Line

makePoint :: (Double, Double) -> Point
makePoint (x, y) = Point x y

makeLine :: (Double, Double) -> (Double, Double) -> Line
makeLine start end = Line (makePoint start) (makePoint end)
```

Now since data is immutable, you'd actually be creating a new point/line with given parameters.

Getters =>

```hs
> let line = makeLine (0, 1) (2, 4)

-- Record syntax gives functions for accessing the fields
-- following gives the _y coordinate of _end of line

> _y . _end $ line
```

Now consider you want to change the `_x` position of `_start` point of the `line` we have defined above, you have to write this:

```hs
line2 = line { _start = (_start line) { _x = 4 } }
```

It works, but it looks clumsy. And it definitely gets tougher to pack/upack data as more fields are added to each data type! Or if your data becomes more deeply nested.

This is because you need to re-create all of the wrapping objects around the value that you are changing, because Haskell values are immutable.

## Simple Lens

A lens, by definition, is a first class getter and setter.

**What do you mean by first-class?**

A function is called `first class` when you can manipulate them using ordinary functional programming ways. You can pass them around the same way as integers, sequences.

* Can be used without restrictions - put in containers, passed as input and returned as output from functions
* Can be constructed without restrictions - locally, globally, in-exression

More details can be read [here](http://freecontent.manning.com/learning-haskell-first-class-functions/). The above definition is more than enough.

Lenses package `get` and `set` functionality into a single value. So you can loosely say, that lens is a record with two fields `view` and `set` and can be defined as

```hs
data Lens' s a = Lens
  { view :: s -> a
  , set :: a -> s -> s }
```

Well, that's not how a lens is actually implemented, but it is the intuition. The above definition helps us define our lenses .

Lens comes in very handy in situation where data is deeply nested. What Lens typically does is; it groups two things together corresponding to a data structure:

1. The view of a value (like `_x` and `_end` given above).
2. A corresponding function.

Lenses combine 1 & 2, and give us the ability to read the values in the data structure, create new data structures with the value changed.

Going back the example we started with;

```hs
data Point = Point { _x :: Double, _y :: Double }

-- The "view" of the point variable _x

viewX :: Point -> Double
viewX p = _x p

-- The "set" or "update" of point variable _x

setX :: Point -> Double -> Point
setX p x = p { _x = x }

-- Now the lens combines view and set; by definiton

xLens :: (Point -> Double, Point -> Double -> Point)
xLens = (viewX, setX)
```

There you have it! You've successfully defined your first `Lens` !

Now there any many, many ways of defining lenses.

We will go more into detail about the `:type` of lenses.

According to our definition above, we can abstract out the type of lens as

```hs
type Lens' s a = Functor f => (a -> f a) -> s -> f s

data Lens' s a = Lens'
                     { view   :: s -> a
                     , over   :: (a -> a) -> s -> s}
```

i.e.; it is the combination of a view, and a set for some type `s` which has a field of type `a` .

The `xLens` above would be of type `Lens' Point Double` .

But the problem with this approach of writing a getter and a setter into a data type is that it doesnt scale very well.

If we wanted to do something like - increment the value of some field by 1, first we will have to define a getter for that field, then apply `+ 1` to it and then define a setter to set the new value.

We can try to combine this entire process, by providing another function to `Lens'` : `over` .

You can think of `over` as similar to `set` , with slightly different defintion, i.e. `over` can be understood as combinator for setters. It works a lot like fmap, except that you pass a setter as it's first argument to specify which part of the data structure you want to focus on.

In fact, you could use this similarly to `set`.

```hs
-- using over to move _x by +1

over :: Lens' -> (a -> a) -> (s -> s)

let p1 = Point { _x = 1, _y = 3 }
over xLens (+ 1) p1
```

Okay, let's look back at our definition of `xLens` ; and see if we can redefine the definition of our lens using over.

```hs
-- xLens :: (Point -> Double, Point -> Double -> Point)
-- using "over"

xLens :: Lens' Point Double
xLens = Lens' _x
                (\a s -> s { _x = a })           -- getter
                (\f s -> s { _x = f (_x s) })    -- over
```

Good so far!

But the problem now is that for each lens, we will have to provide a `view`, a `set` and `over` even if we're just using one of them!

We can solve this by using `const` .

It has a type `s -> a -> s` which allows us to write `over :: (a -> a) -> (s -> s)` as `set :: s -> a -> s` by partially applying it. In other words, we can rewrite `set` as :

```hs
set :: Lens' s a -> s -> a -> a
set xLens s a = over xLens (const s) a
```

So the final definition of our lens looks like this :

```hs
data Lens' s a = Lens'
                     { view :: a -> s
                     , over :: (a -> a) -> s -> s }

set :: Lens' s a -> s -> a -> a
set xLens s a = over xLens (const s) a
```

Okay, so recap time!

* We saw how to define lenses using `view`, `set` and `over`
* We saw how to define `view` to retrieve value and `set`/`over` to retrieve values.
* We also saw how we can rewrite `set`.

Going back to definition : lenses package both "get" and "set" functionality into a single value (the lens).

You could pretend that a lens is a record with two fields:

```hs
data Lens' s a = Lens'
    { view :: s -> a
    , over :: (a -> a) -> (s -> s)
    }
```

This looks a little different from our initial defintion, and if you have followed the transition so far, you probably understand why.

## Using functors

So far, we have seen how `over` can be used to reach nested nodes of a data structure. But the functionality was more  related to `insert` than `update` . What if the modifier function needs to perform modifications where there are some side effects?

E.g. : We might want to print the value to the console; which is an IO function.

Just like before, we could add another function, using IO data type :

```hs
-- new definition of lens

data Lens' s a = Lens'
                     { view   :: s -> a
                     , over   :: (a -> a) -> s -> s
                     , overIO :: (a -> IO a) -> s -> IO s }
```

Again, there are some issues with the above implementation

* The definition of lens has grown again, and we would like very much to avoid that
* What if `over` is used in more settings than just **IO**? Would we define new functions for the same?

At this point we would like to revisit the definition of `over` and try to generalize it. You must have noticed, to define `overIO` we added `IO` .

We can easily swap IO for a more generalised type -> **Functor** .

```hs
over :: Functor f => (a -> f a) -> s -> f s
```

Now here is an interesting argument. `over` is typically a functor modified, ie. – it does what the original functor does, but it also attaches a value to it. Similarly, we can pick what Functor we specialize `f` to , and depending on which Functor we pick, we get different defintions.

For example :

**(This is covered in more details in following sections)**

if you pick (f = Const t), you get a `view` like function

```hs
type Getter' a s a  = (a -> Const a a) -> (s -> Const a s)

--  equivalent to: (a -> a) -> (s -> a)
```

if you pick (f = Identity) , you get an `over` like function

```hs
type Setter' s a   = (a -> Identity a) -> (s -> Identity s)

--  equivalent to: (a -> a) -> (s -> s)
```

Now since we can use functor to implement both; getters and setters; we can redefine our lens to following :

```hs
-- new definition

type Lens' s a = Functor f => (a -> f a) -> s -> f s
```

By making this type an alias, instead of newtype or data, you can define your own lenses without depending on the lens library. Any function which has the appropriate type signature is a lens.

## RankNType

Lens uses [rank 2 types](https://wiki.haskell.org/Rank-N_types) ; i.e. `Lens'` type synonym is polymorphic and occurs in the signature in what is called "negative position", that is, to the left of the `->` .

```hs
type Lens' s a = Functor f => (a -> f a) -> s -> f s

-- This typechecks
checks :: Lens' (s,t) s
checks = _1

-- This fails with error "Illegal polymorphic or qualified type: Lens' s t"
fails :: Lens' (s,t) s -> Int
fails = const 5
```

Here the **lens'** is in negative position, and ranges only over `s -> Int`. It is the implementation of `fails`, and not the caller of `fails`, the one who chooses the type of `s`. The caller must supply an argument function that works for all `s`.

Therefore it is imperetive to use :

```hs
{-# LANGUAGE RankNTypes #-} -- Rank2Types is a synonym for RankNTypes
```

at the top of your file. Without it, it's illegal to even use a lens type synonym since they use a part of the language you must enable. And if you try to use lens without enabling `RankNTypes` it will throw **Illegal polymorphic error**.

## Lens Type

So, the actual type for lenses is:

```hs
type Lens s t a b = Functor f => (a -> f b) -> s -> f t

type Lens' s a = Functor f => (a -> f a) -> s -> f s
```

* [`Lens`](https://hackage.haskell.org/package/lens-4.15.3/docs/Control-Lens-Type.html#t:Lens) : Lens family as described in [mirrored lenses](http://comonad.com/reader/2012/mirrored-lenses/)
* [`Lens'`](https://hackage.haskell.org/package/lens-4.15.3/docs/Control-Lens-Type.html#t:Lens-39-) : Simple Lens, used whenever the type variables don't change upon setting a value.

We've already seen the derivation of `Lens'` type, in the previous section.

### Different forms of lenses

Quick recap of a few things we covered so far (they will be handly in this section)

* Lens represent a getter and a setter into some data type.
* Setter can be generalized to work with functions, by using `over`
* Generalized `over` to the [van Laarhoven lens](https://github.com/ekmett/lens/wiki/History-of-Lenses) of **Functor f => (a -> f a) -> s -> f s** .
* We saw how `Lens' s a` can behave like `over` , `set` and `view` .

But what if you just need a simple modification, and no functors, or vice versa?

In this section, we are going to define different forms of lenses, prove it and understand why it works as intended. We will be using two Functor instances that come from the base library, namely **Data.Functor.Identity** and **Control.Applicative.Const**.

### Lens as ordinary setter

If you just want to set or modify the value, use `Identity`, which is precisely the functor to use when you don't actually need one, because you can put a value in, let it behave as a functor and then the value out.

`Identity` functor is defined as :

```hs
newtype Identity a = Identity { runIdentity :: a }

instance Functor Identity where
  fmap f (Identity a) = Identity (f a)
```

`Identity` is also the "empty" functor and applicative functor, which means, `Identity` composed with another functor or applicative functor is isomorphic to the original. `Lens'` is defined as a function polymorphic in a Functor, therefore we can represent the defintion of `Lens' s a` in terms of `Functor` in the definition of `over`:

```hs
type Lens' s a = Functor f => (a -> f a) -> s -> f s

over :: Lens' s a -> (a -> a) -> s -> s

-- can be rewritten as
over :: (Functor f => (a -> f a) -> (s -> f s)) -> (a -> a) -> s -> s
```

Specializing `f` to `Identity` :

```hs
Functor f => (a -> f a) -> s -> f s

(a -> Identity a) -> s -> Identity s

-- which is isomorphic to
(a -> a) -> s -> s
```

so given an updating function on a, return an updating function on s.

As we have already established, the final form of `over` is

```hs
over :: Lens' s a -> (a -> a) -> s -> s
```

i.e. : Given a lens with focus on `a` inside of `s` , and a function from `a` to `a` and `s` , you get back a modified `s` after applying the function `over` to the focus point of the lens.

Keep in mind that `Lens` is just a function, nothing more

Therefore, by substituting `Lens s a` by `Identity` functor we can define `over` as :

```hs
over :: ((a -> Identity a) -> (s -> Identity s)) -> (a -> a) -> s -> s

-- which helps us define
over ln f x = runIdentity (ln (\y -> Identity (f y)) x)

{-
    Arguments :
    ln :: (a -> Identity a) -> (s -> Identity s)
    f  :: a -> a
    x  :: a
-}

-- a more syntactically pleasing version to write he above expression
-- could be by making use of point free style
over ln f x = runIdentity $ ln (Identity . f) x
```

And since `x` remains unchanged, and no operation is being performed upon it, we can go ahead and remove it from our definition

So our final definition of `over` using Identity stands as :

```hs
over ln f = runIdentity $ ln (Identity . f)

{-
    Arguments :
    ln :: (a -> Identity a) -> (s -> Identity s)
    f  :: a -> a
-}
```

### Lens as a getter

Since we're implementing a getter we need to make use of `view` . The definition of `view` is

```hs
view :: Lens s a -> s -> a
```

which translates as , given a lens that focuses on `a` inside `s` and `s`, it returns `a`

The type of our lens is

```hs
Lens s a :: (a -> f a) -> s -> f s
```

and by definition, view implements `s -> a` , which means, we need to find a way to convert `f s` to `a` .

We will be using Const to imlement the getter because it works for all (applicative) Functors and we wish to reuse the getter in a degenerate sense. Using `Const` is analogous to passing in const or id to functions that work, given arbitrary functions.

Lenses are defined in terms of arbitrary Functors, which is why we can make use of `Const` to derive a field accessor (and trivial Identity to derive a field updater).

#### **Understanding how Const works**

Const is defined as

```hs
newtype Const a b = Const { getConst :: a }

instance Functor (Const a) where
  fmap _ (Const a) = Const a
```

Const works as wrapper, which takes a value and hides it, pretends to be functor which contains something else, and ignores the function you are trying to `fmap` over Const.

For example let's hide string "haskell" inside a Const, and apply an bool function to it, using fmap.

```hs
> let strBool = fmap (&& True) (Const "haskell")
> :t strBool
:: Const [Char] Bool
```

As you can see, the Const is now of type `Const [Char] Bool` .

Similarly, if we map over a function `Bool -> Int` we'll get type as `Const [Char] Int`

```hs
> :t fmap (\_ -> 1 :: Int) strBool
:: Const [Char] Int
```

**Point to note** : Const ignores the function we're doing `fmap` with, and takes on a new type, and ensures safety of our original `b` . We can etract it whenever desired, despite any numbers of `fmap` operations.

```hs
> getConst strBool
"haskell"

> getConst $ fmap (\_ -> 1 :: Int) strBool
"haskell"
```

#### **Back to lens**

The final form of `view` looks like

```hs
view :: Lens' s a -> s -> a
```

Since we can specialize `Lens' s a` type synonym to use (Const a) in place of `f` , we can represent our lens as `(a -> Const a a) -> (s -> Const a s)` .

Doing the same for `view` gives us

```hs
view :: Functor f => (a -> f a) -> (s -> f a)

-- `f` becomes `Const`
view :: ((a -> Const a a) -> (s -> Const a s)) -> s -> a

-- `f x` becomes `ln Const x`
view ln x = _ (ln Const x)

-- using getConst to get back the original value of x
view ln x = getConst (ln Const x)

-- using point free style
view ln x = getConst $ ln Const x

{-
    Arguments :
    ln :: (a -> Const a a) -> (s -> Const a s)
    x  :: s
-}
```

### **Defining setter using Const**

Since we have already defined setters using `over` , the implementation with const is fairly trivial.

```hs
set :: Lens s a -> a -> s -> s

-- set in terms for Functor f
set :: Functor f => (a -> f a) -> s -> f s

-- f becomes `Const`
-- using `over` to go through values in ln
set ln x = over ln (const x)
```

## Write your own lens

* Lenses are nothing more than this: an easy way of modifying parts of some data.

* Because it becomes so much easier to reason about certain concepts because of them, they see a wide use in situations where you have huge sets of data structures that have to interact with one another in various ways.

* A simple lens can be defined as `type Lens' s a = Functor f => (a -> f a) -> s -> f s`

* To use lens as a simple modifier, use Identity in place of f.

* You can also make use of `over` to define modifiers

* To use lens as a getter, use `Const` as f - it would store the a value, save it from fmap and return as it is to you.

* `f` can be generalized to use any applicative functor.

We can use the above knowledge to define our own lenses :

```hs
{-# LANGUAGE Rank2Types #-}

import Control.Applicative
import Control.Monad.Identity

-- The definition of Simple Lens:
type Lens' s a = Functor f => (a -> f a) -> s -> f s

-- Getter passes the Const functor to the lens:
view :: Lens' a b -> a -> b
view l = getConst . (l Const)

-- Updater passes the Identity functor to the lens:
over :: Lens' a b -> (b -> b) -> (a -> a)
over l f = runIdentity . l (Identity . f)

set :: Lens' a b -> b -> (a -> a)
set l r = over l (const r)

-- Example: -------------------------------------------

data Point = Point { _x :: Double, _y :: Double }
    deriving (Show)

xLens :: Lens' Point Double
xLens f (Point x1 y1) = fmap (\x -> Point x y1) (f x1)

data Line = Line { _start :: Point, _end :: Point}
    deriving (Show)

startLens :: Lens' Line Point
startLens f (Line s e) = fmap (\x -> Line x e) (f s)

main :: IO ()
main = do
    let point = Point 2.0 3.0
    print $ view _x point
    print $ set _x 4.0 point

    let line = Line point point
    print $ view _start point
    let newPoint = Point 6.0 3.0
    print $ set _start newPoint line
```

## Combine Lenses

Since lenses are just functions, we can compose them using using ordinary function composition.

Think of your function composition as of type

```hs
(.) :: Lens' a b -> Lens' b c -> Lens' a c
```

Lens' is just a type alias for higher order functions,

```hs
type Lens' a b = forall f . Functor f => (b -> f b) -> (a -> f a)
```

So when you compose two higher order functions, you get back a new-higher order function

```hs
(.) :: Functor f
    => ((b -> f b) -> (a -> f a))
    -> ((c -> f c) -> (b -> f b))
    -> ((c -> f c) -> (a -> f a))
```

Let's take our example of point and segment; and use `xLens` and `startLens` from the section above.

```hs
startLens :: Lens' Line Point
xLens     :: Lens' Point Double
```

We can make a composition of these two lenses, by simply doing,

```hs
startLens . xLens :: Lens' Line Double
```

This composite lens lets us get or set the x coordinate of the starting point of a line. We can use over and view on the composite Lens' and they will behave exactly the way we expect:

```hs
view (point . x) :: Line -> Double

over (point . x) :: (Double -> Double) -> (Line -> Line)
```

## Lens Laws

As per the documentation, there are three lens laws.

1. You get back what you put in:

```hs
view l (set l v s) ≡ v
```

2. Putting back what you got doesn't change anything:

```hs
set l (view l s) s ≡ s
```

3. Setting twice is the same as setting once:

```hs
set l v' (set l v s) ≡ set l v' s
```

**Please follow these laws, because if you don't, the lens police will come and get you!** 🚓

## Exercise

Since we have defined `xLens` and `startLens` , try defining `yLens` and `endLens` on your own, and later combine them.

```hs
data Point = Point { _x :: Double, _y :: Double }
    deriving (Show)

yLens :: Lens' Point Double
yLens f (Point x1 y1) = fmap (\y -> Point x1 y) (f y1)

data Line = Line { _start :: Point, _end :: Point}
    deriving (Show)

endLens :: Lens' Line Point
endLens f (Line s e) = fmap (\y -> Line s y) (f e)

-- ghci
> endLens . yLens :: Lens' Line Double
```

Define a lens for changing the absolute value of a number using a lens :

```hs
_abs :: Real a => Lens' a a
_abs f n = update <$> f (abs n)
  where
    update x
      | x < 0     = error "_abs: negative absolute value"
      | otherwise = signum n * x

-- using _abs in ghci

-- add 10 to absolute value of n
> over _abs (+ 10) (-5)
-15

-- square the absolute value of n
> over _abs (^ 10) (-5)
-25
```
