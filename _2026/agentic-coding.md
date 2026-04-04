---
layout: lecture
title: "Agentdriven kodning"
description: >
  Lär dig hur du använder AI-kodagenter effektivt för uppgifter inom programvaruutveckling.
thumbnail: /static/assets/thumbnails/2026/lec7.png
date: 2026-01-21
ready: true
video:
  aspect: 56.25
  id: sTdz6PZoAnw
---

Kodagenter är konversationella AI-modeller med tillgång till verktyg som läsning/skrivning av filer, webbsökning och körning av skalkommandon.
De finns i IDE:n, i fristående kommandorads- eller GUI-verktyg.
Kodagenter är autonoma och kraftfulla verktyg som möjliggör många olika användningsfall.

Föreläsningen bygger vidare på materialet om AI-stödd utveckling från föreläsningen [Utvecklingsmiljö och verktyg]({{ '/2026/development-environment/' | relative_url }}).
Som en kort demonstration fortsätter vi med exemplet från avsnittet [AI-stödd utveckling]({{ '/2026/development-environment/#ai-powered-development' | relative_url }}):

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

Vi kan prova att ge en kodagent följande uppgift:

```
Gör om detta till ett riktigt kommandoradsprogram, med argparse för argumentparsning.
Lägg till typannoteringar och se till att programmet klarar typkontroll.
```

Agenten kommer att läsa filen för att förstå den, sedan göra ändringar och till sist köra typkontrollen för att säkerställa att typannoteringarna är korrekta.
Om den gör ett misstag som gör att typkontrollen misslyckas kommer den sannolikt att iterera, även om det här är en enkel uppgift där det är mindre troligt.
Eftersom kodagenter har tillgång till verktyg som kan vara skadliga ber agentramverk som standard användaren att bekräfta verktygsanrop.

> Om kodagenten gör ett misstag --- till exempel om du har `mypy`-binären direkt tillgänglig på `$PATH` men agenten försöker köra `python -m mypy` --- kan du ge textåterkoppling för att hjälpa den kurskorrigera.

Kodagenter stöder interaktion i flera turer, så du kan iterera över arbetet i en fram-och-tillbaka-konversation med agenten.
Du kan till och med avbryta agenten om den är på väg åt fel håll.
En hjälpsam mental modell är att tänka på dig själv som chef för en praktikant: praktikanten gör grovjobbet men behöver vägledning och gör ibland fel som måste rättas.

> För en tydligare demonstration kan du som uppföljning be agenten köra det resulterande skriptet.
> Observera utdata och be den göra en ändring (t.ex. att enbart ta med absoluta URL:er).

# Hur AI-modeller och agenter fungerar

