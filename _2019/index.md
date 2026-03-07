---
layout: page
title: "Föreläsningar 2019"
description: >
  Föreläsningsanteckningar och videor för Den saknade terminen, MIT IAP 2019.
permalink: /2019/
phony: true
---

{% assign lecture_date = site['2019'] | group_by: 'date' | sort: 'name' %}
{% for date in lecture_date %}
  {% if date.name == "" %}{% continue %}{% endif %}
  <h2>{{ date.name | date: '%A, %-m/%-d' }}</h2>
  {% assign lectures = date.items | sort: 'order' %}
  <ul>
  {% for lecture in lectures %}
    {% if lecture.phony != true %}
      <li>
        <a href="{{ lecture.url | relative_url }}">{{ lecture.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
  </ul>
{% endfor %}

# Bortom MIT

Vi har också delat kursen bortom MIT i hopp om att fler ska ha nytta av materialet. Du hittar inlägg och diskussioner på

- [Hacker News](https://news.ycombinator.com/item?id=19078281)
- [Lobsters](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on)
- [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/an42uu/mit_hacker_tools_a_lecture_series_on_programmer/)
- [r/programming](https://www.reddit.com/r/programming/comments/an3xki/mit_hacker_tools_a_lecture_series_on_programmer/)
- [Twitter](https://twitter.com/Jonhoo/status/1091896192332693504)
- [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv)

# Tack

Den här kursen gavs inom [SIPB IAP 2019](https://sipb.mit.edu/iap/2019/) och medarrangerades av [SIPB](https://sipb.mit.edu/) och [MIT EECS](https://www.eecs.mit.edu/).
