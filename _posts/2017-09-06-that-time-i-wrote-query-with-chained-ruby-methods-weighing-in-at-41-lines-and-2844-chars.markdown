---
layout: post
title:  "That time I wrote query with chained ruby methods weighing in at 41 lines and 2844 chars"
date:   2017-09-06 04:01:00 -0400
categories: ruby rubyonrails
---
## That time I wrote query with chained ruby methods weighing in at 41 lines and 2844 chars

First things first, **I REFACTORED THIS NASTY MESS!** And this is focused on how I got into the mess and how I addressed it.

In my opinion Ruby is great and Rails is amazing. They make me a happy programmer and pretty productive. However, they let you get into trouble if you're not careful. Actually, this isn't really the fault of the language at all. It does what I tell it to. This is what happens if you don't slow down and look for ways to keep your code clean. Luckily I've had some time to go back and fix this. I've already started trying to hold myself to a few rules i.e. more classes are better then fewer, keep methods short - when I hit 10 lines really think about how to break it out, and a few others.

## How did I get to writing a huge query like 41 lines?

Well actually it's part of a 61 line `index` action in a controller. Which by the way, uses some `Model` methods to help trim it down (writing that made me laugh at myself because of how huge it still is).

Anyway, this controller and action started much smaller. It was a standard index action that evolved quickly. It started with a standard query to get some data and we launched it. The users asked for some filters, so I quickly added some filtering by getting the params, making sure they exist, and setting a default if they don't and we pushed. The user then asked for the data to be grouped by date then the ability to filter and group some 'one-off categories' which I added with Ruby then used JS on the frontend to show and hide things, and they wanted to fast. So I started making the changes right there in the controller action and ruby let me chain things like crazy, so I did. I love ruby...It's so easy to get stuff done...even if it's dirty.

## What I ended up with

After all the updates and iterations I ended up with this big controller action. 

And now my main concern is this giant query thing that's doing way too much. It looks like a great candidate to be pulled out into it's own class.

{% highlight bash %}
# Giant query thing. 
# NOTE: Some table names have been changed.
assigned = Hash[Task.select('tasks.*, date_tasks.meeting_id, date_tasks.milestone_id, dates.id as m_id, dates.meeting_date as meeting_date, committees.acronym as committee_acronym')
                               .joins('RIGHT JOIN date_tasks ON date_tasks.milestone_id = tasks.id RIGHT JOIN dates ON dates.id = date_tasks.meeting_id JOIN committees ON committees.id = dates.committee_id')
                               .where('dates.meeting_date >= ? AND dates.meeting_date <= ?', @beginning_of_month, end_of_range)
                               .includes([{ groups: :status },
                                          :ministries,
                                          { date_tasks: :sub_type },
                                          :dates])
                               .where('committees.id in (?)', committee_ids)
                               .order('committee_acronym asc, tasks.name asc')
                               .group_by { |task| task.meeting_date.month }
                               .map { |month, entries| [month, entries.group_by { |x| x.committee_acronym || 'acronym' } ] }
                               .map do |key, values|
                                  [key, values.map do |k, value|
                                    [k, value.group_by do |v|
                                      val = v.try(:groups).try(:first).try(:category).try(:id) || 0
                                      case val
                                      when 1, 6, 9, 10, 12, 15, 16
                                        'one-off category one'
                                      when 2
                                        'one-off category two'
                                      when 3, 17, 18, 19, 20, 21
                                        'one-off category three'
                                      when 7
                                        'one-off category four'
                                      when 8
                                        'one-off category five'
                                      when 23
                                        'one-off category six'
                                      when 5
                                        'one-off category seven'
                                      when 14
                                        'one-off category eight'
                                      when 13
                                        'one-off category nine'
                                      else
                                        'none'
                                      end
                                    end]
                                  end]
                               end
                      ]
{% endhighlight %}

## Trimming it down

The main fix is to make it more managable and bite-sized. I created a new class with the following.

{% highlight bash %}
class GroupedTasks
  def initialize()
  end

  def assigned
  end
end
{% endhighlight %}

Then I copied the whole `assigned = ...` query into `#assigned` and looked for the variables used in the query. These would need to be pulled out and put in the `#initialize` method and passed in from the controller whan instantiating the class.

