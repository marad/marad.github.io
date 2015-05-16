---
title: The Mighty Monad
date: 2015-04-26 07:57:00
tags: ["monad"]
categories: programming scala
---

When you begin to learn functional programming everybody is telling you that
pure functions with no side effects are the only way to go. This is fine up to
the point. Sooner or later you will hear about monads. This post will try to
explain what the monad is and why we need it so badly in functional programming
that everybody is talking about it.

# The definition

What everyone is telling you is that a monad has its origins in category
theory. This, of course, is true but do we really need that information to use
it? The answer is: no! This is also said everywhere by everybody :)

Nevertheless I think it is a good idea to write down the definition and then
simplify it for our needs. So here it goes!

We can say that _M_ is a monad when:

* there is generic type M[T]
* there is a _unit_ function T => M[T]
* there is a _flatMap_ function (M[T], T => M[T]) => M[T]

To make things even worse, there are three monad laws:

* **Left identity** 

    `unit(x) flatMap f == f(x)`

* **Right identity**

    `m flatMap unit == m`

* **Associativity**

    `(m flatMap f) flatMap g == m flatMap (f flatMap g)

Let me clarify this definition a bit. Leaving the laws for later let us jump
back to the _M_ type and the methods _unit_ and _flatMap_.  Type _M_ is just
regular generic type like `Option[T]` or `Try[T]`.  The _unit_ function takes
the value of type _T_ and wraps it with a monadic type. For `Option[T]` the
_unit_ operation is simple `Some(t: T)`.  Lastly - the _flatMap_. To be honest
literature refers to it as the _bind_ operation. I decided to call it _flatMap_
here so it is much more familiar. It, of course, takes a monadic value _M[T]_
and a function `T => M[T]` and returns _M[T]_. This basically means that we can
do some computation on value that is inside the monad and create a new monad
with the result of that computation. Scala is object oriented so `flatMap`
is just a method and takes one parameter (the function) the other being
just _this object_.

To sum this up with an example:

* we have got generic type Option[T]
* we have _unit_ function `Some(t: T)`
* we have _bind/flatMap_ method `Option.flatMap(f: T => Option[T]): Option[T]`

Going back to the monad laws. The first two laws are fairly simple and describe
the relations between _unit_ and _flatMap_. The _associativity_ law tells us
just that the order of the _flatMap_s doesn't matter.

# Simpler definition

If you feel that the monad laws weren't described well enough - you're probably
right. The reality is that these laws are mainly important when you want to
implement a monad yourself or you need to use one of these properties in your
code. In fact most developers when they're talking about monads they're
thinking just about generic type _M[T]_ with _unit_ and _flatMap_ operations.

There is even more! There are types that we call a monads but they do not
satisfy those laws! Let's take `Try[T]` for example. This monad is used to deal
with possible exceptions that might occur while processing some data. If you
have some operation `expr: T` that can throw some exception you may want to
wrap it with `Try(expr)`. This will return `Success(t: T)` containing the
result of the function or `Failure(ex: Throwable)` with the exception that was
thrown by that function. If you closely examine all monad laws you can see that
the _left identity_ law does not hold.

    `unit(x) flatMap f == f(x)`

This tells us that following should be true:

    `Try(expr) flatMap f == f(expr)`

The law works fine if everything goes smooth and no exception is thrown. The
problem pops up when either `f` or `expr` throws an exception. The left hand
side `Try(expr) flatMap f` never throws an exception and just returns
`Failure(ex)`. The right hand side `f(expr)` will just throw the exception so
the law does not hold thus `Try[T]` is not precisely a monad, but that is not a
problem for us. We are not mathematicians. We just want things to work :)

# So what does all this means for developers?

Monadic structure gives us, developers, a uniform way of defining a chain of
transformations on virtually any type. Just look at this:

val list = List(1, 1, 2, 3, 3, 3, 3, 4, 4, 5)
list                                                        // a list
    .groupBy(identity)                                      // a map
    .mapValues(_.size)
    .reduceLeftOption((a,b) => if (a._2 > b._2) a else b)   // an option
    .map(_._1)
    .foreach(println)

The above code finds the most popular element in the list. In this example the
most popular element is 3 because it appears 4 times. I've defined a chain of
transformations to find this value. The most interesting thing about that is
that the type I'm operating on changed three times, but the chain still looks
uniform! We start with a `List[Int]` after `groupBy` we have a `Map[Int,
List[Int]]`. Then we replace the lists with just their sizesp so after
`mapValues` the type is `Map[Int, Int]`. With `reduceToOption` we find the
element of the map that has the biggest value and create `Option[(Int, Int)]`.
Then we just get the first element from the tuple (this is our number - the
second Int in the tuple tells how many times it occurred in list) with `map`
and the final type is `Option[Int]`. Last operation is just printing the value
(if found) to the standard output.

You may have noticed that I didn't use the `flatMap` method. In fact every
method that I used above CAN be implemented using one or more `flatMaps`. Let's
take simple `map` for example:

    m map f == m flatMap (x => unit(f(x)))

These methods are just some operations that are useful and were given a name.
Of course they are not implemented with `flatMap` in Scala because it would
hurt the performance, but the point is that they are the result of those types
(List, Map, Option) being a monads.

What monads gives in scala (for-expressions) (translating for-expressions into
flatmaps and withFilter)
