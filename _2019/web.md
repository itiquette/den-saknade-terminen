---
layout: lecture
title: "Webben och webbläsare"
presenter: Jose
date: 2019-01-31
order: 1
video:
  aspect: 62.5
  id: XpZO3S8odec
special: true
---

Utöver terminalen är webbläsaren ett verktyg du kommer att tillbringa mycket tid i.
Därför är det värt att lära sig använda den effektivt.

## Kortkommandon

Att klicka runt i webbläsaren är ofta inte snabbast. Att bli bekant med vanliga kortkommandon lönar sig på sikt.

- `Middle Button Click` på en länk öppnar den i en ny flik
- `Ctrl+T` öppnar en ny flik
- `Ctrl+Shift+T` öppnar nyligen stängd flik igen
- `Ctrl+L` markerar innehållet i adressfältet
- `Ctrl+F` söker på en webbsida.
Om du gör detta ofta kan du ha nytta av ett tillägg som stödjer reguljära uttryck i sökning.


## Sökoperatorer

Sökmotorer på webben, som Google eller DuckDuckGo, erbjuder sökoperatorer för mer avancerade sökningar:

- `"bar foo"` tvingar exakt matchning av bar foo
- `foo site:bar.com` söker efter foo inom bar.com
- `foo -bar ` utesluter träffar som innehåller bar
- `foobar filetype:pdf` söker efter filer med den filändelsen
- `(foo|bar)` söker efter träffar med foo ELLER bar

