---
layout: page
title: "Föreläsningar 2020"
description: >
  Föreläsningsanteckningar och videor för Den saknade terminen, MIT IAP 2020.
permalink: /2020/
phony: true
---

<ul class="double-spaced">
  {% assign lectures = site['2020'] | sort: 'date' %}
  {% for lecture in lectures %}
    {% if lecture.phony != true %}
      <li>
        <strong>{{ lecture.date | date: '%-m/%-d' }}</strong>:
        {% if lecture.ready %}
          <a href="{{ lecture.url | relative_url }}">{{ lecture.title }}</a>
        {% elsif lecture.noclass %}
          {{ lecture.title }} [ingen föreläsning]
        {% else %}
          {{ lecture.title }} [kommer snart]
        {% endif %}
        {% if lecture.details %}
          <br>
          ({{ lecture.details }})
        {% endif %}
      </li>
    {% endif %}
  {% endfor %}
</ul>

Videoinspelningarna finns [på YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J).

# Bortom MIT

Vi har också delat kursen bortom MIT i hopp om att fler ska ha nytta av materialet.
Du hittar inlägg och diskussioner på:

 - [Hacker News](https://news.ycombinator.com/item?id=22226380)
 - [Lobsters](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit)
 - [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/)
 - [r/programming](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/)
 - [Twitter](https://twitter.com/jonhoo/status/1224383452591509507)
 - [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J)

{% comment %}
Några fler länkar:

- https://news.ycombinator.com/item?id=27154577
- https://news.ycombinator.com/item?id=34934216
- https://www.reddit.com/r/learnprogramming/comments/nca1v3/mit_the_missing_semester_of_your_cs_education/
- https://www.reddit.com/r/compsci/comments/eyywv8/the_missing_semester_of_your_cs_education_from_mit/
- https://www.reddit.com/r/programming/comments/io7nq3/the_missing_semester_of_your_cs_education_mit/
- https://twitter.com/MIT_CSAIL/status/1349766980413263873
- https://twitter.com/MIT_CSAIL/status/1481676163491659780
- https://twitter.com/MIT_CSAIL/status/1581313961093484545
{% endcomment %}

# Tack

Vi tackar Elaine Mello, Jim Cain och [MIT Open Learning](https://openlearning.mit.edu/) för att de gjorde det möjligt för oss att spela in föreläsningsvideor; Anthony Zolnik och [MIT AeroAstro](https://aeroastro.mit.edu/) för A/V-utrustning; och Brandi Adams och [MIT EECS](https://www.eecs.mit.edu/) för stöd till kursen.
