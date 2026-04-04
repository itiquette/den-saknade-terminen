---
layout: lecture
title: "Metaprogrammering"
description: >
  Lär dig om byggsystem, beroendehantering, testning och kontinuerlig integration.
thumbnail: /static/assets/thumbnails/2020/lec8.png
details: byggsystem, beroendehantering, testning, CI
date: 2020-01-27
ready: true
video:
  aspect: 56.25
  id: _Ms1Z4xfqv4
---

Vad menar vi med "metaprogrammering"?
Det var helt enkelt den bästa samlingsterm vi kunde komma på för en uppsättning saker som handlar mer om _process_ än om att skriva kod eller jobba snabbare.
I föreläsningen tittar vi på system för att bygga och testa kod, och för att hantera beroenden.
Det här kan verka ha begränsad betydelse i din vardag som student, men så fort du interagerar med en större kodbas via praktik eller arbetsliv kommer du att se detta överallt.
Vi bör också nämna att "metaprogrammering" kan betyda "[program som opererar på program](https://en.wikipedia.org/wiki/Metaprogramming)", vilket inte riktigt är den definition vi använder i föreläsningen.

# Byggsystem

Om du skriver en artikel i LaTeX, vilka kommandon behöver du köra för att producera artikeln?
Och vilka kommandon behövs för att köra prestandamätningar, rita diagram och sedan lägga in diagrammen i artikeln?
Eller för att kompilera kod från en kurs och sedan köra testerna?

För de flesta projekt, oavsett om de innehåller kod eller inte, finns en "byggprocess".
Det är en sekvens av operationer du behöver göra för att gå från indata till utdata.
Ofta har processen många steg och många grenar.
Kör detta för att generera det här diagrammet, kör det där för att generera de där resultaten, och något annat för att få fram slutartikeln.
Precis som med mycket annat vi sett i kursen är du inte den första som stöter på det här irritationsmomentet, och som tur är finns många verktyg som hjälper.

De kallas vanligtvis "byggsystem", och det finns _många_.
Vilket du använder beror på uppgiften, vilket språk du föredrar och projektets storlek.
I grunden är de ändå ganska lika.
Du definierar ett antal _beroenden_, ett antal _mål_ och _regler_ för att gå från det ena till det andra.
Du säger till byggsystemet att du vill ha ett visst mål, och dess jobb är att hitta alla transitiva beroenden till målet och sedan tillämpa reglerna för att producera mellanmål tills slutmålet är klart.
Idealiskt gör byggsystemet detta utan att i onödan köra regler för mål vars beroenden inte ändrats och där resultatet redan finns från en tidigare byggning.

`make` är ett av de vanligaste byggsystemen, och det finns oftast installerat på i princip alla UNIX-baserade datorer.
Det har sina skavanker, men fungerar mycket bra för små till medelstora projekt.
När du kör `make` läser det en fil som heter `Makefile` i aktuell katalog.
Alla mål, deras beroenden och reglerna definieras där.
Vi tittar på ett exempel:

```make
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

Varje direktiv i filen är en regel för hur vänstersidan produceras med hjälp av högersidan.
Eller uttryckt på ett annat sätt: sakerna på högersidan är beroenden, och vänstersidan är målet.
Det indenterade blocket är en sekvens av program som producerar målet från dessa beroenden.
I `make` definierar första direktivet också standardmålet.
Om du kör `make` utan argument är det detta mål som byggs.
Alternativt kan du köra till exempel `make plot-data.png`, så bygger det det målet i stället.

`%` i en regel är ett "mönster" och matchar samma sträng på vänster och höger sida.
Om målet `plot-foo.png` till exempel efterfrågas letar `make` efter beroendena `foo.dat` och `plot.py`.
Nu kan vi se vad som händer om vi kör `make` i en tom källkatalog.

```console
$ make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```

`make` berättar hjälpsamt att för att bygga `paper.pdf` behövs `paper.tex`, och det finns ingen regel som säger hur den filen ska skapas.
Vi testar att skapa den.

```console
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop.
```

Intressant, det _finns_ en regel för `plot-data.png`, men det är en mönsterregel.
Eftersom källfilerna inte finns (`data.dat`) säger `make` helt enkelt att den inte kan skapa filen.
Vi testar att skapa alla filer:

```console
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

Vad händer nu om vi kör `make`?

```console
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... massor av utdata ...
```

Och där skapades en PDF åt oss.
Vad händer om vi kör `make` igen?

```console
$ make
make: 'paper.pdf' is up to date.
```

Den gjorde ingenting.
Varför?
Jo, för att den inte behövde.
Den kontrollerade att alla tidigare byggda mål fortfarande var uppdaterade i förhållande till sina listade beroenden.
Vi kan testa detta genom att ändra `paper.tex` och köra `make` igen:

