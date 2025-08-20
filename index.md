
---
layout: home
title: Home
---

## Blog Posts

<ul>
{%- for post in site.posts -%}
  <li>
    <a href="{{ post.url | absolute_url }}">{{ post.title }}</a>{% if post.date %} - {{ post.date | date: "%B %d, %Y" }}{% endif %}
  </li>
{%- endfor -%}
</ul>


