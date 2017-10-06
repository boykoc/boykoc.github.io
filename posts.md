---
layout: page
title: Posts
permalink: /posts/
---

<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@site_username">
<meta name="twitter:creator" content="@creator_username">
{% if page.title %}
  <meta name="twitter:title" content="{{ page.title }}">
{% else %}
  <meta name="twitter:title" content="{{ site.title }}">
{% endif %}
{% if page.url %}
  <meta name="twitter:url" content="{{ site.url }}{{ page.url }}">
{% endif %}
{% if page.description %}
  <meta name="twitter:description" content="{{ page.description }}">
{% else %}
  <meta name="twitter:description" content="{{ site.description }}">
{% endif %}
{% if page.image %}
  <meta name="twitter:image:src" content="{{ site.url }}/path/to/image/{{ page.image }}">
{% else %}
  <meta name="twitter:image:src" content="{{ site.url }}/path/to/image/logo.png">
{% endif %}

<h1 class="page-heading">Posts</h1>

<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

      <h2>
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title | escape }}</a>
      </h2>
    </li>
  {% endfor %}
</ul>

<p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>
