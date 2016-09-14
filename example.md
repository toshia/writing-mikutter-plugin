---
layout: page
title: Example
permalink: /example/
---

mikutterプラグインは何でもできますが、初めてプラグインを作ろうとする人は、何処から手を付けていいかわからないかもしれません。

いくつかの簡単なプラグインの作り方を読んで、どんなことができるのか学んでみましょう。

#### 一覧
{% for post in site.categories.example %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endfor %}

