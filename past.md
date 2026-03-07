---
layout: page
title: Tidigare omgångar
description: >
  Hitta alla tidigare omgångar av Den saknade terminen.
---

{% comment %} pop to remove default "posts" collection {% endcomment %}
{% assign sorted_collections = site.collections | sort: 'label' | pop | reverse %}
<ul>
{% for collection in sorted_collections %}
    <li><a href="{{ '/' | append: collection.label | append: '/' | relative_url }}">{{ collection.label }}</a></li>
{% endfor %}
</ul>

Varje års föreläsningar är helt fristående.
Vi rekommenderar att du börjar med den senaste versionen av materialet.
Ämnena varierar från år till år, så vi fortsätter att tillgängliggöra anteckningar och videor från tidigare versioner av kursen.
