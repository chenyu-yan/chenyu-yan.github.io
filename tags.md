---
layout: page
title: Tags
permalink: /tags/
---

{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
* **[{{ tag[0] | replace: "_", " " }}](#{{ tag[0] | slugify }})**
{% endfor %}

{% for tag in sorted_tags %}
<span id="{{ tag[0] | slugify }}"></span>
## {{ tag[0] | replace: "_", " " }}
{% assign sorted_post = tag[1] | sort: "date" %}
{% for post in sorted_post reversed %}
* [{{ post.title }}]({{ site.baseurl }}{{ post.url }}) - {{ post.date | date_to_string }}&nbsp;&nbsp;&nbsp;{{ post.categories | join: " " | replace: "_", " " | prepend: "`" | append: "`" }}
{% endfor %}
{% endfor %}