```console
$ vim paper.tex
$ make
pdflatex paper.tex
...
```

Notera att `make` _inte_ körde `plot.py` igen eftersom det inte behövdes.
Inga beroenden till `plot-data.png` hade ändrats.

# Beroendehantering

På en mer övergripande nivå har dina programvaruprojekt sannolikt beroenden som i sig är egna projekt.
Du kan bero på installerade program (som `python`), systempaket (som `openssl`) eller bibliotek i programspråket (som `matplotlib`).
I dag finns de flesta beroenden i ett _kodförråd_ som samlar många beroenden på ett ställe och ger en smidig mekanism för installation.
Exempel är Ubuntus paketkodförråd för systempaket (som du når via `apt`), RubyGems för Ruby-bibliotek, PyPI för Python-bibliotek och Arch User Repository för användarbidragna Arch-paket.

Eftersom de exakta mekanismerna skiljer sig mycket mellan olika kodförråd och verktyg går vi inte djupt in i något specifikt i föreläsningen.
Det vi _ska_ gå igenom är viss gemensam terminologi.
Det första är _versionshantering_.
De flesta projekt som andra projekt beror på släpper ett _versionsnummer_ vid varje utgåva.
Ofta ser det ut som 8.1.3 eller 64.1.20192004. Det är ofta, men inte alltid, numeriskt.
Versionsnummer fyller flera syften, och ett av de viktigaste är att säkerställa att programvara fortsätter fungera.
Tänk dig till exempel att jag släpper en ny version av mitt bibliotek där jag bytt namn på en funktion.
Om någon försöker bygga programvara som beror på biblioteket efter den uppdateringen kan bygget misslyckas eftersom koden anropar en funktion som inte längre finns.
Versionshantering försöker lösa detta genom att låta ett projekt säga att det beror på en viss version, eller ett visst versionsintervall, av ett annat projekt.
På så sätt kan beroende programvara fortsätta bygga mot en äldre biblioteksversion även om biblioteket förändras.

