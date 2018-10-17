---
layout: page
title: Categories
permalink: /categories/
---

{% assign sorted_categories = site.categories | sort %}
{% for category in sorted_categories %}
* **[{{ category[0] | replace: "_", " " }}](#{{ category[0] | slugify }})**
{% endfor %}

{% for category in sorted_categories %}
<span id="{{ category[0] | slugify }}"></span>
## {{ category[0] | replace: "_", " " }}
{% assign sorted_post = category[1] | sort: "date" %}
{% for post in sorted_post reversed %}
{% assign tagged_str = "" %}
{% for tag in post.tags %}
    {% assign tagged_str = tagged_str | append: "`" | append: tag | append: "` " %}
{% endfor %}
* [{{ post.title }}]({{ site.baseurl }}{{ post.url }}) - {{ post.date | date_to_string }}&nbsp;&nbsp;&nbsp;{{ tagged_str | replace: "_", " " }}
{% endfor %}
{% endfor %}
