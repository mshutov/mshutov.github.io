---
---
{% for post in site.posts limit:2 %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}