Att fullständigt förklara det inre arbetssättet i moderna [stora språkmodeller (LLM:er)](https://en.wikipedia.org/wiki/Large_language_model) och infrastruktur som agentramverk ligger utanför kursens omfång.
Det är dock hjälpsamt att ha en övergripande förståelse för några nyckelidéer för att effektivt _använda_ den här tekniken i framkant och förstå dess begränsningar.

LLM:er kan ses som modeller av sannolikhetsfördelningen för kompletteringssträngar (utdata) givet promptsträngar (indata).
LLM-inferens (det som händer när du t.ex. skickar en fråga till en konversationsapp) _drar stickprov_ från denna sannolikhetsfördelning.
LLM:er har ett fast _kontextfönster_, den maximala längden på in- och utsträngarna.

{% comment %}
> I matematisk notation modellerar LLM:en sannolikhetsfördelningen $\pi_\theta$ för fullföljanden $y$ givet prompts $x$, och vi samplar från denna fördelning: $\hat{y} \sim \pi_\theta(\cdot \mid x)$.
{% endcomment %}

AI-verktyg som konversationschatt och kodagenter bygger ovanpå denna grundmekanism.
För interaktioner i flera turer använder chattappar och agenter turmarkörer och skickar hela konversationshistoriken som promptsträng varje gång det kommer en ny användarfråga, vilket kör LLM-inferens en gång per användarfråga.
För verktygsanropande agenter tolkar ramverket vissa LLM-utdata som förfrågningar om att anropa ett verktyg, och ramverket skickar tillbaka resultatet av verktygsanropet till modellen som en del av promptsträngen (så LLM-inferens körs igen vid varje verktygsanrop och svar).
Kärnkoncepten i verktygsanropande agenter kan [implementeras på 200 rader kod](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/).

## Integritet

De flesta AI-kodverktyg i standardkonfiguration skickar mycket av din data till molnet.
Ibland kör ramverket lokalt medan LLM-inferensen kör i molnet.
Andra gånger körs ännu mer av programvaran i molnet (och t.ex. kan tjänsteleverantören i praktiken få en kopia av hela ditt kodförråd och alla interaktioner du har med AI-verktyget).

Det finns AI-kodverktyg med öppen källkod och öppna LLM:er som är ganska bra (även om de inte är fullt lika bra som de proprietära modellerna), men i dagsläget är det för de flesta användare inte praktiskt möjligt att köra de mest avancerade öppna LLM:erna lokalt på grund av hårdvarubegränsningar.

# Användningsfall

Kodagenter kan vara hjälpsamma för en stor variation av uppgifter.
Några exempel:

- **Implementera nya funktioner.** Som i exemplet ovan kan du be en kodagent att implementera en funktion.
  Att ge en bra specifikation är just nu mer konst än vetenskap.
  Du vill att indata till agenten ska vara tillräckligt beskrivande för att den ska göra det du vill (åtminstone vara på rätt spår så att du kan iterera), men inte så överdetaljerad att du gör för mycket av arbetet själv.
  Testdriven utveckling kan vara särskilt effektivt: skriv tester (eller använd kodagenten för att hjälpa dig skriva tester), granska dem så att de verkligen fångar det du vill och be sedan kodagenten implementera funktionen.
  Modeller förbättras kontinuerligt, så du behöver hålla din intuition uppdaterad om vad modellerna klarar. > Vi använde Claude Code för att [implementera](https://github.com/missing-semester/missing-semester/pull/345) dessa Tufte-liknande marginalnoter.
{%- comment %}
Ingen demo behövs här, eftersom introduktionen av en föreläsning redan var en liten demo av att lägga till en ny funktion.
{% endcomment %}
- **Fixa fel.** Om du har fel från kompilator, linter, typkontroll eller tester kan du be agenten rätta dem, till exempel med en uppmaning som "fixa problemen med mypy".
  Kodmodeller är särskilt effektiva när du kan få in dem i en återkopplingsslinga, så försök att sätta upp det så att modellen kan köra den felande kontrollen direkt, vilket låter den iterera autonomt.
  Om det är opraktiskt kan du ge modellen återkoppling manuellt. > I incheckningen [f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd) i Missing Semesters kodförråd bad vi Claude Code "Granska föreläsningen om agentdriven kodning för stavfel och grammatiska problem" och bad den därefter att åtgärda problemen den hittade, vilket lades in i [f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637).
{%- comment %}
Demo av en kodagent som åtgärdar programfelet i https://github.com/anishathalye/dotbot/commit/cef40c902ef0f52f484153413142b5154bbc5e99.

Skriv de misslyckade testerna för att demonstrera programfelet, och be sedan agenten fixa det.
Förberett i grenen demo-bugfix.

Det misslyckade testet kan köras med:

    hatch test tests/test_cli.py::test_issue_357

Du kan ge kodagenten den här uppmaningen:

    Det finns ett programfel som jag har skrivit ett misslyckat test för, och du kan reproducera det med `hatch test tests/test_cli.py::test_issue_357`.
    Åtgärda programfelet.

Få den att skapa en incheckning med ändringarna.
{% endcomment %}
- **Refaktorering.** Du kan använda kodagenter för att refaktorera kod på olika sätt, från enkla uppgifter som att byta namn på en metod (den typen av refaktorering stöds också av [kodintelligens]({{ '/2026/development-environment/#code-intelligence-and-language-servers' | relative_url }})) till mer komplexa uppgifter som att bryta ut funktionalitet till en separat modul. > Vi använde Claude Code för att [dela upp](https://github.com/missing-semester/missing-semester/pull/344) agentdriven kodning till en egen föreläsning.
{%- comment %}
Visa användning i Missing Semester, och påpeka att agenten gjorde några misstag.
{% endcomment %}
- **Kodgranskning.** Du kan be kodagenter granska kod.
  Du kan ge enkel vägledning, som "granska mina senaste ändringar som ännu inte ligger i en incheckning".
  Om du vill granska en ändringsförfrågan (PR) och din kodagent kan hämta webbsidor, eller om du har kommandoradsverktyg som [GitHub CLI](https://cli.github.com/) installerade, kan du kanske till och med be kodagenten "granska ändringsförfrågan {länk}" och låta den hantera resten.
{%- comment %}
I Porcupines kodförråd, ge agenten följande uppmaning:

    Granska denna PR: https://github.com/anishathalye/porcupine/pull/39
{% endcomment %}
- **Kodförståelse.** Du kan ställa frågor till en kodagent om en kodbas, vilket kan vara särskilt hjälpsamt när du är ny i ett projekt.
{%- comment %}
Några uppmaningar att prova i Missing Semesters kodförråd:

    Hur kör jag den här sajten lokalt?

    Hur är de sociala förhandsvisningskorten implementerade?
{% endcomment %}
- **Som ett skal.** Du kan be kodagenten använda ett visst verktyg för att lösa en uppgift, så att du kan köra skalkommandon med naturligt språk, till exempel "använd find-kommandot för att hitta alla filer äldre än 30 dagar" eller "använd mogrify för att ändra storlek på alla jpg-filer till 50 % av originalstorleken".
{%- comment %}
I Dotbots kodförråd, ge agenten följande uppmaning:

    Använd ag-kommandot för att hitta alla omdöpta importer i Python
{% endcomment %}
- **Vibekodning.** Agenter är tillräckligt kraftfulla för att du ska kunna implementera vissa applikationer utan att själv skriva en enda rad kod. > [Här är ett exempel](https://github.com/cleanlab/office-presence-dashboard) på ett verkligt projekt som en av instruktörerna vibekodade.
{%- comment %}
I Missing Semesters kodförråd, ge agenten följande uppmaning:

    Få den här sajten att se retro ut.
{% endcomment %}

# Avancerade agenter

Här ger vi en kort översikt över några mer avancerade användningsmönster och förmågor hos kodagenter.

- **Återanvändbara uppmaningar.** Skapa återanvändbara uppmaningar eller mallar.
  Du kan till exempel skriva en detaljerad uppmaning för kodgranskning på ett särskilt sätt och spara den som en återanvändbar uppmaning. > Agentverktyg utvecklas snabbt. > I vissa verktyg är återanvändbara uppmaningar som fristående funktion avvecklade. > I till exempel Codex och Claude Code [ingår de](https://developers.openai.com/codex/custom-prompts) i [färdigheter (_skills_)](https://code.claude.com/docs/en/skills).
- **Parallella agenter.** Kodagenter kan vara långsamma: du kan ge agenten en uppmaning och låta den arbeta på ett problem i tiotals minuter.
  Du kan köra flera kopior av agenter samtidigt, antingen på samma uppgift (LLM:er är stokastiska, så det kan vara hjälpsamt att köra samma sak flera gånger och välja bästa lösningen) eller på olika uppgifter (t.ex. implementera två icke-överlappande funktioner samtidigt).
  För att undvika att ändringar från olika agenter stör varandra kan du använda [git-arbetsträd](https://git-scm.com/docs/git-worktree), som vi tar upp i föreläsningen om [versionshantering]({{ '/2026/version-control/' | relative_url }}).
- **MCP:er.** MCP, som står för _Model Context Protocol_, är ett öppet protokoll som du kan använda för att koppla dina kodagenter till verktyg.
  Till exempel kan denna [Notion MCP-server](https://github.com/makenotion/notion-mcp-server) låta agenten läsa/skriva Notion-dokument, vilket möjliggör användningsfall som "läs specifikationen länkad i {Notion-dokument}, utarbeta en implementationsplan som en ny sida i Notion och implementera sedan en prototyp".
  För att hitta MCP:er kan du använda kataloger som [Pulse](https://www.pulsemcp.com/servers) och [Glama](https://glama.ai/mcp/servers).
- **Kontexthantering.** Som vi noterade [ovan](#hur-ai-modeller-och-agenter-fungerar) har LLM:erna som ligger bakom kodagenter ett begränsat _kontextfönster_.
  Effektiv användning av kodagenter kräver att du hanterar kontext väl.
  Du vill säkerställa att agenten har tillgång till informationen den behöver men undvika onödig kontext för att inte överfylla kontextfönstret eller försämra modellens prestanda (vilket ofta händer när kontextstorleken växer, även om den inte överstiger kontextfönstret).
  Agentramverk tillför och i viss grad hanterar kontext automatiskt, men mycket kontroll lämnas till användaren.
    - **Rensa kontextfönstret.** Den mest grundläggande kontrollen är att kodagenter stöder att rensa kontextfönstret (starta en ny konversation), vilket du bör göra för orelaterade frågor.
    - **Spola tillbaka konversationen.** Vissa kodagenter stöder att ångra steg i konversationshistoriken.
      I stället för att skicka ett uppföljningsmeddelande som styr agenten åt ett annat håll kan ett "undo" i vissa lägen hantera kontext mer effektivt.
{%- comment %}
Hitta på en snabb demo.
{% endcomment %}
    - **Kompaktering.** För att möjliggöra konversationer med obegränsad längd stöder kodagenter kontext-_kompaktering_: om konversationshistoriken blir för lång anropar de automatiskt en LLM för att sammanfatta början av konversationen och ersätter historiken med sammanfattningen.
      Vissa agenter ger användaren kontroll att utlösa kompaktering när det önskas.
{%- comment %}
Visa `/compact` i Claude Code, och visa hela sammanfattningen.
{% endcomment %}
    - **llms.txt.** Filen `/llms.txt` är en föreslagen [standardplats](https://llmstxt.org/) för ett dokument som LLM:er kan använda vid inferens.
      Produkter (t.ex. [cursor.com/llms.txt](https://cursor.com/llms.txt)), programvarubibliotek (t.ex. [ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt)) och API:er (t.ex. [apify.com/llms.txt](https://apify.com/llms.txt)) kan ha `llms.txt`-filer som är praktiska i utveckling.
      Sådana dokument är mer informationstäta per token och därmed mer kontexteffektiva än att be kodagenten hämta och läsa en HTML-sida.
      Extern dokumentation är användbar när kodagenten saknar inbyggd kunskap om ett beroende du försöker använda (t.ex. för att det publicerades efter LLM:ens kunskapsgräns).
{%- comment %}
Jämförelse sida vid sida i ett tomt kodförråd (på skrivbordet eller annan självbärande plats, med `git init` kört i det):

    Skriv ett exempelprogram i Python i en enda fil, demo.py, som använder semlib för att sortera "Ilya Sutskever", "Soumith Chintala" och "Donald Knuth" utifrån deras berömmelse som AI-forskare.

    Skriv ett exempelprogram i Python i en enda fil, demo.py, som använder semlib för att sortera "Ilya Sutskever", "Soumith Chintala" och "Donald Knuth" utifrån deras berömmelse som AI-forskare. Se https://semlib.anish.io/llms.txt. Följ länkar till Markdown-versioner av alla sidor som länkas från llms.txt-filer.

Inte säker på varför agenten inte gör detta som standard.
Du skulle förmodligen lägga den sista meningen i en CLAUDE.md-fil.
{% endcomment %}
    - **AGENTS.md.** De flesta kodagenter stöder [AGENTS.md](https://agents.md/) eller liknande (t.ex. letar Claude Code efter `CLAUDE.md`) som en README för kodagenter.
      När agenten startar förfyller den kontexten med hela innehållet i `AGENTS.md`.
      Du kan använda det för att ge agenten råd som gäller över sessioner (t.ex. instruera den att alltid köra typkontroll efter kodändringar, förklara hur man kör enhetstester eller länka tredjepartsdokumentation som agenten kan läsa).
      Vissa kodagenter kan autogenerera den här filen (t.ex. kommandot `/init` i Claude Code).
      Se [här](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md) för ett verkligt exempel på en `AGENTS.md`.
{%- comment %}
Dotbot-exempel, CLAUDE.md som innefattar @DEVELOPMENT.md och säger att man alltid ska köra typkontroll och kodformaterare efter ändringar i Python-kod.

Exempelprompt, utifrån master:

    Ta bort command-line-flaggan "--version".

Det här går snabbt och är bra för demonstrationssyfte.
{% endcomment %}
    - **Färdigheter (_skills_).** Innehåll i `AGENTS.md` laddas alltid, i sin helhet, in i agentens kontextfönster.
      _Färdigheter_ lägger till ett lager av indirektion för att undvika kontextuppblåsning: du kan ge agenten en lista med färdigheter och beskrivningar, och agenten kan "öppna" en färdighet (ladda den i sitt kontextfönster) vid behov.
    - **Underagenter.** Vissa kodagenter låter dig definiera underagenter, alltså agenter för uppgiftsspecifika arbetsflöden.
      Toppnivåagenten kan anropa en underagent för att lösa en viss uppgift, vilket gör att både toppnivåagenten och underagenten kan hantera kontext mer effektivt.
      Toppnivåagentens kontext sväller inte av allt underagenten ser, och underagenten kan få precis den kontext den behöver för uppgiften.
      Som exempel implementerar vissa kodagenter webbundersökning som en underagent: toppnivåagenten ställer en fråga till underagenten, som gör webbsökning, hämtar enskilda webbsidor, analyserar dem och returnerar ett svar till toppnivåagenten.
      På så sätt får toppnivåagenten inte sin kontext uppblåst av allt innehåll från hämtade webbsidor, och underagenten får inte resten av toppnivåagentens konversationshistorik i sin kontext.

För många av de avancerade funktioner som kräver att du skriver uppmaningar (t.ex. färdigheter eller underagenter) kan du använda LLM:er för att komma igång.
Vissa kodagenter har till och med inbyggt stöd för detta.
Till exempel kan Claude Code generera en underagent från en kort uppmaning (anropa `/agents` och skapa en ny agent).
Prova att skapa en underagent med följande uppmaning:

```
En Python-agent för kodkontroll som använder `mypy` och `ruff` för typkontroll, lintning och formatkontroll av alla filer som har ändrats sedan senaste git-incheckning.
```

Sedan kan du använda toppnivåagenten för att uttryckligen anropa underagenten med ett meddelande som "använd underagenten för kodkontroll".
Du kan också vid behov få toppnivåagenten att automatiskt anropa underagenten när det är lämpligt, till exempel efter att Python-filer har ändrats.

# Saker att se upp med

AI-verktyg kan göra misstag.
De bygger på LLM:er, som bara är probabilistiska modeller för nästa token.
De är inte "intelligenta" på samma sätt som människor.
Granska AI-utdata för korrekthet och säkerhetsfel.
Ibland kan det vara svårare att verifiera kod än att skriva koden själv.
För kritisk kod kan det vara bättre att skriva den för hand.
AI kan hamna på villovägar och försöka vilseleda dig, så var uppmärksam på felsökningsspiraler.
Använd inte AI som krycka, och var vaksam på överberoende eller ytlig förståelse.
Det finns fortfarande en stor klass av programmeringsuppgifter som AI ännu inte klarar.
Beräkningstänkande är fortfarande värdefullt.

# Rekommenderad programvara

Många IDE:er och AI-kodtillägg innehåller kodagenter (se rekommendationerna från [föreläsningen om utvecklingsmiljö]({{ '/2026/development-environment/' | relative_url }})).
Bland andra populära kodagenter finns Anthropics [Claude Code](https://www.claude.com/product/claude-code), OpenAI:s [Codex](https://openai.com/codex/) och agenter med öppen källkod som [opencode](https://github.com/anomalyco/opencode).

# Övningar

1. Jämför upplevelsen av att koda för hand, använda AI-autokomplettering, inbäddad chatt och agenter genom att göra samma programmeringsuppgift fyra gånger.
1. Den bästa kandidaten är en liten funktion i ett projekt du redan arbetar med.
1. Om du vill ha fler idéer kan du överväga att lösa uppgifter av typen "good first issue" i öppen källkod-projekt på GitHub, eller problem från [Advent of Code](https://adventofcode.com/) eller [LeetCode](https://leetcode.com/).
1. Använd en AI-kodagent för att navigera i en obekant kodbas.
1. Det fungerar bäst när du vill felsöka eller lägga till en ny funktion i ett projekt du faktiskt bryr dig om.
1. Om du inte kommer på något kan du prova att använda en AI-agent för att förstå hur säkerhetsrelaterade funktioner fungerar i agenten [opencode](https://github.com/anomalyco/opencode).
1. Vibekoda en liten app från grunden.
1. Skriv inte en enda rad kod för hand.
1. För den kodagent du föredrar, skapa och testa en `AGENTS.md` (eller motsvarande för din agent, som `CLAUDE.md`), en färdighet (t.ex. [_skill_ i Claude Code](https://code.claude.com/docs/en/skills) eller [_skill_ i Codex](https://developers.openai.com/codex/skills/)) och en underagent (t.ex. [underagenter i Claude Code](https://code.claude.com/docs/en/sub-agents)).
1. Fundera på när du vill använda den ena jämfört med den andra.
1. Observera att din valda kodagent kanske inte stöder alla dessa funktioner.
1. Du kan då antingen hoppa över dem eller prova en annan kodagent som har stöd.
1. Använd en kodagent för att uppnå samma mål som i regex-övningen om Markdown-punktlistor från [föreläsningen om kodkvalitet]({{ '/2026/code-quality/' | relative_url }}).
1. Löser den uppgifterna via direkta filändringar?
1. Vilka nackdelar och begränsningar finns med att låta en agent redigera filen direkt för att lösa en sådan uppgift?
1. Ta reda på hur du ska formulera uppmaningen så att agenten inte löser uppgiften via direkta filändringar.
1. Tips: be agenten använda ett av kommandoradsverktygen som nämns i [första föreläsningen]({{ '/2026/course-shell/' | relative_url }}).
1. De flesta kodagenter stöder någon form av "yolo mode" (t.ex. i Claude Code, `--dangerously-skip-permissions`).
1. Det är inte säkert att använda detta läge direkt, men det kan vara acceptabelt att köra en kodagent i en isolerad miljö som en virtuell maskin eller container och sedan aktivera autonom drift.
1. Få den här uppsättningen att fungera på din dator.
1. Dokumentation som [Claude Code devcontainers](https://code.claude.com/docs/en/devcontainer) eller [Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/agents/claude-code/) kan vara användbar.
1. Det finns flera sätt att komma igång på.
