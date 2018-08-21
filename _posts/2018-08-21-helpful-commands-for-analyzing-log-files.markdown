---
layout: post
title:  "Helpful commands for analyzing log files"
date:   2018-08-21 15:00:00 -0400
categories: linux logs
---
There's lots of ways to go through log files and dig out what you need. Some or more complex then others and some will get you what you're looking for faster, better or easier. But I've found there are some basic commands that will get you rolling quickly.

## Basic commands

This ones kind of obvious but sometimes it's easiest to just open the file with an editor and search for what you're looking for.

{% highlight bash %}
# Replace nano with your editor of choice.
# Replace file with your file of choice.
nano log_file.log
# Search for the word/thing your looking for.
ctrl + w
# Enter the word you're looking for.
enter
# To keep looking for the next occurance, repeat.
ctrl + w
enter
{% endhighlight %}

Sometimes you don't want to filter through the whole file but still want something basic and easy to use.

These came in handy after a script I wrote to port data from a CSV to a new system dumped all errors into a log file.

{% highlight bash %}
# Print any line in the log_file.log that matches the string '"maintainer_email": ' and print the line after it as well.
grep -A 1 '"maintainer_email": ' log_file.log
{% endhighlight %}

To get the number of lines with the error. I use this to see how big of a problem I have.

`grep -A 1 '"maintainer_email": ' log_file.log | wc -l`
