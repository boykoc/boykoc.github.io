---
layout: post
title:  "Extracting a class from a method"
date:   2017-10-06 10:47:00 -0400
categories: ruby rubyonrails
---

Sometimes it's nice to see how people are doing things in real codebases. So here is another snippet from a real application I'm working on.

## The Task

I had a class that generated a PDF report with the PRAWN gem that looked at data over a 5 year period. I was asked to create another report that was similar in design but only looked at a single year of data.

## How I Went About This

I first reviewed the original class to see what would have to change to create the new report. I was trying to see if I could make the changes within the class or at least how to reuse the majority of the code as the reports are so similar.

Quickly I knew a new class would be needed for the new report (separate responsibilities and all). I also found that the original PDF class would need to be refactored, it was 773 lines and did way too much without letting me easily reuse anything. 

I figured I could start breaking out the original PDF class into smaller classes that would be easier to maintain, test and reuse.

Below is one of the changes I made. This one method from the original PDF class seemed to do one thing, group some data into a specific structure for the report. It was ideal for extracting into it's own class. It also had many areas within it that were ideal for extracting smaller methods. And, when done, this new class could be called from the new report.

## Original Method

This is one method from a 773 line class (including comments).  **Note: comments have been trimmed to keep the post shorter and I have changed various naming/terms.**

{% highlight bash %}

  ##
  # Get all todos and group them by year for PDF creation.
  #
  # * *Args*    :
  #   - +project+ -> {Project} project object.
  # * *Returns* :
  #   - Hash of todos for given project grouped by year. Years are the keys for array of todo objects.
  # * *Raises* :
  #   - ++ ->
  def get_todos_grouped_by_year(project)
    todos = project.todos.select('todos.*, meetings.meeting_date, groups.acronym, group_sub_types.acronym as group_sub_type_acronym').joins("left join group_meetings on todos.id = group_meetings.todo_id left join meetings on meetings.id = group_meetings.meeting_id left join group_sub_types on group_sub_types.id = group_meetings.group_sub_type_id left join groups on groups.id = meetings.committee_id").joins(:todo_type).where("(group_types.name = 'Content' and (groups.acronym = 'TIL') and (group_sub_types.acronym = 'POL' or group_sub_types.acronym = 'INFO')) or (group_types.name = 'Strategy') or (group_types.name = 'Content' and todos.actual_end_date > '2017-06-30' and meetings.meeting_date IS NULL)")
    additional_todos = project.todos.select('todos.*, MAX(meetings.meeting_date) as meeting_date, groups.acronym, group_sub_types.acronym as group_sub_type_acronym').joins("left join group_meetings on todos.id = group_meetings.todo_id left join meetings on meetings.id = group_meetings.meeting_id left join group_sub_types on group_sub_types.id = group_meetings.group_sub_type_id left join groups on groups.id = meetings.committee_id").joins(:todo_type).where("(group_types.name = 'Content' and todos.additional = true)").group('todos.id')
    todos += additional_todos

    group_of_todos_by_year = todos.group_by do |element|
      if element[:meeting_date].try(:year)
        if (element[:acronym] == 'TIL') || element[:additional]
          # Group by fiscal date.
          year = (element[:meeting_date].month < 4 ? element[:meeting_date].year-1 : element[:meeting_date].year)
          element.date_to_use = element[:meeting_date]
          year
        else
          # Mark todos for deletion by setting key as delete.
          'delete'
        end
      else
        year = (element[:actual_end_date].month < 4 ? element[:actual_end_date].year-1 : element[:actual_end_date].year)
        element.date_to_use = element[:actual_end_date]
        year
      end
    end

    # Get rid of todos marked for deletion.
    group_of_todos_by_year.reject!{|k| k == "delete"}

    # Ensure each todo is show limited times.
    group_of_todos_by_year.each do |key, value|
      group_of_todos_by_year[key] = value.group_by(&:id)
                                         .map { |_k, v| v.delete_if { |x| (v.any? { |s| s[:acronym] == 'TIL' } && x[:acronym] != 'TIL') } }
                                         .delete_if(&:blank?)
                                         .flatten(1)
    end
    group_of_todos_by_year
  end

{% endhighlight %}

## Method as a Class

The new class does everything the big weird method (above) was doing.

Basic steps:

* I first extracted the method to a class with an `initialize(project)` and everything else became a single method called `get_todos_grouped_by_year(project)`
* I then extracted new methods from the large single method
* Areas with multiple levels of nesting I tried to break out into new methods
* Areas that were hard to understand what was going on I broke out
* I made sure the user could instantiate the class and only invoke `call` to get the results (e.g. `todos = ToDosGroupedByYear.new(@project).call`)
* I Updated names of methods to try and make them easier to understand and reduce commenting

{% highlight bash %}

##
# Get todos for the PDF Report and group them by year.
class ToDosGroupedByYear
  def initialize(project)
    @project = project
  end

  ##
  # Get all todos and group them by year for PDF creation.
  #
  # * *Args*    :
  #   - ++ -> {}
  # * *Returns* :
  #   - Hash of todos for given project grouped by year. Years are the keys for array of todo objects.
  # * *Raises* :
  #   - ++ ->
  def call
    final_hash = {}
    group_year.each do |key, value|
      final_hash[key] = value(value)
    end
    final_hash
  end

  ## 
  # Return refined array of todos for year (each todo is shown limited times).
  def value(value)
    value.group_by(&:id)
         .map { |_k, v| v.delete_if { |x| (v.any? { |s| s[:acronym] == 'TIL' } && x[:acronym] != 'TIL') } }
         .delete_if(&:blank?)
         .flatten(1)
  end

  ##
  # Return ALL default todos and the extras not included in the defaults
  # already.
  def combined_todos
    # Queries are now broken out into their own class.
    todos_query = ToDosQuery.new(@project.todos)
    defaults = todos_query.defaults
    todos_query.defaults + (todos_query.additionally_flagged - defaults)
  end

  ##
  # Return fiscal year.
  def year(date)
    date.month < 4 ? date.year - 1 : date.year
  end

  ##
  # Get rid of todos marked for deletion via their hash key.
  def remove_todos(grouped_todos)
    grouped_todos.reject { |k| k == 'delete' }
  end

  ##
  # Returns string to group todo by (year or 'delete').
  def mark_printable_meetings(element)
    if (element[:acronym] == 'TIL') || element[:additional]
      # Group by fiscal date.
      element.date_to_use = element[:meeting_date]
      year(element[:meeting_date])
    else
      # Mark items for deletion by setting key as delete.
      'delete'
    end
  end

  ##
  # Group todos by meeting date or actual end date if no meeting date.
  def group_year
    grouped = combined_todos.group_by do |element|
      if element[:meeting_date].try(:year)
        mark_printable_meetings(element)
      else
        element.date_to_use = element[:actual_end_date]
        year(element[:actual_end_date])
      end
    end
    remove_todos(grouped)
  end
end

{% endhighlight %}

## Why This is Better

I'll keep improving this as I go but here are the main benefits:

* The original PDF class does 1 less thing and closer to focusing on only generating the PDF
* The new class `ToDosGroupedByYear` does exactly what it's called
* Smaller classes all-around
* The new class is testable
* Smaller methods in the new class which are easier to read and maintain
* Enables/encrouages appropriate reuse of the class
* The smaller chunks make it easier to see where additional refactoring is needed or **wonky** stuff is happening
