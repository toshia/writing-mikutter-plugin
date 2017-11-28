---
layout: page
title: Reference
permalink: /reference/
---
## 共通
{% for post in site.categories.common %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endfor %}

## 基礎知識
{% for post in site.categories.basis %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endfor %}

## BOT作成
{% for post in site.categories.bot %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endfor %}

## 応用編

### Model

{% for post in site.categories.model %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endfor %}

## 付録

{% for post in site.categories.reference %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endfor %}

