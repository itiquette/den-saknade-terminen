---
layout: lecture
title: "Automatisering"
presenter: Jose
date: 2019-01-24
order: 3
video:
  aspect: 56.25
  id: BaLlAaHz-1k
special: true
---

Ibland skriver du ett skript som utför något, men du vill att det ska köras regelbundet, till exempel en säkerhetskopieringsuppgift.
Du kan alltid skriva en *ad hoc*-lösning som kör i bakgrunden och vaknar med jämna mellanrum.
Men de flesta UNIX-system kommer med cron-demonen, som kan köra uppgifter så ofta som varje minut utifrån enkla regler.

På de flesta UNIX-system kör cron-demonen `crond` som standard, men du kan alltid kontrollera med `ps aux | grep crond`.

## Crontab

Cron-konfigurationen kan visas med `crontab -l` och redigeras med `crontab -e`.
Tidsformatet som cron använder består av fem blankstegsseparerade fält, tillsammans med användare och kommando.

- **minute** - vilken minut i timmen kommandot ska köras på, och ligger mellan '0' och '59'
- **hour** - styr vilken timme kommandot ska köras på, och anges i
         24-timmarsformat, värdet måste vara mellan 0 och 23 (0 är midnatt)
- **dom** - Day of Month, alltså vilken dag i månaden kommandot ska köras, t.ex. för att köra den 19:e varje månad är dom 19.
- **month** - månaden som kommandot ska köras i, kan anges numeriskt (0-12) eller som månadens namn (t.ex. May).
- **dow** - Day of Week, vilken veckodag kommandot ska köras på, kan anges numeriskt (0-7) eller som veckodagens namn (t.ex. sun).
- **user** - användaren som kör kommandot.
- **command** - kommandot du vill köra.
  Det här fältet kan innehålla flera ord eller blanksteg.

Observera att en asterisk `*` betyder alla värden, och en asterisk följd av snedstreck och tal betyder varje n:te värde.
Alltså betyder `*/5` var femte.
Några exempel:

```shell
*/5   *    *   *   *       # Var femte minut
  0   *    *   *   *       # Varje hel timme
  0   9    *   *   *       # Varje dag kl. 09:00
  0   9-17 *   *   *       # Varje timme mellan 09:00 och 17:00
  0   0    *   *   5       # Varje fredag kl. 00:00
  0   0    1   */2 *       # Varannan månad, den första, kl. 00:00
```
Du hittar flera exempel på vanliga crontab-scheman på [crontab.guru](https://crontab.guru/examples.html).

## Skalmiljö och loggning

En vanlig fallgrop med cron är att den inte läser in samma miljöskript som vanliga skal, till exempel `.bashrc`, `.zshrc`, osv, och den loggar inte heller utdata någonstans som standard.
I kombination med att minsta tidsenhet är en minut kan felsökning av cron-skript i början vara ganska plågsam.

För att hantera miljön bör du använda absoluta sökvägar i alla skript och justera miljövariabler som `PATH` så att skriptet kan köras korrekt.
En bra rekommendation för enklare loggning är att skriva din crontab i stil med detta.


```shell
* * * * *   user  /path/to/cronscripts/every_minute.sh >> /tmp/cron_every_minute.log 2>&1
```

Skriv skriptet i en separat fil.
Kom ihåg att `>>` lägger till i filen och att `2>&1` omdirigerar `stderr` till `stdout` (du kan vilja hålla dem separata).

## Anacron

En begränsning med cron är att om datorn är avstängd eller i viloläge när cron-skriptet skulle köras, så körs det inte.
För täta uppgifter kan det vara okej, men om en uppgift körs mer sällan kan du vilja säkerställa att den faktiskt körs.
[anacron](https://linux.die.net/man/8/anacron) fungerar likt `cron`, men frekvensen anges i dagar.
Till skillnad från cron antar det inte att maskinen körs kontinuerligt.
Det gör att det kan användas på maskiner som inte är igång dygnet runt, för återkommande jobb som dagliga, veckovisa och månatliga jobb.


## Övningar

1. Skapa ett skript som varje minut tittar i din nedladdningskatalog efter bildfiler (du kan använda [MIME-typer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types) eller ett reguljärt uttryck som matchar vanliga filändelser) och flyttar dem till din bildkatalog.

1. Skriv ett cron-skript som varje vecka kontrollerar om du har föråldrade paket i systemet och antingen frågar om uppdatering eller uppdaterar automatiskt.



{% comment %}

- [fswatch](https://github.com/emcrisostomo/fswatch)
- GUI automation (pyautogui) [Automating the boring stuff Chapter 18](https://automatetheboringstuff.com/chapter18/)
- Ansible/puppet/chef

- https://xkcd.com/1205/
- https://xkcd.com/1319/

{% endcomment %}
