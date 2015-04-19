---
title: Hidden Functions
date: 2015-04-09 18:22:23
tags: ["seq", "map", "functional programming"]
categories: programming scala
---

Scala classes can extends functions:

{% highlight scala %}
class Fun extends (Int => String) {
  def apply(i: Int): String
}
{% endhighlight %}

Scala's `Seq` and `Map` are functions.

{% highlight scala %}
val letters = Seq("a", "b", "c", "d")
val result = Seq(1, 2, 1).map(letters)
{% endhighlight %}

This would result in `Seq("a", "b", "a")`
