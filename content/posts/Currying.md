---
title: "What is Currying"
tags: [Haskell]
date: 2018-02-22 12:07:52 +0530
keywords: ["haskell", "currying"]
markup: mmark
---

## Inspiration

This post is written after I saw this tweet

<blockquote class="twitter-tweet" data-cards="hidden" data-lang="en"><p lang="en" dir="ltr">Every functional programming tutorial:<br><br>&gt; OK, we&#39;re going to talk about currying. Currying is when you break down a function tha-<br>*scrolls down*<br>&gt; const Y = f =&gt; (g =&gt; g(g))(g =&gt; f(x =&gt; g(g)(x))) <a href="https://t.co/EPHH9SG5ay">pic.twitter.com/EPHH9SG5ay</a></p>&mdash; Ben Howdle (@ben_howdle) <a href="https://twitter.com/ben_howdle/status/966358609683668992?ref_src=twsrc%5Etfw">February 21, 2018</a></blockquote>

And realized that we need a simpler tutorial to understand currying, the logic behind it, the nuances and the usage.

We will be using Haskell for this tutorial, but this can be extended to any functional programming language.

## Higher-order functions!

In mathematics, higher-order functions (HOFs) are called functionals. But to simplify it for general software engineers, HOF is simply a functions that

* either takes a function as an argument
* or returns a function as a result

A well-known higher order function is map, which performs an action on each element in a collection, e.g.:

```hs
-- Haskell
map (\x -> 2 * x) [1, 2, 3, 4, 5] -- => [2, 4, 6, 8, 10]
```

In usual imperative programming, functions can accept values, like integers and strings (first order functions) and and return a value of some other type.

Consider a function, `f` such that -

```hs
f a b c = a + b - c
```

What if we wanted to generalize the operators in above function? That would make our operators a variable too. Right? Let's see a way to define a more generalized version of `f`, where `g` and `h` represent the operator-functions.

```hs
let f g h a b c = a `g` b `h` c

> f (+) (-) 2 3 4 -- returns 1
> f (*) (/) 2 3 4 -- returns 1.5
```

That was simple! But, we mentioned that HOFs can return a function too. Yes, we can create a function that accepts a function and an argument and returns another function, which accepts an argument and returns a result. For example :

```hs
let g f n = (\m -> m `f` n)

> f = g (+) 2
> f 10 -- returns 12

> f = g (*) 2
> f 10 -- returns 20
```

In the above example, `(\m -> m `f` n)` construct is an anonymous function of 1 argument `m` which applies `f` to `m` and `n`. And using this anonymous functions, we can define `g h 2` which is a function that accepts one argument `g` and operates `h` to it.

**Trivia : ** Scheme, according to Wikipedia, was the first language to introduce proper higher-order functions as first-class citizens, however the first mention dates back to Frege in "Funktion und Begriff" (1891).

## Currying

Currying is a technique that transforms a function of several arguments to a function of a single argument, which returns a function of 1 argument, which returns a functions of 1 argument ... till it returns a value.

A curried function is one that returns a function as its result. A fully curried function is a one-argument function that either returns an ordinary result or returns a fully curried function.

Note that a curried function is necessarily a higher-order function, since it returns a function as its result.

Let's take an example, where we have a function of two args, like (+). But what if you could define the same function, by giving only one argument to it, thereby returning a function, which you could use later to add thid 1st argument, now encased in this new function, to something else.

```hs
f n = (\m -> n + m)

> g = f 10
> g 8 -- would return 18
> g 4 -- would return 14
```

Same can be written in javascript like :

```js
function add (a) {
  return function (b) {
    return a + b;
  }
}

add(3)(4)
```

How could be make it more abstract? Lets define `curry` , which takes a function and an argument, such that

```hs
curry f n = \m -> f n m

g = curry (+) 5
> g 10 -- returns 15

h = curry (*) 5
> g 10 -- returns 50
```

If we check the type `:t` of `curry` function defined above, we find,

```hs
curry :: (a -> b -> c) -> a -> (b -> c)
```

## Haskell functions

Haskell curries all functions by default. There are no functions of multiple arguments in Haskell. What you have are only functions of one argument, some of which may return new functions of one arguments. So you can define a multi-argument function as you would in any other language and you automatically get a curried version of it, without having to write lamdas yourself.

For example, take the function `(++)` which concatenates two strings together.

```hs
:t (++)
(++) :: [a] -> [a] -> [a]
```

The type definition can be understood as, `(++)` accepts one list, and returns a function of type `[a] -> [a]`. The resultant function can accept yet another list, and we get a new list of type `[a]`.

