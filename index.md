---
layout: page
title: Ralf's coding blog
tagline: 
---
{% include JB/setup %}

## Recent posts

<ul class="posts">
{% for post in site.posts limit: 5 %}
  <div class="post_info">
    <li>
            <a href="{{ post.url }}">{{ post.title }}</a>
            <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
    <p> {{ post.excerpt }} </p>
    </div>
  {% endfor %}
</ul>


