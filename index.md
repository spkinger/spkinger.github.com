---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts limit:5 %}
<h2><a class="post_title" href="{{post.url}}">{{post.title}}</a></h2>
<div class="post-content">{{post.content}}</div>
{% endfor %}
