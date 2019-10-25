---
layout: post
title:  "CKAN 2.8.3 Database Diagram"
date:   2019-10-21 15:00:00 -0400
categories: CKAN
---

## Overview

I took a schema dump from postgres and used some reverse engineering tools to create the diagram. 

Areas of to take note of:

* comments/notes of 4 tables
* 3 "orphan" tables
* 1 polymorphic table
* default schema dump creates 6 unconnected tables, 3 have missing FK constraints:

{% highlight sql %}

alter table tracking_summary
   add constraint tracking_summary_package_id_fk
      foreign key (package_id) references package;

-- Note: this one is to help demonstrate is relation but read the comment!
alter table task_status
   add constraint task_status_entity_id_fk
      foreign key (entity_id) references resource;

alter table activity
   add constraint activity_user_id_fk
      foreign key (user_id) references user;
      
{% endhighlight %}

**Right click, open image in new tab to zoom in.**

<img alt="ckan 2.8.3 DDL physical diagram" src="/assets/img/ckan-2.8.3-ddl-diagram-2019-10-21.png" />

## Physical Data Model

{% assign tables = site.data.ckan-2-8-2-data-model.Tables %}
{% for table in tables %}

   <h3>{{ table["Metadata"]["Name"] }}</h3>
   <p>{{ table["Metadata"]["Description"] }}</p>

   <table>
     <thead>
       <tr>
         <th>Field</th>
         <th>Description</th>
       </tr>
     </thead>
     <tbody>
        {{ table["Columns"] }}
       {% for column in table["Columns"] %}
         <tr>
           <td>{{ column[0] }}</td>
           <td>{{ column[1] }}</td>
         </tr>
       {% endfor %}
     </tbody>
   </table>

{% endfor %}
