---
layout: page
title: "Föreläsningar 2026"
description: >
  Föreläsningsanteckningar och videor för Den saknade terminen, MIT IAP 2026.
permalink: /2026/
phony: true
---

<ul class="double-spaced">
  {% assign lectures = site['2026'] | sort: 'date' %}
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

Videoinspelningar av föreläsningarna finns [på YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L).

# Bortom MIT

Vi har också delat kursen bortom MIT i hopp om att fler ska ha nytta av materialet. Du hittar inlägg och diskussioner på

- [Hacker News](https://news.ycombinator.com/item?id=47124171)
- [Lobsters](https://lobste.rs/s/q4ykw7/missing_semester_your_cs_education_2026)
- [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/1r93yk6/the_missing_semester_of_your_cs_education_2026/)
- [X](https://x.com/anishathalye/status/2024521145777848588)
- [Bluesky](https://bsky.app/profile/jonhoo.eu/post/3mfa2bhyuj22i)
- [Mastodon](https://fosstodon.org/@jonhoo/116098318361854057)
- [LinkedIn](https://www.linkedin.com/posts/anishathalye_i-returned-to-mit-during-iap-january-term-activity-7430285026933522433-Ehr9)
- [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L)

# Tack

Vi tackar Elaine Mello och [MIT Open Learning](https://openlearning.mit.edu/) för att de gjorde det möjligt för oss att spela in föreläsningsvideor. Vi tackar Luis Turino / [SIPB](https://sipb.mit.edu/) för stöd till kursen inom ramen för [SIPB IAP 2026](https://sipb.mit.edu/iap/).
