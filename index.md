---
title: A Mostly Technical Blog
layout: page
---
{% include JB/setup %}

## About Me

<div class="clearfix">

<a href="/assets/images/shashank-ramaprasad.jpg">
  <img
    style="padding-right: 8px; float: left;"
    width="241" height="242"
    src="/assets/images/shashank-ramaprasad.jpg"
    title="Picture of Shashank Ramaprasad [click/touch to zoom]"
    alt="Picture of Shashank Ramaprasad [click/touch to zoom]" />
</a>

My name is Shashank Ramaprasad. This website has my collected writings on
mostly technical topics (software engineering, computer science and math).
I am currently a Software Engineer at
<a href="https://www.google.com/">Google</a>,
where I work on Payments. I've previously worked at Facebook and Adobe.
Check out <a href="/resume">my resume</a> for more detail.
The opinions expressed
on this website are emphatically and solely my own, and do not in any way
reflect those of any employers of mine.

</div> <!--  class="clearfix"> -->

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date: "%Y-%m-%d" }}</span> &raquo;
    <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
