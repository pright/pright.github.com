---
layout: page
title: 
---
{% include JB/setup %}

{% for post in site.posts %}
<h1><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
<hr>
<h5>{{ post.date | date_to_long_string }}</h5>
{{ post.content }}
{% endfor %}
