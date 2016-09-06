---
layout: page
title: Reference
permalink: /reference/
---
## 共通
{% for post in site.categories.common %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

## 基礎知識
{% for post in site.categories.basis %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

## BOT作成
{% for post in site.categories.bot %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}



