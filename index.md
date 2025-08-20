
---
layout: home
title: Home
---

<div class="page-with-sidebar">
  <div class="content-area">

## Blog Posts

<ul>
{%- for post in site.posts -%}
  <li>
    <a href="{{ post.url | absolute_url }}">{{ post.title }}</a>{% if post.date %} - {{ post.date | date: "%B %d, %Y" }}{% endif %}
  </li>
{%- endfor -%}
</ul>

  </div>

  <div class="custom-sidebar-wrapper">
    <div class="custom-sidebar">
      {% include sidebar.html %}
    </div>
  </div>
</div>


