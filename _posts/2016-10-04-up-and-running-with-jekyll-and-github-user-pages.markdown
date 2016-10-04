---
layout: post
title:  "Getting up and Running with Jekyll and Github User Pages!"
date:   2016-10-04 1:48:00 -0400
categories: jekyll update
---
## Overview
This was my second attempt at setting up Jekyll and after a few hours from start to finish I was done.

I managed to get it working durring attempt one but decided to delete everything and start from scratch to make sure I understood what I did.

## Here are my steps and setup:

*Note:* I set this up using  a VM on a Windows machine. To do this I had to setup port forwarding (I used port 4000 to 4000) and set the host when building and starting the local dev server.

Originally I saw a talk at WebU2016 about Jekyll and thought I'd try it. Next I checked out the [Jekyll docs][jekyll-docs] and the [Github docs][github-docs].

First, I create a git repo, go to it and look inside.

{% highlight bash %}
git init my-jekyll-site-boykoc`
cd my-jekyll-site-boykoc/`
ls
{% endhighlight %}

Then I created a Gemfile because there wasn't anything in the directory.

{% highlight bash %}
nano Gemfile
{% endhighlight %}

In the Gemfile I added the following lines to get the needed gem.

{% highlight bash %}
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins
{% endhighlight %}

Then install the gem and its dependencies.

{% highlight bash %}
bundle install
{% endhighlight %}


Then create a new Jekyll site.

{% highlight bash %}
bundle exec jekyll new . --force
{% endhighlight %}

This build replaces the Gemfile. You have to open it and comment out the `"jekyll", "3.2.1"` line and uncomment the `gem "github-pages", group :jekyll_plugins` line.

{% highlight bash %}
nano Gemfile
{% endhighlight %}

After you update the Gemfile you need to bundle update to update your Gemfile.lock.

{% highlight bash %}
bundle update
{% endhighlight %}

After this I added my Github origin. I had already created a new Github repo. For User pages to work you need to follow the instructions. Basically you need to make sure the report name is your `username.github.io`.

{% highlight bash %}
git remote add origin https://github.com/boykoc/boykoc.github.io.git
{% endhighlight %}

Check your status and add all files, commit then push.
{% highlight bash %}
git status
git add .
git commit -m "Initial Jekyll build."
git push -u origin master
{% endhighlight %}

Open a browser and go to `boykoc.github.io` and BAM, new site.

Then to build and start locally run `bundle exec jekyll serve --host 0.0.0.0`. Again, I'm setting the host to work with virtual box. Those familiur with Rails will notice the similarities.

The css is a bit messed up because the domain is wrong in the _config.yml. You can remove the example url and it'll work nicely.

At this point I created a base-build branch for fun and reference.

[jekyll-docs]: http://jekyllrb.com/docs/home
[github-docs]: https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/
