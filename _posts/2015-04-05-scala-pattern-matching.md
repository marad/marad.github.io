---
title: Pattern Matching in Scala
date:   2015-04-05 21:54:00
tags: ["scala", "pattern matching"]
categories: programming
---

Not so long ago I gave a presentation about Scala and ScalaTest to my co-workers. This was just
basics as I wanted to give them a good and solid base for future researching this language. As it
turned out - I probably learned a lot more than they did from this :)

Whenever there is _Scala_ mentioned there also is the phrase _pattern matching_. Of course Scala
does support pattern matching but I didn't know how much fun one can have with this. In this post
I'd like to show a few fun things we can do with this.

## Simple matching

Let's start with something simple:

{% highlight scala %}
def showcase(input: Int) = input match {
  case 1 => "One"
  case 2 => "Two"
  case _ => "Other!"
}
{% endhighlight %}

This is the simplest case - we have value of some concrete type and we want to execute some code
depending on that value. This can be easily replaced with Java's `switch` instruction or bunch of
`if-else` instructions.

If you're new to Scala then you can be wondering what means the `case _` - this will match any other
value than listed above it. In fact it is important that this is at the very bottom `case` because
values are matched from top. If the value didn't match the first `case` then next is checked. If the
value doesn't match any case then exception is thrown. Also - function that is not covering all
possible inputs is called _partial function_.

OK, basic things are simple, but it's not that we couldn't write something like this easily in Java.
Let us try something a bit different:

{% highlight scala %}
sealed abstract class Base
class First extends Base
class Second extends Base

def showcase(input: Base) = input match {
  case f: First => "First"
  case s: Second => "Second"
}
{% endhighlight %}

First of all - we declare the _Base_ class. `sealed` keyword means that all direct subclasses of
this class are contained in the same file. Then we declare subclasses `First` and `Second`.

Having such class structure we can match the class of given `input`. This is of course something we
_CAN_ do in Java but it's not that elegant. We would have to write bunch of `isInstanceOf` and then
cast our input to target class. Casting means that we can mix something up and generate some casting
exceptions. Here - everything is taken care of and is type safe.

You may have noticed that I've not specified the `case _`. You might think that this means that the
`showcase` function is _partial function_ but it is not. In fact by using keyword `sealed` I made
sure that I know _EVERY_ direct implementation of `Base` class. So I know that input can only be
either of type `First` or `Second`. There is just no other option!

Speaking of options - exactly this trick is used with `Option[T]` type. It has two implementations -
`Some()` and `None`. There is no third option.

## Matching case classes

Assume that we have declared following class:

{% highlight scala %}
case class Post(id: Int, title: String, content: String)
{% endhighlight %}

Defining class as `case class` gives us some benefits for pattern matching:

{% highlight scala %}
def showcase(post: Post) = post match {
  case Post(1, _, _) => "post with id = 1"
  case Post(_, title, _) => title
}
{% endhighlight %}

As you can see it's quite different than matching the type. We get some `Post` as the input and then
test it with our matches.

On the second line we check if given post has `id == 1` and if it does then returns some descriptive
text. Note that we can tell the compiler that we don't really care for other parameters with `_`.

If the post's `id` is not 1 then match it's title to variable named `title` and return that
variable. Once again - we ignore inputs that we are not interested in.

But what if we want to check only certain parameter values and then have the whole object to play
with? Don't worry - Scala got you covered!

{% highlight scala %}
def showcase(input: Post) = input match {
  case post@Post(1, _, _) => post.content
}
{% endhighlight %}

Above example shows that we can match the value of `id` and if the value matches 1 we want the whole
object to be assigned to variable `post`. We can later use this variable as a reference to our
matching object. In this case we just return this post's content.

## Matching regular expressions

You can't say _matching_ without saying _regular expressions_, can you? Let's consider this simple
expression for matching dates:

{% highlight scala %}
val date = """(\d\d\d\d)-(\d\d)-(\d\d)""".r
{% endhighlight %}

Having that you can already match any string you want with:

{% highlight scala %}
"2015-04-05" match {
  case date(year, month, day) => s"It's year $year"
}
{% endhighlight %}

As you can see we use the variable name followed parentheses with all the matched groups from
regular expressions. Then we can use this groups as normal variables. In this case we just want to
match a year so maybe `month` and `day` could be replaced with `_`? Sure!

{% highlight scala %}
"2015-04-05" match {
  case date(year, _*) => s"It's year $year"
}
{% endhighlight  %}

Whoah! What happend here? Well... as we are not interested in all remaining groups we can ignore
them all with this simple notation `_*`. In this case it's not that different but if you had ten
matching groups you'd have to ignore them all explicitly. This way you can ignore them all together!

## Matching in `for` loops

Last thing I'd like to show you is that you can easily loop through lists of things that matches
your criteria:

{% highlight scala %}
val posts = List(
  Post(1, "Title 1", "Content 1"),
  Post(2, "Title 2", "Content 2"),
  Post(3, "Other Title", "Content 3"))

for( p @ Post(1, _*) <- posts) {
  // ...
}
{% endhighlight %}

This `for` loop will only execute it's body for the first post from the list. In this case we can
match only one value because the 1 is passed statically. What if we could combine different matching
styles to tell the compiler what we desire? Totally doable!

{% highlight scala %}
val pattern = "^Title.*".r
for( p @ Post(_, pattern(), _) <- posts) {
  // ...
}
{% endhighlight %}

What do we have here? We added simple regex pattern to match only titles that start with "Title".
Please note the parentheses after regex variable name. We haven't specified any names for matched
groups because our regular expression doesn't capture any groups. Empty parentheses are still
required. Otherwise scala would think that we want to catch the title and create varaible called
`pattern` with value of the whole title. Adding parentheses tell the compiler that this is regular
expression and that it should be used for matching.

## Summary

Pattern matching in Scala is a powerful tool. I'm certain that what is describe here is just the tip
of an iceberg, but I hope it gives you some useful tips for working with `match` keyword.

See you next time!