Det är inte heller perfekt.
Vad händer om jag släpper en säkerhetsuppdatering som _inte_ ändrar det publika gränssnittet i biblioteket (dess "API"), och som alla projekt på den gamla versionen borde börja använda direkt?
Det är här de olika siffergrupperna i versionsnumret kommer in.
Den exakta betydelsen varierar mellan projekt, men en relativt vanlig standard är [semantisk versionshantering](https://semver.org/).
Med semantisk versionshantering har varje versionsnummer formen major.minor.patch.
Reglerna är:

  - Om en ny utgåva inte ändrar API:t, öka patchversionen.
 - Om du _lägger till_ i API:t på ett bakåtkompatibelt sätt, öka minorversionen.
 - Om du ändrar API:t på ett icke bakåtkompatibelt sätt, öka majorversionen.

Det ger redan stora fördelar.
Om mitt projekt beror på ditt projekt _bör_ det nu vara säkert att använda senaste utgåva med samma majorversion som jag byggde mot när jag utvecklade, så länge minorversionen är minst lika hög som då.
Med andra ord: om jag beror på version `1.3.7` av ditt bibliotek _bör_ det vara okej att bygga med `1.3.8`, `1.6.1` eller till och med `1.3.0`.
Version `2.2.4` är sannolikt inte okej eftersom majorversionen höjts.
Vi ser ett exempel på semantisk versionshantering i Pythons versionsnummer.
Många av er känner till att Python 2-kod och Python 3-kod inte fungerar särskilt bra tillsammans, vilket är varför det var en _major_-höjning.
På samma sätt kan kod skriven för Python 3.5 fungera fint i Python 3.7, men kanske inte i 3.4.

När du arbetar med beroendehanteringssystem kan du också stöta på _låsfiler_ (lock files).
En låsfil är helt enkelt en fil som listar exakt vilka versioner du _just nu_ beror på för varje beroende.
Vanligtvis måste du uttryckligen köra ett uppdateringskommando för att uppgradera beroenden till nyare versioner.
Det finns många skäl till det, till exempel att undvika onödiga omkompileringar, få reproducerbara byggen eller undvika automatisk uppgradering till senaste version (som kan vara trasig).
En extrem variant av denna typ av beroendelåsning är _beroendeinbäddning_ (vendoring), där du kopierar in all kod från dina beroenden i ditt eget projekt.
Det ger total kontroll över ändringar och låter dig göra egna modifieringar, men betyder också att du aktivt måste dra in uppdateringar från de ansvariga för ursprungsprojektet över tid.

# System för kontinuerlig integration

När du arbetar med större och större projekt märker du att det ofta finns extra uppgifter som behöver göras varje gång du ändrar något.
Du kanske behöver publicera en ny dokumentationsversion, ladda upp en kompilerad version någonstans, släppa kod till PyPI, köra testsviten och mycket annat.
Kanske vill du att varje ändringsförfrågan (PR) på GitHub ska stilkontrolleras och att vissa prestandamätningar ska köras.
När sådana behov uppstår är det dags att titta på kontinuerlig integration.

Kontinuerlig integration, eller CI, är ett paraplybegrepp för "saker som körs när din kod ändras", och det finns många företag som erbjuder olika typer av CI, ofta gratis för öppen källkod-projekt.
Några stora aktörer är Travis CI, Azure Pipelines och GitHub Actions.
Alla fungerar ungefär likadant.
Du lägger till en fil i kodförrådet som beskriver vad som ska hända när olika saker händer i kodförrådet.
Det vanligaste är en regel i stil med "när någon skickar kod, kör testsviten".
När händelsen utlöses startar CI-leverantören en eller flera virtuella maskiner, kör kommandona i ditt "recept" och sparar sedan vanligtvis resultatet någonstans.
Du kan till exempel sätta upp notiser när testsviten börjar misslyckas, eller ett litet statusmärke i kodförrådet så länge testerna går igenom.

Som exempel på CI är kursens webbplats uppsatt med GitHub Pages.
Pages är en CI-åtgärd som kör Jekyll varje gång kod skickas till `master` och publicerar den byggda sajten på en viss GitHub-domän.
Det gör det väldigt enkelt för oss att uppdatera webbplatsen.
Vi gör ändringar lokalt, checkar in till Git och skickar.
CI sköter resten.

## En kort utvikning om testning

De flesta större programvaruprojekt har en "testsvit".
Du kanske redan känner till grundidén med testning, men vi tänkte snabbt nämna några testangreppssätt och termer du kan stöta på:

 - Testsvit: samlingsnamn för alla tester.
 - Enhetstest: ett "mikrotest" som testar en specifik funktion isolerat.
 - Integrationstest: ett "makrotest" som kör en större del av systemet för att kontrollera att olika funktioner eller komponenter fungerar _tillsammans_.
 - Regressionstest: ett test som implementerar ett mönster som _tidigare_ orsakade ett programfel, för att säkerställa att programfelet inte återkommer.
 - Simulering: att ersätta en funktion, modul eller typ med en låtsasimplementation för att undvika att testa orelaterad funktionalitet.
   Till exempel kan du "simulera nätverket" eller "simulera disken".

# Övningar

  1. De flesta makefiler har ett mål som heter `clean`.
     Det är inte tänkt att producera en fil som heter `clean`, utan att städa bort filer som kan byggas om av make.
     Se det som ett sätt att "ångra" alla byggsteg.
     Implementera ett `clean`-mål för `paper.pdf`-`Makefile` ovan.
     Du behöver göra målet [phony](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html).
     Du kan ha nytta av subkommandot [`git ls-files`](https://git-scm.com/docs/git-ls-files).
     Fler vanliga make-mål listas [här](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets).
  2. Titta på de olika sätten att ange versionskrav för beroenden i [Rusts byggsystem](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html).
     De flesta paketkodförråd stödjer liknande syntax.
     För varje variant (caret, tilde, wildcard, comparison och multiple), försök hitta ett användningsfall där just den typen av krav är rimlig.
  3. Git kan fungera som ett enkelt CI-system i sig.
     I `.git/hooks` i valfritt git-kodförråd hittar du filer (just nu inaktiva) som körs som skript när en viss händelse inträffar.
     Skriv en [`pre-commit`](https://git-scm.com/docs/githooks#_pre_commit)-krok som kör `make paper.pdf` och vägrar incheckningen om `make` misslyckas.
     Det ska förhindra incheckningar med en obar byggversion av artikeln.
  4. Sätt upp en enkel sida som autopubliceras med [GitHub Pages](https://pages.github.com/).
     Lägg till en [GitHub Action](https://github.com/features/actions) i kodförrådet som kör `shellcheck` på alla skalfiler i kodförrådet (här är [ett sätt att göra det](https://github.com/marketplace/actions/shellcheck)).
     Kontrollera att det fungerar.
  5. [Bygg din egen](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions) GitHub Action för att köra [`proselint`](https://github.com/amperser/proselint) eller [`write-good`](https://github.com/btford/write-good) på alla `.md`-filer i kodförrådet.
     Aktivera den i kodförrådet och verifiera att den fungerar genom att skapa en ändringsförfrågan (PR) med en stavmiss.
