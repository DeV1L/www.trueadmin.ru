
---
layout: default
title: Home
---

## Blog Posts

<ul>
{%- if site.posts != empty -%}
  {%- assign posts_list = site.posts -%}
{%- else -%}
  {%- assign posts_list = site.pages | where_exp: "p", "p.path contains 'posts/'" | sort: 'date' | reverse -%}
{%- endif -%}
{%- for post in posts_list -%}
  <li>
    <a href="{{ post.url | absolute_url }}">{{ post.title }}</a>{% if post.date %} - {{ post.date | date: "%B %d, %Y" }}{% endif %}
  </li>
{%- endfor -%}
</ul>


