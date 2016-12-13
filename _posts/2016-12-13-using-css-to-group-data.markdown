---
layout: post
title:  "Using CSS to Group Data"
date:   2016-12-13 11:43:00 -0400
categories: css ruby rubyonrails
---
## Overview
I had a relatively simple task. I wanted to display a calendar with a sidebar that showed upcoming items (what these items are is a bit more detailed but not important here).

I wanted the sidebar to show 10 items to start with and let the user load another 10 by clicking a button.

One key peice was that I wanted the items to be ordered by month with a header every time a new month started.

The last part was a time-suck but in the end turned out well.

## What I Spent Time Trying

I knew I would use the following:
* Rails (this is a feature for an existing project)
* AJAX to load in the 10 new items at a time
* Altered pagination to get 10 items at a time (already using the pagination gem so the overhead was already there so I figured I might as well use it again for this)

And then how to order items and show a month heading when loading 10 results at a time that could span any number of months (up to 10 I guess). Below was my desired output.

**Sidebar**

Month 1:
* Item 1
* Item 2
* Item 3

Month 2: 
* Item 4
* Item 5
* Item 6
* Item 7
* Item 8

Month 3:
* Item 9
* Item 10

*Load More Button*

If Item 11 is in month 3 it should be added to the month 3 list without a new heading. If it's a new month, it should get a heading and be added to the list.

I could easily figure out how to return a group of items with a month header or how to return 10 items at a time in order either with Ruby and/or sql. **But I couldn't figure out how to return 10 items and only add a new heading at the start of a new month**.

I could use sql to group or order the results. I could get all the results and use Ruby's `group_by` method. But once I thought about returning the results to the front-end I ran into troubles.

## What I Ended Up Doing

I realized I was thinking about the front-end result and trying to get my back-end to structure my data based on a UI design I had in mind. Not clean seperation of responsibilities.

I thought about if I was building a service or writing an Angular app consuming a service I would likely get a list of items, specify the order, and possible the number of records. After that it would be up to the front-end to decide how to handle it and display it.

From that I decided to add a `data-month` attribute to my items when I was building them out on the front-end.

{% highlight bash %}
# _item_partial.html.erb
<li class="item" data-month="<%= items.item_date.beginning_of_month.strftime("%Y-%B") %>">
  <%= link_to items.name, edit_items_path(items) %>
</li>
{% endhighlight %}

With the `data-month` attribute I could use css to insert the month before items each time a new month appears. I wish the css was a bit cleaner/less repetative but for now it works well for now.

{% highlight bash %}
#sidebar {
    #items {
        // Insert month heading before each item.
        > li[data-month*="January"]::before,
        > li[data-month*="February"]::before,
        > li[data-month*="March"]::before,
        > li[data-month*="April"]::before,
        > li[data-month*="May"]::before,
        > li[data-month*="June"]::before,
        > li[data-month*="July"]::before,
        > li[data-month*="August"]::before,
        > li[data-month*="September"]::before,
        > li[data-month*="October"]::before,
        > li[data-month*="November"]::before,
        > li[data-month*="December"]::before {
            content: attr(data-month) ":";
            display: block;
            font-weight: 500;
            margin-top: 16px;
            margin-bottom: 8px;
        }
        
        // Remove heading for all but the first occurance.
        > li[data-month*="January"] ~ li[data-month*="January"]::before,
        > li[data-month*="February"] ~ li[data-month*="February"]::before,
        > li[data-month*="March"] ~ li[data-month*="March"]::before,
        > li[data-month*="April"] ~ li[data-month*="April"]::before,
        > li[data-month*="May"] ~ li[data-month*="May"]::before,
        > li[data-month*="June"] ~ li[data-month*="June"]::before,
        > li[data-month*="July"] ~ li[data-month*="July"]::before,
        > li[data-month*="August"] ~ li[data-month*="August"]::before,
        > li[data-month*="September"] ~ li[data-month*="September"]::before,
        > li[data-month*="October"] ~ li[data-month*="October"]::before,
        > li[data-month*="November"] ~ li[data-month*="November"]::before,
        > li[data-month*="December"] ~ li[data-month*="December"]::before {
            display: none;
        }
    }
{% endhighlight %}

And my back-end can easily return ordered results 10 at a time. No grouping or sorting to worry about.

## Wrap it up

I sometimes spend a lot of time trying to figure out the best way to do something (how can I do it all at once, the most effecient way, etc). Here I realized I need to take a step back and keep it simple.

The basics:
* Keep responsibilities seperate
* Don't be affraid to get it working and then refactor
* Let each thing do what it's good at
