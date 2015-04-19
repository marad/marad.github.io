---
title: Optional in Java 8
date: 2015-04-19 19:06:46
tags: ["optional", "functional programming"]

categories: programming java
---

Recently I've been a part of self-educating team. My workmates and me decided to learn Java 8
features so we could use it at work. Each of us have one or two topics to cover and we had to teach
each other about the cool new features like streams or better type inference. My part was to
cover the `Optional` class, and today I'd like to show and explain how and why you should use it.

The `Optional` class is a very underestimated addition in Java 8. [Some people may have a feeling
that `Optional` is pointless][bad-optional]. In fact if you have function that may or may not
produce some result you should use `Optional`. Even using it as shown in linked post gives you some
benefit. Method signature tells you that when called it may return a value or nothing. No need to
read the docs to find out where your `NullPointerException` is coming from.

Author of the post tries to understand why `Optional` fixes the problem with `NullPointerException`
while giving the examples that it causes more problems than it solves. This is mostly because he's
using this class in a way he used to work with `null`s - treating `Optional` as a value. You should
treat `Optional` more as a container for value and **NEVER** make a method that returns `Optional`
that is `null`.

Doing this already fixes most NPEx'es but why stop here? By properly using optionals we can achieve
cleaner and more readable code.

Assume we've got a library. Library has books, and each book should have an author. We know for a
fact though that some books in our library doesn't have an author assigned. So we've got structure
along the lines of:

{% highlight java %}
class Author {
  private String name;
  // ...
}

class Book {
  private String title;
  private Author author;

  public Optional<Author> getAuthor() {
    return Optional.ofNullable(author);
  }
  // ...
}

interface Library {
  public Optional<Book> findBook(String title);
}
{% endhighlight %}

Now that we have our model defined let's think about finding author by book's title. Normally we
would have to find book by title, and then check if it's not null and then get the author and get
check if that is not null. Finally we could do what we want with the author object. So without
optionals it would look like this:

{% highlight java %}
Book book = library.findBook("Some Title");
if (book != null) {
  Author author = book.getAuthor();
  if (author != null) {
    doStuff(author);
  }
}
{% endhighlight %}

With optionals we can't get the `null` so that's already a gain. Well - we could but one should
really have no idea about programming to achieve this in real life. But the code isn't that great at
all. I'd say that it looks even worse:

{% highlight java %}
Optional<Book> book = library.findBook("Some Title");
if (book.isPresent()) {
  Optional<Author> author = book.get().getAuthor();
  if (author.isPresent()) {
    doStuff(author.get());
  }
}
{% endhighlight %}

Still needs some improvement. There is a method in `Optional` called `ifPresent()` that takes a
function and invokes it on the inner object if the object is, well, present. Using this method we
could make it a bit better:

{% highlight java %}
library.findBook("Some Title")
  .ifPresent(book -> {
    book.getAuthor().ifPresent(this::doStuff);
  });
{% endhighlight %}

But as Hugues states - this is still not entirely readable. What he didn't mention is the fact that
`Optional` has also another method that give it it's awesomeness. Meet the `flatMap`:

{% highlight java %}
library.findBook("Some Title")
  .flatMap(Book::getAuthor)
  .ifPresent(this::doStuff);
{% endhighlight %}

What does the `flatMap` do? If the value of `Optional` is present then it invokes the function
passed as parameter on that value. If value is not present - `flatMap` returns empty `Optional`. The
function must return an `Optional` which is in later returned by the `flatMap`.

In this case we pass the `Book::getAuthor` which is a short for `book -> book.getAuthor()`. The
`getAuthor()` method returns `Optional<Author>` which is returned further by the `flatMap` so we can
invoke `ifPresent()` method to operate on `Author` object - if it's present.

Please note how this notation hides all implementation details from us. We don't really care if
something is `null` or if something has gone wrong. We only care that if everything is fine we
just want to invoke some operation on the `Author` object.

You may be saying now: "This is not more readable at all!". You're right but your statement is
false. What do I mean? Well `flatMap` is something that one should get used to when playing with
functional programming because it's one of the most used functions there. Lambdas, method
references, functional interfaces and better type inference (which still needs some love) introduced
in Java 8 is nothing more than inserting functional language properties into Java.

I mean that when you get the hang of it, it'll be more readable to you.

In fact `flatMap` is something that has it's origins in category theory as the part of _monad_
disguised under the name of `bind` operation. The category theory isn't required to use monads or
`flatMaps`. You don't really have to know how the engine works to drive a car, do you?

What are monads? I believe that is something to cover in following posts. Thanks for reading!

[bad-optional]: http://huguesjohnson.com/programming/java/java8optional.html
