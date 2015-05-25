---
title: Custom Collectors in Java 8
date: 2015-05-25 20:57:56
tags: ["collector", "stream", "java 8"]
categories: programming java
---

Java 8 sure bring a few interesting features. One of them are definitely the streams. Internet is
full of the instructions on how to create and use them. Today I'd like to talk about something a
little bit different - collectors.

# What is a collector?

Streams can be used to process collections. The way it works is that we first define all
transformations on the collection with non-terminal operations like `map`, `flatMap`, `filter`, etc.
Calling these functions on the stream only defines the transformations but doesn't run them unless
there is a terminal operation called at the end.

There are few basic terminal operations like `sum`, `count`, or `reduce` but there is also more
generic method: `collect`. This method takes a _collector_ and returns whatever that
collector returns.

The purpose of the collector is to compose the data from the stream into one another type.

# How does the collector look like?

Let's see the collector interface:

{% highlight java %}
interface Collector<T,A,R> {
    Supplier<A>          supplier()
    BiConsumer<A,T>      accumulator()
    BinaryOperator<A>    combiner()
    Function<A,R>        finisher()
    Set<Characteristics> characteristics()
}
{% endhighlight %}

As you can see the collector is a generic type that has type parameters named _T_, _A_, and _R_.
First one (_T_) is the stream type. For example for `Stream<String>` the _T_ is `String`. Second
(_A_) is the accumulator type. This is the type that is used to store values while processing the
stream. Finally the _R_ is returned type. This is what the collector actually returns.

This means that you can collect `Stream<T>` to value of type `R` using some helper variables of
type `A` while collecting.

The `supplier()` method returns a function (!) that is used to create our accumulator. This is
important here. All this methods (except for the `characteristics()`) return functions.

Next up is method named `acumulator`. This returns a function that takes our accumulator (of type
_A_) and a value from a stream (of type _T_) and combines them. For example if _A_ would be of type
`List<T>` then we could just add the element to the list here. This is exactly what
`Collectors.toList` does :)

The `combiner()` is a method that returns a binary combiner for our accumulator. This means that the
returned function should take two arguments of type _A_, combine them, and return new value of
type _A_. Continuing the example for lists this would simply merge two lists together into one.

The `finisher()` method returns a function that extracts the final result of type _R_ from
intermediate accumulator. Our example doesn't help much here because for lists the accumulator type
and returned type could be the same and `finisher()` could simply return the final accumulator
value. It's not hard to imagine tough, that we could use some kind of a container while processing
the collection and finally extract the result from the container. This is the function for the job.

Finally the `characteristics()` method. This returns a set of characteristics that the collector
has. This is used to optimize collecting the stream. We can return there a set of values from
`Collector.Characteristics` enum. The values are:

* CONCURRENT - This tells that the collector can be used with multiple threads.
* IDENTITY_FINISH - This indicates that the intermediate type _A_ is the same as returned type _R_
so there is no need to call `finisher()` method.
* UNORDERED - Means that the order of elements is not important.

What it gives us?
How to create a collector?

bibliography
http://www.nurkiewicz.com/2014/07/introduction-to-writing-custom.html