If you are familiar with other languages, you can correlate `f a b c` as Lisp's `(((f a) b) c)` or Java's `f(a, b, c)` . This makes sense as , in Haskell, the function `f` is curried by default.

However, when you're analyzing types, the association is from right to left, so `[a] -> [a] -> [a]` is equivalent to `[a] -> ([a] -> [a])`, which means that when you apply an argument to the function, you get back a function of type `[a] -> [a]`.

Similiary, you can analyze `map`, which accepts a function `(a -> b)` and a list `[a]` , and returns a transformed list `[b]`, such that function is mapped over all elements of `[a]`.

```hs
:t map
map :: (a -> b) -> [a] -> [b]
```

Try the following snippet out in ghci

```hs
(*)
(*) 5
(*) 5 10
map
map (\x -> head x)
map (\x -> head x) ["hello", "haskell", "kitty"]
map head
map head ["hello", "haskell", "kitty"]
```

## Partial application

Partial application is when you call a function with 1 or multiple arguments to get back a function that still accepts arguments. This allows us to write shorter and concise code.

Partial application can be used as a substitute for anonymous functions. Let's look at some examples.

```hs
(\x -> take 2 x)

-- can be written as
(take 2)
```

What if you want to use an operator, say `(+)` instead of `take`? For infix functions, you can partially apply it using [sections](https://wiki.haskell.org/Section_of_an_infix_operator).

```hs
(\x -> 2 + x)
-- can be written as
(2+)

-- and
(\x -> x + 2)
-- can be written as
(+2)
```

In place of `(+)` , you can use any binary function to define sections.

```hs
(\xs -> 2 elem' xs)

-- can be written as
(2elem')
```

The point to note here is , since every function is curried by default, and accepts one argument only, so sections can be used with any function. For example ,

```hs
let f a b c d = a + b + c

-- such that f is (+)

> (2f) 3 4 5 -- returns 14
```

## Currying and Pattern Matching

We take for granted that we can have nested patterns and patterns over more than one term, when in reality for the purposes of a compiler the only thing you can do is branch on the top-level constructor of a single value. So the first stage of the compiler is to turn nested patterns (and patterns over more than one value) into simpler patterns.

Let's take an example, a naive algorithm might transform your function into something like this:

```hs
myFunc = \x y -> case x of
    0 -> case y of
        0 -> 0
        _ -> x `some_operation` y
    1 -> case y of
        1 -> 1
        _ -> x `some_operation` y
    _ -> x `some_operation` y
```

As we can see, there are a lot of caveats in the above implementation.

* the `some_operation` term is repeated a lot
* the function expects both arguments before it will even start to do a case at all

Refer to [A Term Pattern-Match Compiler Inspired by Finite Automata Theory](https://www.classes.cs.uchicago.edu/archive/2011/spring/22620-1/papers/pettersson92.pdf) for further discussions on how we can improve it.

Anyway, in the form below, it should actually be a bit more clear how the currying step happens. We can directly substitute for `x` in this expression to look at what `myFunc 0` does:

```hs
myFunc 0 = \y -> case 0 of
    0 -> case y of
        0 -> 0
        _ -> 0 `some_operation` y
    1 -> case y of
        1 -> 1
        _ -> 0 `some_operation` y
    _ -> 0 `some_operation` y
```

Now this is still a lambda, so no further reduction is done.

We will need to change the definition of our function if we want GHC to do more computation after supplying only one argument. There's a time/space tradeoff here. So GHC leaves it in the programmer's hands to make this choice. For example, you could explicitly write

```hs
myFunc 0 = \y -> case y of
    0 -> 0
    _ -> 0 `some_operation` y

myFunc 1 = \y -> case y of
    1 -> 1
    _ -> 1 `some_operation` y

myFunc x = \y -> x `some_operation` y
```

and then `myFunc 0` would reduce to a much smaller expression.

## Compositions

Compositions and Application operator are very handy for writing concise and flexible code.

* **Composition operator** `(.)` chains functions together.
* **Application operator** `($)` applies function on the left side to the argument on the right side

`f $ x` is equivalent to `f x`. However `($)` has the lowest precedence of all operators, so we can use it to get rid of parentheses: `f (g x y)` is equivalent to `f $ g x y`.

It is also helpful when we need to apply multiple functions to the same argument:

```hs
map ($2) [(2+), (10-), (20/)]

-- would yield
[4,8,10]
```

The following expressions are all equivalent

```hs
f (g (h (x + y + z)))   -- 1

(f . g . h) (x + y + z) -- 2

f $ g $ h $ x + y + z   -- 3

f . g . h $ x + y + z   -- 4
```

`(.)` and `($)` are different things. More can be read about that here - [Learn you a haskell ($)](http://learnyouahaskell.com/higher-order-functions#function-application)

