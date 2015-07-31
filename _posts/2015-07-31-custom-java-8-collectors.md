---
title: Custom Collectors in Java 8
date: 2015-07-31 08:20:10
tags: ["collector", "stream", "java 8"]
categories: programming java
---

Java 8 sure did bring a few interesting features. One of them are definitely the streams. Internet is
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
First one (_T_) is the type of a stream we want to collect. For example for `Stream<String>`
the _T_ is `String`. Second (_A_) is the accumulator type. This is the type that is used to store
values while processing the stream. Finally the _R_ is returned type. This is what the collector
actually returns.

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
* UNORDERED - Means that the order of elements is not important. This information can be used to
optimize processing.

# Implement, all the collectors!

Well, maybe let us choose one collector that we would like to have. Recently I've stumbled upon the
problem of finding the most popular element in a collection. By most popular I mean the element that
occurs the most. So in `a, a, a, b, b, c` the most popular is obviously `a` because it occurs three
times.

In terms of a collector. We want to collect the stream of type _X_ to single value of type
_Optional&lt;X&gt;_. This value should be the most popular item. We use optional here because if we supply
empty list then we do not have any most popular item there. As the accumulator type I suggest using
`Map<X, Integer>` to store elements with their occurrence count. I'll explain this in detail later.
So the collector generic parameters should be `Collector<X, Map<X,Integer>, Optional<X>>`

To implement the collector we can simply implement the interface:

{% highlight java %}
public class MostPopular<T> implements Collector<T, Map<T, Integer>, Optional<T>> {
  // all the methods listed for collector above
}
{% endhighlight %}

Now we can just implement the functions. Let's start with simple accumulator supplier:

{% highlight java %}
@Override
public Supplier<Map<T, Integer>> supplier() {
  return HashMap::new;
}
{% endhighlight %}

Here we simply return the reference to the `HashMap` constructor as the supplier. Whenever the
supplier method is called new `HashMap` will be created.

Moving on, to the accumulator:

{% highlight java %}
@Override
public BiConsumer<Map<T, Integer>, T> accumulator() {
  return (acc, elem) -> {
    Integer value = acc.get(elem);
    acc.put(elem, Optional.ofNullable(value).orElse(1));
  };
}
{% endhighlight %}

This method should be used to add another element to our accumulator. In this implementation we
simply fetch the occurrence count for this element from the map. If the value is `null` then this is
the first occurrence and we set the value to 1. If the value is other than `null` then we simply add
1 to this value, and lastly we store the value back in the map.

This method gets called for every element. This means that after all elements are processed by this
method we end up with the accumulator that have the mapping from the element to its occurrence
times.

Next on the list is the combiner method:

{% highlight java %}
@Override
public BinaryOperator<Map<T, Integer>> combiner() {
  return (acc1, acc2) -> {
    throw new UnsupportedOperationException();
  };
}
{% endhighlight %}

Well... this is not what you expected at all. I can tell! Let me explain myself. The combiner method
is used when the process can be parallelized. This method is here to merge processing results from
different threads. Every thread gets part of the stream to collect, and in the end all the
resulting accumulators are merged by this method. In this particular collector the implementation
for this method can be a bit tricky, but is not crucial so I just let it go :)

This method is not used unless you create `parallelStream()` instead of `stream()` from the
collection.

Let's almost finish with finisher method implementation:

{% highlight java %}
@Override
public Function<Map<T, Integer>, Optional<T>> finisher() {
  return (acc) -> acc.entrySet().stream()
    .reduce((a, b) -> a.getValue() > b.getValue() ? a : b)
    .map(Map.Entry::getKey);
}
{% endhighlight %}

Here we want to extract the final result from accumulator.
This implementation is pretty straight forward if you are used to the stream-way of processing data.
We have our accumulator `acc` which is a map of elements with their occurrence count. We create a
stream from this map's entry set and using reduce we find the value of type `Optional<Map.Entry<T,
Integer>>` that represents the entry with the biggest occurrence value.

As here we have got whole `Map.Entry<T, Integer>` but need only the key from the entry we simply use
`map(Map.Entry::getKey)` on the optional value. This gives us exactly what we want - `Optional<T>`
with the most popular value.

Last method that we have to implement is the `characteristics()` method:

{% highlight java %}
@Override
public Set<Characteristics> characteristics() {
  return Collections.emptySet();
}
{% endhighlight %}

Not very exciting but does the job. I guess we could use `UNORDERED` here as well.

Finally you can see whole implementation [here][code]. Having this collector we can simply use it to
find the most popular element in collection of any type:

{% highlight java %}
Lists.newArrayList(1, 1, 2, 2, 2, 3, 4, 5, 5)
  .stream().collect(new MostPopular<>());

Lists.newArrayList('a', 'b', 'c', 'c', 'c', 'd')
  .stream().collect(new MostPopular<>());
{% endhighlight %}

#Summing up

Collectors are in fact a Java way of defining custom [folds][folds]. As it's the Java way it has to
be a little verbose ;) Anyway - folds are great thus collectors are great too! Thanks for reading!

[code]: http://pastebin.com/k5QPqn7Q
[folds]: https://en.wikipedia.org/wiki/Fold_(higher-order_function)
[source]: http://www.nurkiewicz.com/2014/07/introduction-to-writing-custom.html


