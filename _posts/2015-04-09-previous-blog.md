---
title: Why I abandoned my old blog?
date: 2015-04-09 17:47:55
tags: vim jekyll editing
categories: blog
---

This blog has very little content, but I'm not new to blogging at all.  In fact [I've been
publishing my thoughts and ideas online for a few years now][blog]. The only problem is that I've
got tired of it. I don't mean blogging.  I mean _writing_ posts.

I can hear you ask "What is the difference?". Well... I was using [Wordpress][wordpress]. This
software is great for blogging, but sometimes I wanted more. I wanted to insert my code into pages,
create custom themes and extensions etc.  Of course with tools like [Heroku][heroku] I could host my
own instance of Wordpress and create themes or extensions myself, but I don't really like PHP and
Wordpress theming is not something I wanted to learn.

Then there is also Vim. After I started using it I really didn't want to edit text in the browser
ever again. Creating content with online editor wasn't (and still isn't) the best way to spend your
evenings - especially if you hit `Esc` key all the time...

So I stopped blogging. I liked to share the ideas and get feedback from readers, but I needed some
motivation to get that going again - and Wordpress wasn't helping with that ;)

## The repair program

I've started developing my own blog with Scala and [Play Framework][playframework]. You can view
it's [source code on my GitHub][scala-blog]. While doing that I've learned a lot of Scala. It was
great experience and I'm very happy that I did that! But when the page was up and running - I still
didn't want to write posts because the editor was even worse! Posts had to be written in Markdown so
I could edit them locally (using Vim) and then copy the text to the page. I wasn't happy with that
solution either.

## Salvation

Lately I've decided to check out what are [GitHub Pages][gh-pages]. This was the best idea I've had
for a while :) Through that I discovered [Jekyll][jekyll]. I wasn't aware that there exists
  something like _static page generator_. It turned out to be perfect solution for me!

Now I can create my posts and pages using any editor I want and any markup language I like. Content
is versioned in git repository. Pages are hosted by GitHub and deployment is as easy as pushing
changes to remote branch. That's just perfect :)

## The language

If you checked out [my previous blog][blog] you may have noticed that I used my native language
there. I decided to change that because English has just bigger audience. Watching page statistics
I've discovered that some Google searches got foreign people to my page and the content could
probably help... if they could read them :)

[wordpress]: http://wordpress.com
[heroku]: http://heroku.org
[playframework]: http://playframework.com/
[scala-blog]: https://github.com/marad/blog
[gh-pages]: https://pages.github.com/
[jekyll]: http://jekyllrb.com/
[blog]: https://moriturius.wordpress.com/
