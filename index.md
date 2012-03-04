---
layout: page
title: Latest posts
---
{% include JB/setup %}

{% for post in site.posts %}
<div><h2>{{ post.date | date_to_string }} &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2></div>
<div>{{ post.content }}</div>
{% endfor %}
