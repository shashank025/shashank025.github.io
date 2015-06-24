---
title: Welcome!
layout: page
---
{% include JB/setup %}

My collected writings on technical topics (software engineering, computer science and math). Occasionally, I may write about culture and politics.
    
## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
        <ul> <li> {{ post.content | strip_html | truncatewords: 25 }} </li> </ul>
    </li>
  {% endfor %}
</ul>
