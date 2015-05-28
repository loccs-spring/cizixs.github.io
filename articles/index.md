---
layout: page
title: Articles
excerpt: "An archive of article posts sorted by date."
search_omit: true
---

<ul class="post-list">
{% for article in site.categories.article %} 
  <li><article><a href="{{ site.url }}{{ article.url }}">{{ article.title }} <span class="entry-date"><time datetime="{{ article.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span>{% if post.excerpt %} <span class="excerpt">{{ post.excerpt }}</span>{% endif %}</a></article></li>
{% endfor %}
</ul>
