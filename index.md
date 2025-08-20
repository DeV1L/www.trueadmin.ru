
---
layout: home
title: Home
sidebar:
  nav: "posts"
---

<div class="page-with-sidebar">
  <div class="content-area">
    <div class="archive">
      <h1 id="page-title" class="page__title">{{ page.title }}</h1>
      {% for post in site.posts %}
        {% include archive-single.html %}
      {% endfor %}
    </div>
  </div>

  <div class="custom-sidebar-wrapper">
    <div class="custom-sidebar">
      {% include sidebar.html %}
    </div>
  </div>
</div>


