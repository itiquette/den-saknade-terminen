---
layout: lecture
title: "Pakethantering och beroendehantering"
presenter: Anish
date: 2019-01-29
order: 2
video:
  aspect: 56.25
  id: tgvt473T8xA
---

Programvara bygger oftast på (en samling av) annan programvara,
vilket gör beroendehantering nödvändigt.

Program för paket-/beroendehantering är språkspecifika,
men många delar samma grundidéer.

# Paketförråd

Paket lagras i _paketförråd_.
Det finns olika förråd för olika språk (och ibland flera för samma språk),
till exempel [PyPI](https://pypi.org/) för Python, [RubyGems](https://rubygems.org/) för Ruby,
och [crates.io](https://crates.io/) för Rust.
De lagrar normalt programvara (källkod och ibland förkompilerade binärer för specifika plattformar)
för alla versioner av ett paket.

# Semantisk versionshantering

Programvara utvecklas över tid,
och vi behöver ett sätt att referera till versioner.
Enkla sätt är till exempel löpnummer eller commit-hash,
men vi kan kommunicera mer information med versionsnummer.

Det finns många angreppssätt.
Ett populärt är [Semantic Versioning](https://semver.org/):

```
x.y.z
^ ^ ^
| | +- patch
| +--- minor
+----- major
```

Öka **major** när du gör icke bakåtkompatibla API-ändringar.

Öka **minor** när du lägger till funktionalitet bakåtkompatibelt.

Öka **patch** när du gör bakåtkompatibla buggfixar.

Om du till exempel beror på en funktion som introducerades i `v1.2.0` av en viss programvara,
kan du installera `v1.x.y` för alla minor-versioner `x >= 2` och alla patch-versioner `y`.
Du behöver major-version `1` (eftersom `2` kan introducera icke bakåtkompatibla ändringar),
och du behöver en minor-version `>= 2` (eftersom din funktion kom i den minor-versionen).
Du kan använda valfri nyare minor- eller patch-version,
eftersom de inte bör introducera icke bakåtkompatibla ändringar.

# Låsfiler

Utöver att ange versioner kan det vara bra att säkerställa att
_innehållet_ i beroenden inte har ändrats, för att motverka manipulering.
Vissa verktyg använder _låsfiler_ för att ange kryptografiska hashvärden för beroenden (tillsammans med versioner),
som verifieras vid paketinstallation.

# Ange versioner

Verktyg låter dig ofta ange versioner på flera sätt, till exempel:

- exakt version, t.ex. `2.3.12`
- minsta major-version, t.ex. `>= 2`
- specifik major-version och minsta patch-version, t.ex. `>= 2.3, <3.0`

Att ange en exakt version kan vara fördelaktigt för att undvika olika beteenden
beroende på installerade beroenden (detta borde inte hända om alla följer semver,
men misstag sker).
Att ange ett minimikrav har fördelen att buggfixar kan installeras
(t.ex. patch-uppgraderingar).

# Beroendelösning

Pakethanterare använder olika algoritmer för beroendelösning för att uppfylla beroendekrav.
Det blir ofta svårt vid komplexa beroenden
(t.ex. att ett paket beror indirekt via flera toppnivåberoenden som kräver olika versioner).
Olika pakethanterare har olika grad av sofistikation i beroendelösningen,
men det är något du bör känna till:
du kan behöva förstå detta när du felsöker beroenden.

# Virtuella miljöer

Om du utvecklar flera programvaruprojekt kan de bero på olika versioner av samma programvara.
Ibland hanterar byggverktyget detta naturligt (t.ex. genom att bygga en statisk binär).

För andra byggverktyg och språk är en väg att hantera detta med virtuella miljöer
(t.ex. med
[virtualenv](https://docs.python-guide.org/dev/virtualenvs/) för Python).
I stället för att installera beroenden systembrett kan du installera dem per projekt i en virtuell miljö,
och _aktivera_ den miljö du vill använda när du arbetar med ett visst projekt.

# Vendoring (incheckade beroenden)

En annan, mycket annorlunda, metod för beroendehantering är _vendoring_ (att checka in beroenden).
I stället för att använda en beroendehanterare eller ett byggverktyg för att hämta programvara,
kopierar du in hela källkoden för ett beroende i projektets kodförråd.
Fördelen är att du alltid bygger mot samma beroendeversion och inte behöver lita på ett paketförråd,
men det kräver mer arbete att uppgradera beroenden.
