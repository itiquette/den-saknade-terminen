---
layout: lecture
title: "Kodkvalitet"
description: >
  Lär dig om formatering, lintning, testning, kontinuerlig integration och mer.
thumbnail: /static/assets/thumbnails/2026/lec9.png
date: 2026-01-23
ready: true
video:
  aspect: 56.25
  id: XBiLUNx84CQ
---

Det finns en uppsjö av verktyg och tekniker som hjälper utvecklare att skriva kod av hög kvalitet.
I föreläsningen går vi igenom:

- [Formatering](#formatering)
- [Lintning](#lintning)
- [Testning](#testning)
- [Pre-commit-krokar](#pre-commit-krokar)
- [Kontinuerlig integration(CI)](#kontinuerlig-integration)
- [Kommandokörare](#kommandokörare)

Som bonusämne går vi också igenom [reguljära uttryck](#reguljära-uttryck), ett tvärgående ämne som används inom kodkvalitet (t.ex. för att köra en delmängd tester som matchar ett mönster) och i andra områden som IDE:er (t.ex. för sök och ersätt).

Flera av dessa verktyg är språkspecifika (t.ex. lintnings-/formateringsverktyget [Ruff](https://docs.astral.sh/ruff/) för Python).
I vissa fall stöder verktyg flera språk (t.ex. kodformateraren [Prettier](https://prettier.io/)).
Koncepten är däremot nästan allmängiltiga --- du kan hitta kodformaterare, linters, testbibliotek och så vidare för vilket programmeringsspråk som helst.

# Formatering

Automatiska kodformaterare snyggar upp ytsyntaxen automatiskt.
På så sätt kan du fokusera på djupare och mer utmanande problem, medan formateringsverktyget hanterar vardagsdetaljer som konsekvent användning av `'` kontra `"` i strängar, mellanslag runt binära operatorer (`x + y` i stället för `x+y`), sorterade `import`-satser och att undvika för långa rader.
En stor fördel med kodformaterare är att de standardiserar kodstilen för alla utvecklare som arbetar i kodbasen.

Vissa verktyg, som Prettier, är [i hög grad konfigurerbara](https://prettier.io/docs/configuration), och du bör versionshantera konfigurationsfilen i [versionshantering]({{ '/2026/version-control/' | relative_url }}) för projektet.
Andra verktyg, som [Black](https://github.com/psf/black) och [gofmt](https://pkg.go.dev/cmd/gofmt), har begränsad eller ingen konfigurerbarhet för att minska [trivialitetsdebatter](https://en.wikipedia.org/wiki/Law_of_triviality) (s.k. _bikeshedding_).

Du kan sätta upp [integrering i IDE:n]({{ '/2026/development-environment/#code-intelligence-and-language-servers' | relative_url }}) med din kodformaterare, så att koden formateras automatiskt medan du skriver eller när du sparar en fil.
Du kan också lägga till en [EditorConfig](https://editorconfig.org/)-fil i projektet, som kommunicerar projektnivåinställningar till IDE:n, till exempel indenteringsstorlek per filtyp.

# Lintning

Linters kör statisk analys (analyserar din kod utan att köra den) för att hitta antimönster och potentiella problem i koden.
Dessa verktyg går djupare än autoformaterare och tittar bortom ytsyntax.
Hur djup analysen är varierar mellan verktyg.

Linters kommer med listor av _regler_, med förinställningar som kan konfigureras på projektnivå.
Vissa lintregler ger falska positiva resultat, så du kan stänga av dem per fil eller per rad.

Bra linters har inbyggd hjälp eller dokumentation som förklarar varje lintregel --- vad regeln letar efter, varför det är dåligt och vad som är ett bättre alternativ för kodmönstret.
Se till exempel dokumentationen för regeln [SIM102](https://docs.astral.sh/ruff/rules/collapsible-if/) i [Ruff](https://docs.astral.sh/ruff/), som fångar onödigt nästlade `if`-satser i Python-kod.

Vissa linters kan inte bara flagga problem utan också automatiskt fixa vissa problem åt dig.

Utöver språkspecifika linters kan ett annat verktyg som kan vara användbart vara [semgrep](https://github.com/semgrep/semgrep), ett "semantiskt grep"-verktyg som arbetar på AST-nivå (i stället för teckennivå, som grep) och stöder många språk.
Du kan använda semgrep för att enkelt skriva egna lintregler för dina projekt.
Om du till exempel vill förhindra farlig användning av `subprocess.Popen(..., shell=True)` i Python kan du hitta det kodmönstret med:

```bash
semgrep -l python -e "subprocess.Popen(..., shell=True, ...)"
```

# Testning

Programvarutestning är en standardteknik för att öka din tillit till att koden är korrekt.
Du skriver kod, och sedan skriver du kod som kör den kod du skrev och kastar ett fel om koden inte fungerar som förväntat.

Du kan skriva tester för kodblock på olika granularitetsnivåer: _enhetstester_ för enskilda funktioner, _integrationstester_ för samspel mellan moduler eller tjänster och _funktionella tester_ för heltäckande scenarier.
Du kan arbeta med _testdriven utveckling_, där du skriver tester innan du skriver implementationen.
När du hittar programfel i koden kan du skriva _regressionstester_ så att du fångar om funktionaliteten går sönder i framtiden.
Du kan skriva _egenskapsbaserade tester_, introducerade i [QuickCheck](https://hackage.haskell.org/package/QuickCheck) i Haskell och implementerade i många bibliotek, som [Hypothesis](https://hypothesis.readthedocs.io/) för Python.
Vilken teststrategi som passar beror på projektet, och du kommer sannolikt att använda en kombination.

Om programmet har externa beroenden som en databas eller ett webb-API kan det vara hjälpsamt att _simulera_ dessa beroenden i testerna i stället för att låta koden interagera med tredjepartsberoenden vid testkörning.

## Kodtäckning

Kodtäckning är ett mått som du kan använda för att mäta hur bra dina tester är.
Kodtäckning tittar på vilka rader i koden som exekveras när testerna körs, så att du kan säkerställa att du täcker alla kodvägar.
Verktyg för kodtäckning kan visa täckning rad för rad för att hjälpa dig skriva tester.
Tjänster som [Codecov](https://app.codecov.io) erbjuder webbgränssnitt för att följa och visa kodtäckning över projektets historik.

Som alla mätetal är kodtäckning inte perfekt, så överoptimera inte för täckning utan fokusera på att skriva högkvalitativa tester.

<span id="pre-commit-krokar"></span>
# Pre-commit-krokar

Git [pre-commit-krokar](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks), som blir enklare med ramverket [pre-commit](https://pre-commit.com/), kör automatiskt användarspecificerad kod före varje Git-incheckning.
Projekt använder ofta pre-commit-krokar för att köra formaterare och linters, och ibland tester, automatiskt före varje incheckning för att säkerställa att kod i incheckningen följer projektets kodstil och är fri från vissa typer av problem.

# Kontinuerlig integration

Tjänster för kontinuerlig integration (CI), som [GitHub Actions](https://github.com/features/actions), kan köra skript åt dig varje gång du skickar kod (eller vid varje ändringsförfrågan (PR), eller enligt schema).
Utvecklare använder ofta CI-tjänster för att köra kodkvalitetsverktyg, inklusive formaterare, linters och tester.
För kompilerade språk kan du säkerställa att koden kompilerar, och för statiskt typade språk kan du säkerställa att den typkontrollerar.
Att köra CI varje gång ny kod skickas kan fånga fel som förs in i huvudversionen av koden.
Att köra vid ändringsförfrågningar kan fånga problem i bidrag från andra.
Att köra enligt schema kan fånga problem med externa beroenden (t.ex. när en utvecklare av misstag släpper en brytande ändring som [semver-kompatibel]({{ '/2026/shipping-code/#utgavor-och-versionering' | relative_url }})).

Eftersom CI-skript körs separat från utvecklarnas datorer kan du enkelt köra långkörande jobb där.
Det kan till exempel användas för att köra en test-_matris_ över olika operativsystem och versionskombinationer av programmeringsspråk för att säkerställa att programvaran fungerar korrekt på alla.

Generellt ska skriptet som körs i CI inte direkt ändra koden.
Det kör verktyg i kontrollläge i stället för rättningsläge, så till exempel formateraren höjer ett fel när koden inte följer formatet.

Kodförråd innehåller ofta [statusmärken](https://docs.github.com/en/actions/how-tos/monitor-workflows/add-a-status-badge) i README, som visar CI-status och annan information som kodtäckning.
Nedan är Missing Semesters nuvarande byggstatus.

[![Build Status](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml) [![Links Status](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml)

> Vår [länkkontroll](https://github.com/missing-semester/missing-semester/blob/master/.github/workflows/links.yml), som använder GitHub Action [proof-html](https://github.com/anishathalye/proof-html), misslyckas ofta, vanligtvis på grund av problem på tredjepartswebbplatser.
> Trots det har den hjälpt oss att hitta och fixa många brutna länkar (ibland på grund av stavfel, oftast för att webbplatser flyttar innehåll utan att lägga till omdirigeringar eller för att webbplatser försvinner).

Ett bra sätt att lära sig detaljerna i CI-tjänster, formaterare, linters och testbibliotek är att lära genom exempel.
Hitta högkvalitativa öppen källkod-projekt på GitHub --- ju mer de liknar ditt projekt i språk, domän, storlek, omfattning och så vidare, desto bättre --- och studera deras `pyproject.toml`, `.github/workflows/`, `DEVELOPMENT.md` och andra relevanta filer.

## Kontinuerlig driftsättning (CD)

Kontinuerlig driftsättning använder CI-infrastruktur för att faktiskt _driftsätta_ ändringar.
Till exempel använder Missing Semesters kodförråd kontinuerlig driftsättning till GitHub Pages, så att webbplatsen byggs och driftsätts automatiskt när vi skickar uppdaterade föreläsningsanteckningar med `git push`.
Du kan bygga andra typer av [artefakter]({{ '/2026/shipping-code/' | relative_url }}) i CI, till exempel binärer för applikationer eller Docker-avbilder för tjänster.

# Kommandokörare

Kommandokörare som [just](https://github.com/casey/just) förenklar uppgiften att köra kommandon i projektets kontext.
När du bygger upp infrastruktur för kodkvalitet i projektet vill du inte att utvecklarna ska behöva memorera kommandon som `uv run ruff check --fix`.
Med ett sådant verktyg kan detta bli `just lint`, och du kan ha motsvarande kommandon som `just format`, `just typecheck` och så vidare för alla olika verktyg som en utvecklare kan vilja köra i projektet.

Vissa språkspecifika projekt- eller pakethanterare har inbyggt stöd för sådan funktionalitet, vilket betyder att du inte behöver använda ett språkagnostiskt verktyg som `just`.
Till exempel stöder `scripts`-sektionen i en `package.json` för [npm](https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager) (Node.js) och sektionerna `tool.hatch.envs.*.scripts` i en `pyproject.toml` för [Hatch](https://hatch.pypa.io/) (Python) detta.

<span id="regular-expressions"></span>
# Reguljära uttryck

_Reguljära uttryck_, ofta förkortat "regex", är ett språk för att representera mängder av strängar.
Regex-mönster används ofta för mönstermatchning i olika sammanhang, till exempel kommandoradsverktyg och IDE:er.
Till exempel stöder [ag](https://github.com/ggreer/the_silver_searcher) regex-mönster för sökning i hela kodbasen (t.ex. `ag "import .* as .*"` hittar alla omdöpta importer i Python), och [go test](https://pkg.go.dev/cmd/go#hdr-Test_packages) stöder alternativet `-run [regexp]` för att välja en delmängd av tester.
Dessutom har programmeringsspråk inbyggt stöd eller tredjepartsbibliotek för reguljära uttryck, så du kan använda regex för funktioner som mönstermatchning, validering och tolkning.

För att bygga intuition följer här några exempel på regex-mönster.
I föreläsningen använder vi [Python-syntax för regex](https://docs.python.org/3/library/re.html).
Det finns många regex-varianter med små skillnader mellan dem, särskilt i mer avancerad funktionalitet.
Du kan använda en webbaserad testare som [regex101](https://regex101.com/) för att utveckla och felsöka reguljära uttryck.

- `abc` --- matchar den bokstavliga strängen "abc".
- `missing|semester` --- matchar strängen "missing" eller strängen "semester".
- `\d{4}-\d{2}-\d{2}` --- matchar datum i formatet YYYY-MM-DD, till exempel "2026-01-14".
  Utöver att säkerställa att strängen består av fyra siffror, ett bindestreck, två siffror, ett bindestreck och två siffror validerar det inte själva datumet, så "2026-01-99" matchar också detta regex-mönster.
- `.+@.+` --- matchar e-postadresser, alltså strängar som innehåller text, sedan ett "@" och sedan mer text.
  Detta gör bara en väldigt grundläggande validering och matchar strängar som "nonsense@@@email".
  Ett regex som matchar e-postadresser utan falska positiva eller negativa [finns](https://pdw.ex-parrot.com/Mail-RFC822-Address.html), men är opraktiskt.

## Regex-syntax

Du hittar en omfattande guide till regex-syntax i [den här dokumentationen](https://docs.python.org/3/library/re.html#regular-expression-syntax) (eller i någon av många andra resurser på nätet).
Här är några grundläggande byggstenar:

- `abc` matchar den bokstavliga strängen när tecknen inte har särskild betydelse (i det här exemplet "abc")
- `.` matchar ett valfritt enskilt tecken
- `[abc]` matchar ett enskilt tecken som finns inom hakparenteserna (i det här exemplet "a", "b" eller "c")
- `[^abc]` matchar ett enskilt tecken utom dem som finns inom hakparenteserna (t.ex. "d")
- `[a-f]` matchar ett enskilt tecken inom intervallet i hakparenteserna (t.ex. "c", men inte "q")
- `a|b` matchar något av mönstren (t.ex. "a" eller "b")
- `\d` matchar valfri siffra (t.ex. "3")
- `\w` matchar valfritt ordtecken (t.ex. "x")
- `\b` matchar en ord-_gräns_ (t.ex. i strängen "missing semester", precis före "m", precis efter "g", precis före "s" och precis efter "r")
- `(...)` matchar en grupp i ett mönster
- `...?` matchar noll eller en av ett mönster, till exempel `words?` för att matcha "word" eller "words"
- `...*` matchar valfritt antal av ett mönster, till exempel `.*` för att matcha valfritt antal av valfritt tecken
- `...+` matchar en eller flera av ett mönster, till exempel `\d+` för att matcha ett antal siffror större än noll
- `...{N}` matchar exakt N av ett mönster, till exempel `\d{4}` för 4 siffror
- `\.` matchar ett bokstavligt "."
- `\\` matchar ett bokstavligt "\\"
- `^` matchar början av raden
- `$` matchar slutet av raden

## Fångstgrupper och referenser

Om du använder regex-grupper `(...)` kan du referera till delmängder av matchningen för extrahering eller sök-och-ersätt.
För att till exempel extrahera bara månaden från ett datum i stil med YYYY-MM-DD kan du använda följande Python-kod:

```python
>>> import re
>>> re.match(r"\d{4}-(\d{2})-\d{2}", "2026-01-14").group(1)
'01'
```

I din textredigerare kan du använda referenser till fångstgrupper i ersättningsmönster.
Syntaxen kan variera mellan IDE:er.
I VS Code kan du till exempel använda variabler som `$1`, `$2` och så vidare, och i Vim kan du använda `\1`, `\2` och så vidare för att referera till grupper.

## Begränsningar

[Reguljära språk](https://en.wikipedia.org/wiki/Regular_language) är kraftfulla men begränsade.
Det finns klasser av strängar som inte kan uttryckas med standardregex (t.ex. är det [inte möjligt](https://en.wikipedia.org/wiki/Pumping_lemma_for_regular_languages) att skriva ett reguljärt uttryck som matchar mängden strängar {a^n b^n \| n &ge; 0}, alltså mängden strängar med ett antal "a" följt av samma antal "b", och mer praktiskt sett är språk som HTML inte reguljära språk).
I praktiken stöder moderna regex-motorer funktioner som lookahead och backreferences som utökar stödet bortom reguljära språk, och de är extremt användbara i praktiken, men det är viktigt att veta att de fortfarande är begränsade i uttryckskraft.
För mer avancerade språk kan du behöva använda en kraftfullare typ av tolk (se till exempel [pyparsing](https://github.com/pyparsing/pyparsing), en [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar)-tolk).

## Lära sig regex

Vi rekommenderar att du lär dig grunderna (det vi har täckt i föreläsningen) och sedan tittar i regex-referenser när du behöver dem, i stället för att memorera hela språket.

Samtalsbaserade AI-verktyg kan vara effektiva för att hjälpa dig skapa regex-mönster.
Prova till exempel att fråga din favorit-LLM med följande fråga:

```
Write a Python-style regex pattern that matches the requested path from log lines from Nginx.
Here is an example log line:

169.254.1.1 - - [09/Jan/2026:21:28:51 +0000] "GET /feed.xml HTTP/2.0" 200 2995 "-" "python-requests/2.32.3"
```

# Övningar

1. Konfigurera en formaterare, en linter och pre-commit-krokar för ett projekt du arbetar med.
   Om du har många fel bör autoformatering ta hand om formateringsfelen.
   För linterfelen kan du prova att använda en [AI-agent]({{ '/2026/agentic-coding/' | relative_url }}) för att fixa alla linterfel.
   Se till att AI-agenten kan köra lintern och observera resultaten, så att den kan arbeta iterativt för att fixa alla problem.
   Granska resultaten noga för att säkerställa att AI inte förstör din kod.
1. Lär dig ett testbibliotek för ett språk du kan och skriv ett enhetstest för ett projekt du arbetar med.
   Kör ett verktyg för kodtäckning, generera en HTML-formaterad täckningsrapport och studera resultatet.
   Kan du hitta raderna som täcks?
   Din kodtäckning blir sannolikt väldigt låg.
   Prova att manuellt skriva några tester för att förbättra den.
   Prova att använda en [AI-agent]({{ '/2026/agentic-coding/' | relative_url }}) för att förbättra täckningen.
   Se till att kodagenten kan köra tester med täckning och producera en rad-för-rad-rapport, så att den vet var den ska fokusera.
   Är de AI-genererade testerna faktiskt bra?
1. Sätt upp kontinuerlig integration som körs varje gång du skickar kod för ett projekt du arbetar med.
   Låt CI köra formatering, lintning och tester.
   Bryt din kod med flit (t.ex. genom att introducera en linteröverträdelse), och säkerställ att CI fångar det.
1. Prova att skriva ett [regex-mönster](#reguljära-uttryck) och använd kommandoradsverktyget `grep` [kommandoradsverktyg]({{ '/2026/course-shell/' | relative_url }}) för att hitta förekomster av `subprocess.Popen(..., shell=True)` i din kod.
   Försök sedan att "bryta" regex-mönstret.
   Matchar [semgrep](#lintning) fortfarande korrekt den farliga kod som gör att ditt grep-anrop missar?
1. Öva regex-sök-och-ersätt i din IDE eller textredigerare genom att ersätta `-` [Markdown-punktlistemarkörer](https://spec.commonmark.org/0.31.2/#bullet-list-marker) med `*` i [dessa föreläsningsanteckningar](https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/code-quality.md).
   Observera att det vore fel att bara ersätta alla "-" i filen, eftersom tecknet används i många sammanhang som inte är punktlistemarkörer.
1. Skriv ett regex för att ur JSON-strukturer av formen `{"name": "Alyssa P. Hacker", "college": "MIT"}` fånga namnet (t.ex. `Alyssa P. Hacker` i detta exempel).
   Tips: i ditt första försök kan du råka skriva ett regex som extraherar `Alyssa P. Hacker", "college": "MIT`.
   Läs om giriga kvantifierare i [Python regex-dokumentationen](https://docs.python.org/3/library/re.html) för att förstå hur du fixar det.
    1. Få regex-mönstret att fungera även när namnet innehåller tecknet `"` (dubbla citattecken kan skrivas med undantagstecken i JSON: `\"`).
    1. Vi **rekommenderar inte** att använda reguljära uttryck för avancerade tolkningsproblem i praktiken.
    1. Ta reda på hur du använder ditt programmeringsspråks JSON-tolk för denna uppgift.
    1. Skriv ett kommandoradsprogram som tar en JSON-struktur av formen ovan på stdin och skriver ut namnet på stdout.
    1. Du behöver sannolikt bara några få rader kod.
    1. I Python kan du göra det enkelt på en enda rad kod utöver `import json`.
