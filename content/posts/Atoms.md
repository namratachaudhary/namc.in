---
title: "Clojure - Atoms"
tags: [Clojure]
date: 2017-12-26 12:39:46 +0530
keywords: [Clojure, Atoms]
---

In Clojure, everything is immutable! So how can Clojure be used to build stateful applications? That’s where `Atom` enter the picture.

In simple words, atom in Clojure is a built-in mechanism to manage application state. `Atoms` are mutable, and can be updated as many times as you want.

Let’s look at a practical example to understand the usage and implications of atom . We begin by looking up the documentation of atom in repl.

```clojure
user> (use 'clojure.repl)
user> (doc atom)

-------------------------
clojure.core/atom
([x] [x & options])
  ..
;; => nil
```

Now we know how to create an `Atom`. So let’s go ahead and define an atom which points to a value; say "pikachu". This is just for simplicity, the atom can point to any thing; an int 1, a map `{:a "apple" :b "boogy"}` ; you can create an atom of any values.

```clojure
user> (def atom-str (atom "pikachu"))
;; => #'user/atom-str
```

The value of `atom-str` can be obtained in two ways

```clojure
user> @atom-str

-- Or

user> (deref atom-str)
;; => "pikachu"
```

### Swap! and Reset!

**Note :** Exclamation mark `(!)` means that this function has side-effects. i.e. it changes state for atoms, metadata, vars, transients, agents and io as well

`swap!` allows us to use a function to update the value of an atom. Let’s look at how to use `swap!`.

```clojure
user> (doc swap!)

-------------------------
clojure.core/swap!
([atom f] [atom f x] [atom f x y] [atom f x y & args])
  ..
;; => nil

```

The function used to update the value should be pure ; because `swap!` internally uses `compare-and-set!` to update the value atomically. And failures could make `swap!` re-run the function. Hence, it is important the function returns a value without interacting with any other part of the system; and thus should be free of side effects.

Let’s use swap! to add a string `"_new"` at the end of our atom.

```clojure
user> (swap! atom-str (fn [value] (str value "_new")))
;; => "pikachu_new"
```

`reset!` changes the current value of atom; without caring what the current value is.

```clojure
user> (reset! atom-str "bulbasaur")
;; => "balbasaur"
```

**Use swap! if you need to use the current value to determine the new value.**

Another example to illustrate the difference between `swap!` and `reset!` is

```clojure
(def atom-int (atom 1))
(swap! atom-int inc)   ; @x is now 2
(reset! atom-int 100)  ; @x is now 100
(swap! atom-int inc)   ; @x is now 101
```

### Threadsafety

`Atoms` are threadsafe. Atomic mutable state with immutable values gives you composable concurrency semantics. They can be understood as similar to mutable variables in other programming languages.

Values can be assigned and updated any time. The value will be updated by one thread at one time using swap!. And since it uses pure function; it is safe to say that the process has no side effects. You could also use locks with vars, but it is undeterministic to say what the final value will be, since the code will be executed in parallel on different threads.

Exercise : Update a nested map in an atom thread-safely.

```clojure
user> (def nested-map (atom {:id {:count 0}}))
;; => #'user/nested-map

user> (defn update-count [id]
  (swap! nested-map update-in [(keyword id) :count] inc))
;; => #'user/update-count

user> (update-count "id")
;; => {:id {:count 1}}
```

### Conclusion

`atom` is the state; and mutable object. value is immutable, it can be shared by threads without having any change to it’s value. Atoms guarantee that no matter how many threads are trying to change the value, each change is calculated from previous value; and no previous values are lost.

Atoms basically provide user with a way to make immutable data structures and concurrency primitives to manage state collectively. In Clojure there is a distinct separation of value and state, and this is how Clojure helps with concurrency.

