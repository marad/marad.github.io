---
title: The Mighty Monad
date: 2015-05-31 11:52:40
tags: ["monad"]
categories: programming scala
---

When you begin to learn functional programming everybody is telling you that pure functions with no
side effects are the only way to go. This is fine up to the point. Sooner or later you will hear
about monads. This post tries to explain what the monad is and why you should use it.

# Prerequisites

To fully understand the examples here you should know at least basics of Scala language. You should
also know [how to use `Option[T]`][scala-option] type in Scala or at least know the Java's `Optional<T>`
which I described [here][java-optional]. In the section about pure functional languages I used a bit of
Haskell, but I tried to make the examples as clear as I could for those who don't know it.

# The definition

<p class="notice">This is raw definition of monad. Please note that you don't have to fully
understand it to read the following sections. You can safely skip to the <em>Simpler definition</em>
section. Having said that, I encourage you to face this definition. It may help you later on.</p>

Let's start with something not simple at all! The monad definition:

We can say that _M_ is a monad when:

* it is generic type `M[T]`
* there is a _unit_ function `T => M[T]`
* there is a _flatMap_ function `(M[T], T => M[T]) => M[T]`

To make things even worse, there are three monad laws:

* Left identity: `unit(x) flatMap f == f(x)`
* Right identity: `m flatMap unit == m`
* Associativity: `(m flatMap f) flatMap g == m flatMap (f flatMap g)`

Let me clarify this definition a bit. Leaving the laws for later let us jump back to the _M_ type
and the _unit_ and _flatMap_ functions.  Type _M_ is just regular generic type like `Option[T]` or
`Try[T]`.  The _unit_ function takes the value of type _T_ and wraps it with a monadic type. For
`Option[T]` the _unit_ operation is simple `Some(t: T)`.  Lastly - the _flatMap_. Literature refers
to it as the _bind_ operation. I decided to call it _flatMap_ here so it sounds more familiar. It,
of course, takes a monadic value `M[T]` and a function `T => M[T]` and returns another `M[T]`. This
basically means that we can do some computation on value that is inside the monad and create a new
monad with the result of that computation. Scala is object oriented so `flatMap` is just a method
and takes one parameter (the function) the other being just _this object_.

To sum this up with an example based on type `Option[T]`:

* we have got generic type `Option[T]`
* we have _unit_ function `Some(t: T)`
* we have _bind/flatMap_ method `Option.flatMap(f: T => Option[T]): Option[T]`

Going back to the monad laws. The first two laws are fairly simple and describe the relations
between _unit_ and _flatMap_. The _associativity_ law tells us just that the order of the _flatMap_
doesn't matter. We can write the laws for `Option[T]` as following:

* `Some(t) flatMap f == f(t)`
* `opt flatMap Some[T] == opt`
* `(opt flatMap f) flatMap g == opt flatMap (t => f(t) flatMap g)`

To prove that the laws hold we should replace our methods with their implementations to see if we
can end up with the other side of equation. This could be a topic for whole new post (and maybe will
be) so I'll skip this for now.

# Simpler definition

The reality is that these monad laws are mainly important when you want to implement a monad
yourself or you need to rely on one of these properties in your code. In fact when talking about
monads most developers are thinking just about generic type _M[T]_ with _unit_ and _flatMap_
operations.

There is even more! There are types that we call a monads but they do not satisfy those laws! Let's
take `Try[T]` for example. This monad is used to deal with possible exceptions that might occur
while processing. If you have some operation `expr: T` that can throw some exception you
may want to wrap it with `Try(expr)`. This will return `Success(t: T)` containing the result of the
function or `Failure(ex: Throwable)` with the exception that was thrown by that function. If you
closely examine all monad laws you can see that the _left identity_ law does not hold.

{% highlight scala %}
unit(x) flatMap f == f(x)
{% endhighlight %}

This tells us that following should be true:

{% highlight scala %}
Try(expr) flatMap f == f(expr)
{% endhighlight %}

The law works fine if everything goes smooth and no exception is thrown. The problem pops up when
either `f` or `expr` throws an exception. The left hand side `Try(expr) flatMap f` never throws an
exception and just returns `Failure(ex)`. The right hand side `f(expr)` will just throw the
exception so the law does not hold thus `Try[T]` is not precisely a monad, but that is not a problem
for us. We are not mathematicians (no offense ment!). We just want things to work :)

# So what does all this means for developers?

Monadic structure gives us, developers, a uniform way of defining a chain of transformations on
virtually any type. Just look at this:

{% highlight scala %}
List('a', 'a', 'b', 'c', 'c', 'c', 'c', 'd', 'd', 'e')      // List[Char]
    .groupBy(identity)                                      // Map[Char, List[Char]]
    .mapValues(_.size)                                      // Map[Char, Int]
    .reduceLeftOption((a,b) => if (a._2 > b._2) a else b)   // Option[(Char, Int)]
    .map(_._1)                                              // Option[Char]
    .foreach(println)
{% endhighlight %}

The above code finds the most popular element in the list. In this example the most popular element
is `c` because it appears 4 times. I've defined a chain of transformations to find this value. The
most interesting thing about that is that the type I'm operating on changed three times, but the
chain still looks uniform! We start with a `List[Char]` after calling `groupBy` method we have a
`Map[Char, List[Char]]`. Then we replace the lists with just their sizes so after `mapValues` the
type is `Map[Char, Int]`. With `reduceToOption` we find the element of the map that has the biggest
value and create `Option[(Char, Int)]`. Then we just get the first element from the tuple (this is
our most popular char) with `map` and the final type is `Option[Char]`. Last operation is just
printing the value (if found) to the standard output.

