---
layout: default
title: Home
---

<div class="home">

  <h1 class="page-heading">{{ site.title }}</h1>
  <p>{{ site.description }}</p>

  <h2>Latest posts</h2>

  <ul class="post-list">
  {% assign posts = site.pages | where_exp: "p", "p.url contains '/posts/'" | sort: "date" | reverse %}
  {% for post in posts %}
    <li>
      <span class="post-meta">{% if post.date %}{{ post.date | date: "%b %-d, %Y" }}{% endif %}</span>
      <h3>
        <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </h3>
      {% if post.tags %}
      <p class="post-tags">
        {% for tag in post.tags %}<code>#{{ tag }}</code> {% endfor %}
      </p>
      {% endif %}
    </li>
  {% endfor %}
  </ul>

</div>
