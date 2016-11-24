---
layout: page
title: Archives
---

{% for post in site.posts %}{{ post.date | date_to_string }} <strong>&raquo;</strong> [ {{ post.title }} ]({{ post.url }})  
{% endfor %}
