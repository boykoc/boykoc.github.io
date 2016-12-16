---
layout: post
title:  "NameSpaces and Modules in Rails"
date:   2016-12-16 11:40:00 -0400
categories: ruby rubyonrails
---
## Overview
I've read a few different posts and articles on how to organize your Rails app codebase. Most eventaully reference `modules` and namespaces as one good way to do it. 

A lot of the basic examples show taking a module (or controller or view ) directory that looks like this:

{% highlight bash %}
models/
  foo.rb
  bar.rb
{% endhighlight %}

And grouping the children (loose term) under the parent to bring connected code together (namespace or module).

{% highlight bash %}
models/
  foo.rb
  foo/
    bar.rb
{% endhighlight %}

Then you can update bar.rb from this:

{% highlight bash %}
class Bar < ActiveRecord::Base
end
{% endhighlight %}

to:

{% highlight bash %}
class Foo::Bar < ActiveRecord::Base
end
{% endhighlight %}

And this does work. Yay! (but don't get too excited)

The above made me think that declaring a `module` and a namespace are 100% the same thing. I could write either of the following and I'd get the exact same result.

{% highlight bash %}
class Foo::Bar
  # Compact style.
end
{% endhighlight %}

is the same as:

{% highlight bash %}
module Foo 
  # Nested declaration.
  class Bar
  end
end
{% endhighlight %}

##When $%^& Hit the Fan (ok it wasn't that bad)
The thing that started to confuse me and made me question what I was doing is RuboCop was telling me to use nested definitions instead of the compact style. So I tried changing `class Foo::Bar` to `module Foo ...` and then I recieved an error saying `Foo is not a module` when I loaded my webpage.

When I started re-reading the comments in the blog post and the accepted answer on StackOverflow I realized what was happening.

In this case `class Foo::Bar` is looking for `Foo` to be defined and it finds it as a `class` so everything works. Once you try to break it out and declare `Foo` as a `module` it breaks because `Foo` is a `class`. So, if I wanted to keep the folder structure as is I could use the nested delcaration as follows:

{% highlight bash %}
class Foo
  class Bar
  end
end
{% endhighlight %}

**And Bam, it works!** But am I really wanting to nest my classes?

What we're really doing here is nesting classes and using it as a namespace and calling it a `module`. But it's nesting classes to give reference to us (developers) as to how things are connected together. To me personally, I feel this is what `modules` do a little better, at least in this case.  Kind of like saying, "Hey, these classes aren't nested I just think they belong together in various ways so lets give them a common name."

I had been trying to figure out what the difference between namespaces and modules are in Rails (and by extension Ruby). I had started to think they are the same and interchangable. But now, I'm looking at it like this: NameSpaces are a way to reference related things demonstrating hierarchy. There are different ways to accomplish namespacing (i.e. nesting classes, nesting modules, nesting modules and classes). The NameSpace does not define the type of things being nested/grouped.

Another interesting piece with the above `Foo::Bar` example is that Foo must already be defined when calling `Foo::Bar`. When you call `Foo::Bar` you are just looking it up. If `Foo` hasn't been previsouly defined, you'll get an error. In the first case you're defining `module Foo`. The second is more of a reference. Also, you can't define a `class` and a `module` with the same name.

## Where I Landed
So for me, what I'm going to start trying is the following:

{% highlight bash %}
models/
  foo.rb
  bar.rb
{% endhighlight %}

And grouping the children (loose term) under the parent to bring connected code together (namespace or module).

{% highlight bash %}
models/
  foo.rb
  foos/
    bar.rb
{% endhighlight %}

Then you can update `bar.rb` from this:

{% highlight bash %}
class Bar < ActiveRecord::Base
end
{% endhighlight %}

to:

{% highlight bash %}
module Foo
  class Bar < ActiveRecord::Base
  end
end
{% endhighlight %}

I'm doing this because I like the nested declaration of a `module`. I also feel that in these cases I'm not wanting to nest a `class` but group classes under a common area. To me a `module` seems to be appropriate. Maybe I'll change my mind in the future, but at least I've thought it through and am understanding what I'm doing a bit more vs. blindly jumping in.
