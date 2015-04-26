---
title: No one expects the Spanish functions
date: 2015-04-26 15:07:45
tags: ["seq", "map", "functional programming"]
categories: programming scala
---

Scala, as a functional language, treats functions as first class citizens. Of course it's also
object oriented language. Those two paradigms can be seen as conflicting with each other. In this
post I'd like to show you some different ways of creating a functions in Scala to show that these
two worlds can coexist.

# Foreword

## Function
Let us start by defining what a function is. Wikipedia supplies us with this:

> A function f from X to Y is a subset of the Cartesian product X × Y subject to the following
> condition: every element of X is the first component of one and only one ordered pair in the
> subset.

This is of course mathematical definition of a function. It says that given two sets `X` and `Y` we
define `f` as some kind of mapping from `X` to `Y`. Also we have to define such mapping for all
elements from `X`, and each element from `X` have only one mapping to any element from `Y`.

So let's get back to the programming world. We have functions here, but our functions are not quite
like the definition says. We can make the function be non-deterministic which means that it can
basically return different values when called with the same inputs twice. Those functions have some
kind of side effect. Reading data from network with `receive` function is a good example. We always
call it with the same arguments, yet it always returns different data.

But as it turns out - if we create a function that operates **only** on its arguments it's
impossible to make it return different values for the same arguments.

Functions that operate only on their arguments are called _pure functions_. We all should always use
pure functions, because they are easy to reason about.  Unfortunately sometimes we have to interact
with outside world and this is mostly done through side effects.

## Partial function

Partial function is a function that is defined only for some elements of `X`. This means that
there are some elements of `X` that cannot be supplied as an argument to our function - it would
give us an error.

{% highlight scala %}
def partial(input: Int) = match {
  case 1 => "One"
}
{% endhighlight %}

This example function will work only for one argument: `1`. Any other value would cause an error.

## Higher order function

One last thing we should know before going further is: what is higher order function? Well it's just
a function that operates on other functions. It can receive a function as an argument or it can
create a function and return it as a value.

{% highlight scala %}
def apply(f: Int => String, arg: Int) = f(arg)
{% endhighlight %}

This simple higher order function just takes two parameters - a function and a number. Then it
applies the `f` function to the number.

# Define a function in Scala

To define a function or method in Scala we normally write:

{% highlight scala %}
def greet(name: String): String = s"Hello $name"
{% endhighlight %}

Then we can use this function as a parameter to other function:

{% highlight scala %}
Seq("Bob", "Ben").map(greet)
{% endhighlight %}

Here `map` is a higher order function which translates each `Seq` element to new element using the
function that was supplied as its argument. Result will be: `Seq("Hello Bob", "Hello Ben")`.

In Scala there is a special method that we can implement in our class or object that gives it
function-like invocation:

{% highlight scala %}
object Greeter {
  def apply(name: String) = s"Hello $name"
}

Greeter("Ben") // will return "Hello Ben"
{% endhighlight %}

So the `apply` method gives us some syntactic sugar in Scala - nice! But if we can use it as a
function, can we pass it to the higher order function like `map`?

{% highlight scala %}
Seq("Bob", "Ben").map(Greeter)
{% endhighlight %}

This will unfortunately throw an error at compile time saying that it wanted a function `String =>
String` and got `Greeter`.

The good news is that we can fix this! We must tell the compiler that our object is also a function
by extending it's type signature:

{% highlight scala %}
object Greeter extends (String => String) {
  def apply(name: String) = s"Hello $name"
}
{% endhighlight %}

Now we can pass our `Greeter` object to `map` and everything works fine!

As you can see Scala merges the world of object oriented programming and the world of functional
programming by allowing objects and classes to act like functions.

# Sequences and Maps

As I mentioned above - functions are mappings. This means that we could define a new function just
by defining a mapping between two sets of data. This is why Scala's `Map[A, B]` extends the `A => B`
type signature. It basically allows us to use any `Map` as a function from type `A` to type `B`:

{% highlight scala %}
val numbers = Map(1 -> "one", 2 -> "two", 3 -> "three")
val result = Seq(3, 2, 1).map(numbers)
{% endhighlight %}

This will give the `result` a value of `Seq("three", "two", "one")` so map acts exactly as partial
function. It's partial because it's defined only for values 1, 2, and 3.

And what about other collections? Well, a sequence maps index to a value, doesn't it?

{% highlight scala %}
val letters = Seq("e", "h", "l", "o")
val result = Seq(1, 0, 2, 2, 3).map(letters).mkString
{% endhighlight %}

The `result` value will be _"hello"_.

# Function composition

Mapping over some sequence is fine to show that lists and maps are functions but you may say that it
doesn't really give us much because after all we can write it like this:

{% highlight scala %}
Seq(1, 0, 2, 2, 3).map(num => letters(num))
{% endhighlight %}

The true fun begins when you want to operate on some functions. For example we would like to compose
two functions. This means that we apply the first function to the argument and then apply the second
to the value returned by the first: `f(g(x))`.

This can be also done with maps and sequences:

{% highlight scala %}
val englishToInteger = Map("zero" -> 0, "one" -> 1, "two" -> 2, "three" -> 3)
val integerToGerman = Seq("null", "eins", "zwei", "drei")
val englishToGerman = englishToInteger andThen integerToGerman

val zwei = englishToGerman("two")
{% endhighlight %}

Of course value of `zwei` will be a string _"zwei"_. The `andThen` is a method of
[Function1][scala-function] class and it composes two functions. Here we create the function
`englishToGerman` by composing `englishToInteger` with `integerToGerman`. The argument flow is quite
straightforward here - _"one"_ is passed to the `englishToInteger` function (which is a Map!). This
gives us an integer representing the number: `1`. Then this integer is passed to `integerToGerman`
sequence which maps indexes to German numbers.

# Summary

Even though I've been using Scala for some time now I've discovered that maps and lists act as
functions very recently. What is great here is that knowing that gives me a new perspective on those
data structures. I hope that for you this is as much fun as it is for me.

Thanks for reading!

If you are still wondering what's with the title [watch this][spanish-inquisition].

[scala-function]: http://www.scala-lang.org/api/current/index.html#scala.Function1
[spanish-inquisition]: https://www.youtube.com/watch?v=7WJXHY2OXGE