Mer utförliga listor finns för populära motorer som [Google](https://ahrefs.com/blog/google-advanced-search-operators/) och [DuckDuckGo](https://duck.co/help/results/syntax).


## Adressfältet

Adressfältet är också ett kraftfullt verktyg.
De flesta webbläsare kan härleda sökmotorer från webbplatser och lagrar dem.
Genom att redigera nyckelordsargumentet:

- I Google Chrome finns de på [chrome://settings/searchEngines](chrome://settings/searchEngines)
- I Firefox finns de på [about:preferences#search](about:preferences#search)

Till exempel kan du göra så att `y SOME SEARCH TERMS` söker direkt på YouTube.

Om du äger en domän kan du dessutom sätta upp vidarebefordran av subdomäner hos din registrar.
Jag har till exempel pekat `https://ht.josejg.com` till kurswebbplatsen.
Då kan jag bara skriva `ht.` så autokompletterar adressfältet.
En annan fördel med upplägget är att det, till skillnad från bokmärken, fungerar i alla webbläsare.

## Integritetstillägg

Numera kan webbsurfning bli ganska störig på grund av annonser och påträngande spårning.
En bra annonsblockerare blockerar inte bara annonsinnehåll, utan kan också blockera misstänkta och skadliga webbplatser eftersom de ofta finns i vanliga blocklistor.
Ibland förbättras även laddningstider eftersom färre förfrågningar skickas.
Några rekommendationer:

- **uBlock origin** ([Chrome](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm), [Firefox](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)): blockerar annonser och spårare utifrån fördefinierade regler.
Du bör också titta på aktiverade blocklistor i inställningarna, eftersom du kan slå på fler beroende på region eller surfvanor.
Du kan till och med installera filter från [olika källor på webben](https://github.com/gorhill/uBlock/wiki/Filter-lists-from-around-the-web).

- **[Privacy Badger](https://privacybadger.org/)**: upptäcker och blockerar spårare automatiskt.
När du går mellan olika webbplatser kan annonsbolag till exempel spåra vad du besöker och bygga en profil av dig.

- **[HTTPS everywhere](https://www.eff.org/https-everywhere)** är ett utmärkt tillägg som automatiskt omdirigerar till HTTPS-versionen av en webbplats om den finns.

Du hittar fler tillägg av den här typen [här](https://www.privacytools.io/privacy-browser-addons/).

## Stilanpassning

Webbläsare är bara ännu en programvara som kör på _din maskin_, så du har i regel sista ordet om vad de ska visa och hur de ska bete sig.
Ett exempel är anpassade stilar.
Webbläsare avgör hur en webbsidas stil återges med Cascading Style Sheets, oftast förkortat CSS.

Du kan komma åt en webbplats källkod genom att inspektera den och ändra innehåll och stilar tillfälligt.
(Det är också en anledning till att du aldrig bör lita blint på skärmbilder av webbsidor.)

Om du permanent vill att webbläsaren ska skriva över stilinställningar för en webbplats behöver du ett tillägg.
Vår rekommendation är **[Stylus](https://github.com/openstyles/stylus)** ([Firefox](https://addons.mozilla.org/en-US/firefox/addon/styl-us/), [Chrome](https://chrome.google.com/webstore/detail/stylus/clngdbkpkpeebahjckkjfobafhncgmne?hl=en)).


Vi kan till exempel skriva följande stil för kurswebbplatsen.


```css

body {
    background-color: #2d2d2d;
    color: #eee;
    font-family: Fira Code;
    font-size: 16pt;
}

a:link {
    text-decoration: none;
    color: #0a0;
}
```

Stylus kan dessutom hitta stilar skrivna av andra användare och publicerade på [userstyles.org](https://userstyles.org/).
Många vanliga webbplatser har till exempel en eller flera mörka teman.
Använd däremot inte Stylish, eftersom tillägget visat sig läcka användardata.
Läs mer [här](https://arstechnica.com/information-technology/2018/07/stylish-extension-with-2m-downloads-banished-for-tracking-every-site-visit/).


## Anpassa funktionalitet

På samma sätt som du kan ändra stil kan du också ändra beteendet hos en webbplats genom att skriva egen JavaScript och ladda den via ett webbläsartillägg som [Tampermonkey](https://tampermonkey.net/).

Följande skript aktiverar till exempel vim-liknande navigering med tangenterna J och K.

```js
// ==UserScript==
// @name         VIM HT
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  Vim JK for our website
// @author       You
// @match        https://hacker-tools.github.io/*
// @grant        none
// ==/UserScript==


(function() {
    'use strict';

    window.onkeyup = function(e) {
        var key = e.keyCode ? e.keyCode : e.which;

        if (key == 74) { // J is key 74
            window.scrollBy(0,500);;
        }else if (key == 75) { // K is key 75
            window.scrollBy(0,-500);;
        }
    }
})();
```

Det finns också skriptarkiv som [OpenUserJS](https://openuserjs.org/) och [Greasy Fork](https://greasyfork.org/en).
Men var försiktig: att installera användarskript från andra kan vara väldigt farligt, eftersom de i princip kan göra vad som helst, till exempel stjäla dina kortuppgifter.
Installera aldrig ett skript om du inte läst hela själv, förstått vad det gör, och är helt säker på att det inte gör något misstänkt.
Installera aldrig skript med komprimerad eller fördunklad kod som du inte kan läsa.

## Webb-API:er

Det har blivit allt vanligare att webbtjänster erbjuder ett applikationsgränssnitt, alltså ett webb-API, så att du kan interagera med tjänsten via webbförfrågningar.
En mer djupgående introduktion finns [här](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction).
Det finns [många publika API:er](https://github.com/toddmotto/public-apis).
Webb-API:er kan vara användbara av många skäl:

- **Hämtning**.
Webb-API:er kan enkelt ge dig information som kartor, väder eller din publika IP-adress.
Till exempel returnerar `curl ipinfo.io` ett JSON-objekt med detaljer om publik IP, region, plats, osv. Med korrekt tolkning kan sådana verktyg integreras även med kommandoradsverktyg.
Följande bash-funktion pratar med Googles API för autokomplettering och returnerar de tio första träffarna.

```bash
function c() {
    url='https://www.google.com/complete/search?client=hp&hl=en&xhr=t'
    # NB: user-agent must be specified to get back UTF-8 data!
    curl -H 'user-agent: Mozilla/5.0' -sSG --data-urlencode "q=$*" "$url" |
        jq -r ".[1][][0]" |
        sed 's,</\?b>,,g'
}
```

- **Interaktion**.
API-ändpunkter kan också användas för att utlösa handlingar.
Det kräver vanligtvis någon form av autentiseringstoken som du får via tjänsten.
Till exempel skickar följande `curl -X POST -H 'Content-type: application/json' --data '{"text":"Hej, världen!"}' "https://hooks.slack.com/services/$SLACK_TOKEN"` ett `Hej, världen!`-meddelande i en kanal.

- **Kopplingar**.
Eftersom vissa tjänster med webb-API:er är populära finns vanlig API-"ihopkoppling" redan implementerad och tillhandahålls som tjänst. Det gäller tjänster som [If This Then That](https://ifttt.com/) och [Zapier](https://zapier.com/).


## Webbautomatisering

Ibland räcker webb-API:er inte till.
Om du bara behöver läsa innehåll kan du använda en HTML-tolk som `pup` eller ett bibliotek, till exempel BeautifulSoup i Python.
Men om interaktivitet eller JavaScript-körning krävs räcker de lösningarna inte.
Då är WebDriver relevant.


Följande skript sparar till exempel angiven URL i Wayback Machine genom att simulera interaktionen att skriva in webbplatsen.

```python
from selenium.webdriver import Firefox
from selenium.webdriver.common.keys import Keys


def snapshot_wayback(driver, url):

    driver.get("https://web.archive.org/")
    elem = driver.find_element_by_class_name('web-save-url-input')
    elem.clear()
    elem.send_keys(url)
    elem.send_keys(Keys.RETURN)
    driver.close()


driver = Firefox()
url = 'https://hacker-tools.github.io'
snapshot_wayback(driver, url)
```


## Övningar

1. Redigera en nyckelordsbaserad sökmotor som du använder ofta i webbläsaren.
1. Installera de nämnda tilläggen.
Undersök hur uBlock Origin/Privacy Badger kan inaktiveras för en webbplats.
Vilka skillnader ser du?
Testa på en webbplats med mycket annonser, som YouTube.
1. Installera Stylus och skriv en egen stil för kurswebbplatsen med den CSS som ges.
Här är några vanliga programmeringstecken: `= == === >= => ++ /= ~=`.
Vad händer med dem när du byter typsnitt till Fira Code?
Om du vill veta mer, sök på typsnittsligaturer för programmering.
1. Hitta ett webb-API för väder i din stad eller region.
1. Använd en WebDriver-programvara som [Selenium](https://www.selenium.dev/documentation/) för att automatisera en repetitiv manuell uppgift du ofta gör i webbläsaren.