We used three different monads: List, Map, and Option. Every line of code changed the output type.
Yet we still could invoke new transformations like we didn't care! I personally think that this is
fantastic :)

You may have noticed that I didn't use the `flatMap` method. In fact every method that I used above
CAN be implemented using one or more `flatMap` calls. Let's take simple `map` for example:

{% highlight scala %}
m map f == m flatMap (x => unit(f(x)))
{% endhighlight %}

These methods are just some combinations of `flatMap` that are useful and were given a name.  Of
course they are not implemented with `flatMap` in Scala because it would hurt the performance, but
the point is that they are the result of those types (List, Map, Option) being a monads.


# Word about functional programming

<p class="warning">This part contains dangerous amounts of Haskell.</p>

Scala is kind-of functional programming language. You can write some parts functionally and other
imperatively. This is great because by mixing the styles we can end up with code that is both
readable and concise. Monads have another side which is extremely important for pure functional
languages like Haskell.

## The order

You see, in pure functional languages you cannot define the order of operations. You can just define
some equivalences. What do I mean? Take a look:

{% highlight scala %}
def add(a: Int, b: Int) = a + b
{% endhighlight %}

We are used to call this a function definition but what really happens here? We just tell that the
`add(x,y)` string can be replaced with `x + y` string on the code level. In fact we could evaluate
pure functional code just by replacing strings!

Going back to the monads. In pure functional languages to define the order of computation you would
have to invoke a function on a call to function etc.:

{% highlight scala %}
f(g(h(x)))
{% endhighlight %}

You can see that here we have to evaluate `h(x)` then pass the value to `g` and finally to `f`. Try
to imagine bigger program written like that. Well yes, it would be unreadable!

Now we could scream: _Monads to the rescue!_. But let's not. If you look again at the code to find
the most popular element in list you can see that we strictly defined the order for the operations.

This is exactly what we use in Haskell to pretend that we are doing imperative code. Of course
Haskell has some syntactic sugar on top of it so instead of writing `flatMap` (which in Haskell is
`>>=`) you can write it imperatively-ish. The `do` notation:

{% highlight haskell %}
do
  x <- thing1
  y <- func1 x
  thing2
  z <- func2 y
{% endhighlight %}

What is going on here? The `thing1` and `thing2` are monadic values (we can call `flatMap` on them).
The `func1` and `func2` returns a monadic values. What is going on here is that first we get the
value form `thing1` and name it `x`. Then we pass it as a value to the `func1` which returns another
monad. Then we take value from the monad and name it `y`... You see where this is going. This
clearly defines the order of operations.

Example above is exactly the same as:

{% highlight haskell %}
thing1 >>= (\x -> func1 x >>= (\y -> thing2
       >>= (\_ -> func2 y >>= (\z -> return z))))
{% endhighlight %}

You see that the `do` notation is a bit more readable :) This example is taken from [Haskell
Wiki][do-notation].


## The state and side effects

Another thing is that in Scala we can talk about mutable and immutable state. In pure functional
programming there is no state at all! There are only arguments passed to functions. That's the
closest thing to state you can get.

But there is state in the world. Our hard drives have state. Keyboard has state. There is a lot of
state everywhere! How to deal with this in Haskell? You guessed it - monads. Lets look at this quick
example:

{% highlight haskell %}
main :: IO ()
main = do
         c <- getChar
         putChar c
{% endhighlight %}

This program, as you might suspect, reads one character from the standard input and writes it to
standard output. To put it simply: awaits for keyboard button to be pressed and prints the letter to
the console.

Here you can see that `getChar` does have some kind of state as the value seems to materialize from
nothing - it doesn't expect any arguments. So what happens here? Well - the input/output operations
are wrapped with a monad which acts here as a gate between our stateful world and the world of pure
functions.

We can rewrite it using `flatMap` (which in Haskell is `>>=`):

{% highlight haskell %}
main :: IO ()
main = getChar >>= putChar
{% endhighlight %}

In Scala it would look like:

{% highlight scala %}
def main(): Unit = getChar.flatMap(putChar)
{% endhighlight %}

The `getChar` is a monad. If we invoke `flatMap` on it gives us a key pressed on the keyboard as a
parameter to our function. Our function here is `putChar`. It takes one char and returns a monad
back. The returned monad is empty (like `Unit` in Scala) so the value is not interesting. The
`putChar` function does something else behind the scenes. It writes the character to the standard
output. This is side effect that we wanted. The ability to talk to stateful world.

All this is thanks to humble `flatMap` :)

# Summary

Thanks for reading! I hope that you will not be frightened by the _monad_ word anymore! These are
useful little creatures. They are easy to use when you get the hang of them, but quite hard to learn
(and explain!). I really hope that this post was helpful to you because when I wanted to learn
monads the first time I couldn't understand a single thing about them. Then something just _clicked_
and everything was clear. I hope that it just _clicked_ for you today :)

If you have any questions please leave a comment below!

[do-notation]: https://wiki.haskell.org/Monad
[scala-option]: http://www.tutorialspoint.com/scala/scala_options.htm
[java-optional]: /programming/java/2015/04/19/java-optional.html