{% highlight bash %}
class GroupedTasks
  attr_reader :beginning_of_month, :end_of_range, :committee_ids

  def initialize(beginning_of_month, end_of_range, committee_ids)
    @beginning_of_month = beginning_of_month
    @end_of_range = end_of_range
    @committee_ids = committee_ids
  end

  def assigned
    assigned = Hash[Task.select('tasks.*, date_tasks.meeting_id, date_tasks.milestone_id, dates.id as m_id, dates.meeting_date as meeting_date, committees.acronym as committee_acronym')
                         .joins('RIGHT JOIN date_tasks ON date_tasks.milestone_id = tasks.id RIGHT JOIN dates ON dates.id = date_tasks.meeting_id JOIN committees ON committees.id = dates.committee_id')
                         .where('dates.meeting_date >= ? AND dates.meeting_date <= ?', @beginning_of_month, @end_of_range)
                         .includes([{ groups: :status },
                                    :ministries,
                                    { date_tasks: :sub_type },
                                    :dates])
                         .where('committees.id in (?)', @committee_ids)
                         .order('committee_acronym asc, tasks.name asc')
                         .group_by { |task| task.meeting_date.month }
                         .map { |month, entries| [month, entries.group_by { |x| x.committee_acronym || 'acronym' } ] }
                         .map do |key, values|
                            [key, values.map do |k, value|
                              [k, value.group_by do |v|
                                val = v.try(:groups).try(:first).try(:category).try(:id) || 0
                                case val
                                when 1, 6, 9, 10, 12, 15, 16
                                  'one-off category one'
                                when 2
                                  'one-off category two'
                                when 3, 17, 18, 19, 20, 21
                                  'one-off category three'
                                when 7
                                  'one-off category four'
                                when 8
                                  'one-off category five'
                                when 23
                                  'one-off category six'
                                when 5
                                  'one-off category seven'
                                when 14
                                  'one-off category eight'
                                when 13
                                  'one-off category nine'
                                else
                                  'none'
                                end
                              end]
                            end]
                         end
                ]  
  end
end
{% endhighlight %}

At this point I can cleanup my controller a bit, changing it from the above original to:

`assigned = GroupedOutlookMilestones.new(@beginning_of_month, end_of_range, committee_ids).assigned`

**And BAMM!** From 41 lines down to 1. But this so far has just moved the problem. But I can now unit test this one method which is nice. The real fix will be breaking this method down into smaller pieces.

First I broke out the query into it's own method enabling unit testing it specifically if I leave it as a public method.

Next, I moved the case statement out into it's own method. This helped improve readability of the code. Now when I look at the `#assigned` method it's much easier to see what's happening.

After this change all my methods are more readable, managable and testable. They all could be a bit smaller but I think they are ok for now. `#assigned` basically groups the data into a hash structure I wanted. `#query` gets the data I need. And `#category` is basically a getting that returns the correct category.

{% highlight bash %}
class GroupedTasks
  attr_reader :beginning_of_month, :end_of_range, :committee_ids

  def initialize(beginning_of_month, end_of_range, committee_ids)
    @beginning_of_month = beginning_of_month
    @end_of_range = end_of_range
    @committee_ids = committee_ids
  end

  # Returns the grouped hash.
  def assigned
    assigned = Hash[query.group_by { |task| task.meeting_date.month }
                         .map { |month, entries| [month, entries.group_by { |x| x.committee_acronym || 'acronym' } ] }
                         .map do |key, values|
                            [key, values.map do |k, value|
                              [k, value.group_by do |v|
                                val = v.try(:groups).try(:first).try(:category).try(:id) || 0
                                oneOffCategory(val)
                              end]
                            end]
                         end
                ]  
  end
  
  # Returns the needed data.
  def query
    Task.select('tasks.*, date_tasks.meeting_id, date_tasks.milestone_id, dates.id as m_id, dates.meeting_date as meeting_date, committees.acronym as committee_acronym')
        .joins('RIGHT JOIN date_tasks ON date_tasks.milestone_id = tasks.id RIGHT JOIN dates ON dates.id = date_tasks.meeting_id JOIN committees ON committees.id = dates.committee_id')
        .where('dates.meeting_date >= ? AND dates.meeting_date <= ?', @beginning_of_month, @end_of_range)
        .includes([{ groups: :status },
                   :ministries,
                   { date_tasks: :sub_type },
                   :dates])
        .where('committees.id in (?)', @committee_ids)
        .order('committee_acronym asc, tasks.name asc')  
  end
  
  # Returns the correct one-off category.
  def oneOffCategory(value)
    case value
    when 1, 6, 9, 10, 12, 15, 16
      'one-off category one'
    when 2
      'one-off category two'
    when 3, 17, 18, 19, 20, 21
      'one-off category three'
    when 7
      'one-off category four'
    when 8
      'one-off category five'
    when 23
      'one-off category six'
    when 5
      'one-off category seven'
    when 14
      'one-off category eight'
    when 13
      'one-off category nine'
    else
      'none'
    end  
  end
  
end
{% endhighlight %}

## Keep in mind

We don't write perfect code. I've always struggled with finding "the best way to implement" things to the point that it can paralize me and I get nothing done except try and research the absolute best way. Lately I've been trying to hard to just write an implementation that works, use comments like `# OPTIMIZE: ...` to make notes along the way, once it's working with a passing test, go back and make changes and improvements. But, go back before it gets too big and out of hand.
