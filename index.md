---
layout: page
title: Den saknade terminen i din datavetenskapsutbildning
description: >
  Bemästra kraftfulla verktyg som gör dig till en mer produktiv datavetare och programmerare.
# subtitle: IAP 2026
subtitle: "2026"
nositetitle: true
---

På många datavetenskapskurser lär du dig avancerade ämnen, från operativsystem till maskininlärning.
Men ett avgörande område hamnar ofta i skymundan och lämnas åt studenterna själva: att bli skicklig med sina verktyg.
Vi lär dig att behärska kommandoraden, använda en kraftfull textredigerare, utnyttja avancerade funktioner i versionshanteringssystem och mer därtill.

Studenter lägger hundratals timmar på de här verktygen under utbildningen (och tusentals under arbetslivet), så det är rimligt att göra arbetet så smidigt som möjligt.
När du behärskar de här verktygen lägger du mindre tid på att få dem att göra som du vill, och kan samtidigt lösa problem som tidigare verkade nästan omöjliga.

I dag förändras också många delar av programvaruutveckling genom införandet av AI-stödda verktyg och arbetsflöden.
När de används på rätt sätt, och med förståelse för deras begränsningar, kan de ge tydliga fördelar för alla som jobbar med programvara.
Därför är det värt att bygga upp praktisk kunskap om dem.
Eftersom AI är en tvärgående teknik har vi ingen fristående AI-föreläsning; i stället har vi vävt in relevanta AI-verktyg och tekniker direkt i varje föreläsning.

Läs om [motivationen för kursen]({{ '/about/' | relative_url }}).

{% comment %}
# Registration

