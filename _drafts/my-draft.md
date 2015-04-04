---
title:  "My Simple Post"
date:   2015-04-04 17:04:00
categories: first jekyll site
tags: hello world
---

This is sample post which is shown here and there!

{% highlight scala linenos %}
class C1 extends Component
{% endhighlight %}

{{ page.date | date_to_string }}

{{ page.tags | jsonify }}

```scala
class C1 extends Component
```

