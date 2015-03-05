---
layout: page
title: Spkinger
tagline: Deeper and deeper
---
{% include JB/setup %}

{% for post in site.posts limit:5 %}
<h2><a class="post_title" href="{{post.url}}">{{post.title}}</a></h2>
<div class="post-content">{{ post.date | date_to_string }}</div>
{% endfor %} 