Sign up for the IAP 2026 class by filling out this [registration form](https://forms.gle/j2wMzi7qeiZmzEWy9).
{% endcomment %}

# Kursplan

{% comment %}
**Lecture**: [35-225](https://whereis.mit.edu/?go=35), 1:30--2:30pm (_exception_: 3--4pm on Friday 1/16)<br>
**Discussion**: [OSSU Discord](https://ossu.dev/#community) (use `#missing-semester-forum` like you would use Piazza, and `#missing-semester` to chat with the class/instructors)
{% endcomment %}

<ul>
{% assign lectures = site['2026'] | sort: 'date' %}
{% for lecture in lectures %}
    {% if lecture.phony != true %}
        <li>
        <strong>{{ lecture.date | date: '%-m/%-d/%y' }}</strong>:
        {% if lecture.ready %}
            <a href="{{ lecture.url | relative_url }}">{{ lecture.title }}</a>
        {% else %}
            {{ lecture.title }} {% if lecture.noclass %}[ingen föreläsning]{% endif %}
        {% endif %}
        </li>
    {% endif %}
{% endfor %}
</ul>

## Specialämnen från tidigare år

Ämnena vi tar upp varierar från år till år. För studenter som är intresserade av alla ämnen vi har behandlat genom åren lyfter vi här fram ämnen från tidigare år som inte ingick 2026.

{% comment %} pop to remove default "posts" collection {% endcomment %}
{% assign sorted_collections = site.collections | sort: 'label' | pop | reverse %}
<ul>
{% for collection in sorted_collections %}
    {% assign grouped_lectures = site[collection.label] | group_by: 'date' | sort: 'name' %}
    {% for group in grouped_lectures %}
        {% assign sorted_lectures = group.items | sort: 'order' %}
        {% for lecture in sorted_lectures %}
            {% if lecture.special == true %}
                <li>
                    <strong>{{ lecture.date | date: '%-m/%-d/%y' }}</strong>:
                    <a href="{{ lecture.url | relative_url }}">{{ lecture.title }}</a>
                </li>
            {% endif %}
        {% endfor %}
    {% endfor %}
{% endfor %}
</ul>

{% comment %}
Lecture videos will be made available to MIT students immediately after lecture (via Panopto).
The system has a limitation that only those with an MIT Kerberos can access the raw lecture videos.
We are working on editing lecture videos and uploading them to YouTube.
A couple have been uploaded already; we expect the rest to be uploaded by mid-February.

If you can't wait until January 2026, you can also take a look at the lectures
from the [previous offering of the course]({{ '/2020/' | relative_url }}), which covers many of the
same topics.
{% endcomment %}

# Allmän information

**Lärare**: Den här kursen undervisas tillsammans av [Anish](https://anish.io/), [Jon](https://thesquareplanet.com/) och [Jose](https://josejg.com/).<br>
**Frågor**: Mejla oss på [missing-semester@mit.edu](mailto:missing-semester@mit.edu).<br>
**Diskussion**: [OSSU Discord](https://ossu.dev/#community) (använd `#missing-semester-forum` ungefär som Piazza, och `#missing-semester` för att prata med klassen och lärarna).

# Bortom MIT

Vi har också delat den här kursen bortom MIT i hopp om att fler ska kunna dra nytta av materialet.
Du hittar inlägg och diskussioner på

 - Hacker News ([2026](https://news.ycombinator.com/item?id=47124171), [2020](https://news.ycombinator.com/item?id=22226380), [2019](https://news.ycombinator.com/item?id=19078281))
 - Lobsters ([2026](https://lobste.rs/s/q4ykw7/missing_semester_your_cs_education_2026), [2020](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit), [2019](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on))
 - r/learnprogramming ([2026](https://www.reddit.com/r/learnprogramming/comments/1r93yk6/the_missing_semester_of_your_cs_education_2026/), [2020](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/), [2019](https://www.reddit.com/r/learnprogramming/comments/an42uu/mit_hacker_tools_a_lecture_series_on_programmer/))
 - r/programming ([2020](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/), [2019](https://www.reddit.com/r/programming/comments/an3xki/mit_hacker_tools_a_lecture_series_on_programmer/))
 - X ([2026](https://x.com/anishathalye/status/2024521145777848588), [2020](https://twitter.com/jonhoo/status/1224383452591509507), [2019](https://x.com/jonhoo/status/1090323977766137858))
 - Bluesky ([2026](https://bsky.app/profile/jonhoo.eu/post/3mfa2bhyuj22i))
 - Mastodon ([2026](https://fosstodon.org/@jonhoo/116098318361854057))
 - LinkedIn ([2026](https://www.linkedin.com/posts/anishathalye_i-returned-to-mit-during-iap-january-term-activity-7430285026933522433-Ehr9))
 - YouTube ([2026](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L), [2020](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J), [2019](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv))

# Översättningar

{% comment %} keep these in alphabetical order {% endcomment %}

- [Arabic](https://missing-semester-ar.github.io/)
- [Bengali](https://missing-semester-bn.github.io/)
- [Chinese (Simplified)](https://missing-semester-cn.github.io/)
- [Chinese (Traditional, Taiwan)](https://missing-semester-tw.github.io/)
- [German](https://missing-semester-de.github.io/)
- [Italian](https://missing-semester-it.github.io/)
- [Japanese](https://missing-semester-jp.github.io/)
- [Kannada](https://missing-semester-kn.github.io/)
- [Korean](https://missing-semester-kr.github.io/)
- [Persian](https://missing-semester-fa.github.io/)
- [Portuguese](https://missing-semester-pt.github.io/)
- [Russian](https://missing-semester-rus.github.io/)
- [Serbian](https://netboxify.com/missing-semester/)
- [Spanish](https://missing-semester-esp.github.io/)
- [Swedish](https://itiquette.github.io/den-saknade-terminen/)
- [Thai](https://missing-semester-th.github.io/)
- [Turkish](https://missing-semester-tr.github.io/)
- [Vietnamese](https://missing-semester-vn.github.io/)

Obs: detta är externa länkar till gemenskapsöversättningar.
Vi har inte granskat dem.

Har du skapat en översättning av kursanteckningarna från den här kursen?
Skicka en [ändringsförfrågan (PR)](https://github.com/missing-semester/missing-semester/pulls) så kan vi lägga till den i listan.

## Tack

{% comment %}
2026 acks; previous years' acks are on their respective pages
{% endcomment %}

Vi tackar Elaine Mello och [MIT Open Learning](https://openlearning.mit.edu/) för att de gjorde det möjligt för oss att spela in föreläsningsvideor.
Vi tackar Luis Turino / [SIPB](https://sipb.mit.edu/) för att de stöttar den här kursen som en del av [SIPB IAP 2026](https://sipb.mit.edu/iap/).

---

<div class="small center">
<p><a href="https://github.com/missing-semester/missing-semester">Källkod</a>.</p>
<p>Licensierat under CC BY-NC-SA.</p>
<p>Se <a href="{{ '/license/' | relative_url }}">här</a> för riktlinjer för bidrag och översättning.</p>
</div>
