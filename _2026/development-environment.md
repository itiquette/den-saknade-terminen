---
layout: lecture
title: "Utvecklingsmiljö och verktyg"
description: >
  Lär dig om IDE:er, Vim, språkservrar och AI-drivna utvecklingsverktyg.
thumbnail: /static/assets/thumbnails/2026/lec3.png
date: 2026-01-14
ready: true
video:
  aspect: 56.25
  id: QnM1nVzrkx8
---

En _utvecklingsmiljö_ är en uppsättning verktyg för att utveckla programvara.
Kärnan i en utvecklingsmiljö är textredigering, tillsammans med funktioner som syntaxmarkering, typkontroll, kodformatering och autokomplettering.
_Integrerade utvecklingsmiljöer_ (IDE:er) som [VS Code][vs-code] samlar all denna funktionalitet i en enda applikation.
Terminalbaserade arbetsflöden för utveckling kombinerar verktyg som [tmux](https://github.com/tmux/tmux) (en terminalmultiplexer), [Vim](https://www.vim.org/) (en textredigerare), [Zsh](https://www.zsh.org/) (ett skal) och språkspecifika kommandoradsverktyg, som [Ruff](https://docs.astral.sh/ruff/) (en Python-linter och kodformatterare) och [Mypy](https://mypy-lang.org/) (en typkontroll för Python).

IDE:er och terminalbaserade arbetsflöden har båda sina styrkor och svagheter.
Grafiska IDE:er kan till exempel vara lättare att lära sig, och dagens IDE:er har i allmänhet bättre AI-integration direkt ur lådan, som AI-autokomplettering.
Terminalbaserade arbetsflöden är å andra sidan lätta och kan vara ditt enda alternativ i miljöer där du inte har ett GUI eller inte kan installera programvara.
Vi rekommenderar att du skaffar grundläggande vana vid båda och uppnår god behärskning av minst ett av dem.
Om du inte redan har en föredragen IDE rekommenderar vi att börja med [VS Code][vs-code].

I den här föreläsningen går vi igenom:

- [Textredigering och Vim](#textredigering-och-vim)
- [Kodintelligens och språkservrar](#kodintelligens-och-språkservrar)
- [AI-driven utveckling](#ai-driven-utveckling)
- [Tillägg och annan IDE-funktionalitet](#tillägg-och-annan-ide-funktionalitet)

[vs-code]: https://code.visualstudio.com/

# Textredigering och Vim

När du programmerar lägger du mest tid på att navigera i kod, läsa kodsnuttar och göra ändringar i kod, snarare än att skriva långa obrutna textstycken eller läsa filer uppifrån och ner.
[Vim] är en textredigerare som är optimerad för just den här fördelningen av uppgifter.

**Vims filosofi.** Vim bygger på en vacker idé: gränssnittet är i sig ett programmeringsspråk, utformat för att navigera och redigera text.
Tangenttryckningar (med minnesvänliga namn) är kommandon, och dessa kommandon går att komponera.
Vim undviker musen eftersom den är för långsam.
Vim undviker till och med piltangenterna eftersom de kräver för mycket rörelse.
Resultatet är en redigerare som känns som ett hjärn-datorgränssnitt och matchar hastigheten i ditt tänkande.

**Vim-stöd i annan programvara.** Du behöver inte använda [Vim] självt för att dra nytta av idéerna i dess kärna.
Många program som innehåller textredigering har ett "Vim-läge", antingen inbyggt eller som insticksmodul.
VS Code har till exempel insticksmodulen [VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim), Zsh har [inbyggt stöd](https://zsh.sourceforge.io/Guide/zshguide04.html) för Vim-emulering, och till och med Claude Code har [inbyggt stöd](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode) för Vim-redigerarläge.
Chansen är stor att de verktyg du använder för textredigering stöder Vim-läge på något sätt.

## Modalt redigerande

Vim är en _modal redigerare_: den har olika arbetslägen för olika typer av uppgifter.

- **Normal**: för att flytta runt i en fil och göra ändringar
- **Insert**: för att infoga text
- **Replace**: för att ersätta text
- **Visual** (vanligt, rad eller block): för att markera textblock
- **Command-line**: för att köra ett kommando

Tangenttryckningar betyder olika saker i olika lägen.
Bokstaven `x` i Insert-läge skriver till exempel bara in tecknet "x", men i Normal-läge raderar den tecknet under markören, och i Visual-läge raderar den markeringen.

I standardkonfigurationen visar Vim aktuellt läge längst ned till vänster.
Start-/standardläget är Normal-läge.
Du kommer oftast att växla mellan Normal-läge och Insert-läge.

Du byter läge genom att trycka `<ESC>` (escape-tangenten) för att gå tillbaka till Normal-läge från vilket läge som helst.
Från Normal-läge går du till Insert med `i`, Replace med `R`, Visual med `v`, Visual Line med `V`, Visual Block med `<C-v>` (Ctrl-V, ibland skrivet `^V`) och Command-line med `:`.

Du använder `<ESC>` mycket i Vim.
Överväg att mappa om Caps Lock till Escape ([instruktioner för macOS](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)) eller skapa en [alternativ mappning](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings) för `<ESC>` med en enkel tangentsekvens.

## Grunderna: infoga text

Tryck `i` från Normal-läge för att gå till Insert-läge.
Nu fungerar Vim som vilken annan textredigerare som helst, tills du trycker `<ESC>` för att gå tillbaka till Normal-läge.
Detta, tillsammans med grunderna ovan, räcker för att börja redigera filer med Vim.
Det är dock inte särskilt effektivt om du tillbringar all tid i Insert-läge.

## Vims gränssnitt är ett programmeringsspråk

Vims gränssnitt är ett programmeringsspråk.
Tangenttryckningar (med minnesvänliga namn) är kommandon, och dessa kommandon _komponeras_.
Det möjliggör effektiv förflyttning och redigering, särskilt när kommandona sitter i muskelminnet, ungefär som att skrivande blir snabbt när du lärt dig tangentbordslayouten.

### Rörelse

Du bör tillbringa merparten av tiden i Normal-läge och använda rörelsekommandon för att navigera i filen.
Rörelser i Vim kallas också "substantiv", eftersom de syftar på textstycken.

- Grundrörelser: `hjkl` (vänster, ner, upp, höger)
- Ord: `w` (nästa ord), `b` (början av ord), `e` (slutet av ord)
- Rader: `0` (början av rad), `^` (första icke-blanktecken), `$` (slutet av rad)
- Skärm: `H` (överst på skärmen), `M` (mitt på skärmen), `L` (nederst på skärmen)
- Scroll: `Ctrl-u` (upp), `Ctrl-d` (ner)
- Fil: `gg` (början av fil), `G` (slutet av fil)
- Radnummer: `:{number}<CR>` eller `{number}G` (rad {number})
    - `<CR>` syftar på carriage return / enter-tangenten
- Övrigt: `%` (matchande tecken, som parentes eller klammer)
- Sök tecken: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - find/to framåt/bakåt efter {character} på aktuell rad
    - `,` / `;` för att hoppa mellan träffar
- Sökning: `/{regex}`, `n` / `N` för att navigera mellan träffar

### Markering

Visual-lägen:

- Visual: `v`
- Visual Line: `V`
- Visual Block: `Ctrl-v`

Du kan använda rörelsetangenterna för att göra en markering.

### Redigeringar

Allt du brukade göra med musen gör du nu med tangentbordet via redigeringskommandon som komponerar med rörelsekommandon.
Här börjar Vims gränssnitt verkligen likna ett programmeringsspråk.
Vims redigeringskommandon kallas också "verb", eftersom verb agerar på substantiv.

- `i` gå till Insert-läge
    - men för textmanipulation/radering vill du använda något bättre än backsteg
- `o` / `O` infoga rad under / över
- `d{motion}` radera {motion}
    - t.ex. `dw` är radera ord, `d$` är radera till radslut, `d0` är radera till radbörjan
- `c{motion}` ändra {motion}
    - t.ex. `cw` är ändra ord
    - motsvarar `d{motion}` följt av `i`
- `x` radera tecken (motsvarar `dl`)
- `s` ersätt tecken (motsvarar `cl`)
- Visual-läge + redigering
    - markera text, `d` för att radera eller `c` för att ändra
- `u` för ångra, `<C-r>` för gör om
- `y` för kopiera / "yanka" (vissa andra kommandon som `d` kopierar också)
- `p` för klistra in
- Mycket mer att lära: till exempel växlar `~` skiftläge på ett tecken, och `J` fogar samman rader

### Antal

Du kan kombinera substantiv och verb med ett antal, vilket utför en åtgärd ett visst antal gånger.

- `3w` flytta 3 ord framåt
- `5j` flytta 5 rader nedåt
- `7dw` radera 7 ord

### Modifierare

Du kan använda modifierare för att ändra betydelsen av ett substantiv.
Två vanliga modifierare är `i`, som betyder "inner" eller "inside", och `a`, som betyder "around".

- `ci(` ändra innehållet innanför aktuellt parentespar
- `ci[` ändra innehållet innanför aktuellt hakparentespar
- `da'` radera en enkelciterad sträng, inklusive omgivande enkla citationstecken

## Sätt ihop allt

Här är en trasig implementation av [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz):

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20)
```

Vi använder följande kommandosekvens för att rätta felen, med start i Normal-läge:

- Main anropas aldrig
    - `G` för att hoppa till slutet av filen
    - `o` för att **o**ppna en ny rad under
    - skriv `if __name__ == "__main__": main()`
        - om redigeraren har Python-stöd kan den autoindenta i Insert-läge
    - `<ESC>` för att gå tillbaka till Normal-läge
- Börjar på 0 i stället för 1
    - `/` följt av `range` och `<CR>` för att söka efter "range"
    - `ww` för att flytta två **w**ord framåt (du kan också använda `2w`, men i praktiken är det vanligt att upprepa tangenten vid små antal)
    - `i` för att gå till **i**nsert-läge och lägg till `1,`
    - `<ESC>` för att gå tillbaka till Normal-läge
    - `e` för att hoppa till **e**nd of word för nästa ord
    - `a` för att börja **a**ppendera text, och lägg till `+ 1`
    - `<ESC>` för att gå tillbaka till Normal-läge
- Skriver "fizz" för multiplar av 5
    - `:6<CR>` för att gå till rad 6
    - `ci"` för att **c**hange **i**nside `"`, ändra till `"buzz"`
    - `<ESC>` för att gå tillbaka till Normal-läge

## Lära sig Vim

Det bästa sättet att lära sig Vim är att lära sig grunderna (det vi gått igenom hittills) och sedan aktivera Vim-läge i all din programvara och börja använda det i praktiken.
Undvik frestelsen att använda musen eller piltangenterna.
I vissa redigerare kan du avbinda piltangenterna för att tvinga fram bättre vanor.

### Ytterligare resurser

- [Vim-föreläsningen]({{ '/2020/editors/' | relative_url }}) från en tidigare iteration av kursen --- där går vi djupare i Vim
- `vimtutor` är en handledning som följer med Vim --- om Vim är installerat bör du kunna köra `vimtutor` från skalet
- [Vim Adventures](https://vim-adventures.com/) är ett spel för att lära sig Vim
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) har olika Vim-tips
- [VimGolf](https://www.vimgolf.com/) är [code golf](https://en.wikipedia.org/wiki/Code_golf), men där programmeringsspråket är Vims UI
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (bok)

[Vim]: https://www.vim.org/

<span id="code-intelligence-and-language-servers"></span>
# Kodintelligens och språkservrar

IDE:er erbjuder vanligtvis språkspecifikt stöd som kräver semantisk förståelse av koden, via IDE-tillägg som ansluter till _språkservrar_ som implementerar [Language Server Protocol](https://microsoft.github.io/language-server-protocol/).
[Python-tillägget för VS Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python) förlitar sig till exempel på [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance), och [Go-tillägget för VS Code](https://marketplace.visualstudio.com/items?itemName=golang.go) förlitar sig på förstapartsverktyget [gopls](https://go.dev/gopls/).
Genom att installera tillägg och språkserver för de språk du arbetar med kan du aktivera många språkspecifika funktioner i din IDE, till exempel:

- **Kodkomplettering.** Bättre autokomplettering och förslag, som att kunna se ett objekts fält och metoder efter att du skrivit `object.`.
- **Inline-dokumentation.** Se dokumentation vid hover och i förslag.
- **Hoppa till definition.** Hoppa från ett användningsställe till definitionen, som att gå från fältreferensen `object.field` till fältets definition.
- **Hitta referenser.** Motsatsen till ovan, hitta alla ställen där ett visst objekt, till exempel ett fält eller en typ, refereras.
- **Hjälp med importer.** Organisera importer, ta bort oanvända importer, flagga saknade importer.
- **Kodkvalitet.** Dessa verktyg kan användas fristående, men funktionaliteten tillhandahålls ofta av språkservrar också.
Kodformatering autoindenterar och autoformatterar kod, och typkontroller och linters hittar fel i koden medan du skriver.
Vi går djupare i den här klassen av funktionalitet i [föreläsningen om kodkvalitet]({{ '/2026/code-quality/' | relative_url }}).

## Konfigurera språkservrar

För vissa språk räcker det att installera tillägget och språkservern.
För andra språk behöver du tala om för IDE:n hur din miljö ser ut för att få maximal nytta av språkservern.
Att koppla VS Code till din [Python-miljö](https://code.visualstudio.com/docs/python/environments) gör till exempel att språkservern kan se dina installerade paket.
Miljöer behandlas mer ingående i [föreläsningen om paketering och leverans av kod]({{ '/2026/shipping-code/' | relative_url }}).

Beroende på språk kan det finnas inställningar att konfigurera för språkservern.
Med Python-stöd i VS Code kan du till exempel stänga av statisk typkontroll för projekt som inte använder Pythons valfria typannoteringar.

<span id="ai-powered-development"></span>
# AI-driven utveckling

Sedan [GitHub Copilot][github-copilot] med OpenAI:s [Codex-modell](https://openai.com/index/openai-codex/) introducerades i mitten av 2021 har [LLM:er](https://en.wikipedia.org/wiki/Large_language_model) blivit brett använda inom programvaruutveckling.
Det finns just nu tre huvudsakliga format: autokomplettering, inbäddad chatt och kodagenter.

[github-copilot]: https://github.com/features/copilot/ai-code-editor

## Autokomplettering

AI-driven autokomplettering har samma form som traditionell autokomplettering i din IDE och föreslår kompletteringar vid markörens position medan du skriver.
Ibland används den passivt så att den "bara fungerar".
Utöver det styrs AI-autokomplettering vanligtvis med kodkommentarer.

Låt oss till exempel skriva ett skript som laddar ner innehållet i de här föreläsningsanteckningarna och extraherar alla länkar.
Vi kan börja med:

```python
import requests

def download_contents(url: str) -> str:
```

Modellen autokompletterar funktionens kropp:

```python
    response = requests.get(url)
    return response.text
```

Vi kan styra kompletteringar ytterligare med kommentarer.
Om vi till exempel börjar skriva en funktion som ska extrahera alla Markdown-länkar, men funktionen inte har ett särskilt beskrivande namn:

```python
def extract(contents: str) -> list[str]:
```

kommer modellen att autokomplettera något i stil med detta:

```python
    lines = contents.splitlines()
    return [line for line in lines if line.strip()]
```

Vi kan styra kompletteringen med kodkommentarer:

```python
def extract(content: str) -> list[str]:
    # extract all Markdown links from the content
```

Den här gången ger modellen en bättre komplettering:

```python
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)
```

Här ser vi en nackdel med det här AI-kodverktyget: det kan bara ge kompletteringar vid markören.
I det här fallet hade det varit bättre att lägga `import re` på modulnivå i stället för inuti funktionen.

Exemplet ovan använde ett dåligt funktionsnamn för att visa hur kodkomplettering kan styras med kommentarer.
I praktiken vill du skriva kod med mer beskrivande funktionsnamn, som `extract_links`, och du vill skriva docstrings.
Baserat på det bör modellen ge en liknande komplettering som ovan.

För demonstrationssyfte kan vi färdigställa skriptet:

```python
print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

## Inbäddad chatt

Med inbäddad chatt kan du markera en rad eller ett block och sedan be AI-modellen att direkt föreslå en ändring.
I detta interaktionsläge kan modellen ändra befintlig kod, vilket skiljer sig från autokomplettering som bara kompletterar kod bortom markören.

Om vi fortsätter på exemplet ovan och bestämmer oss för att inte använda tredjepartsbiblioteket `requests`, kan vi markera de relevanta tre kodraderna, öppna inbäddad chatt och skriva något i stil med:

```
use built-in libraries instead
```

Modellen föreslår:

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')
```

## Kodagenter

Kodagenter behandlas mer ingående i föreläsningen om [agentdriven kodning]({{ '/2026/agentic-coding/' | relative_url }}).

## Rekommenderad programvara

Några populära AI-IDE:er är [VS Code][vs-code] med tillägget [GitHub Copilot][github-copilot] och [Cursor](https://cursor.com/).
GitHub Copilot finns just nu [gratis för studenter](https://github.com/education/students), lärare och förvaltare av populära öppen källkod-projekt.
Det här området utvecklas snabbt.
Många av de ledande produkterna har ungefär likvärdig funktionalitet.

# Tillägg och annan IDE-funktionalitet

IDE:er är kraftfulla verktyg, och blir ännu kraftfullare med _tillägg_.
Vi kan inte täcka allt i en enda föreläsning, men här ger vi några ingångar till populära tillägg.
Vi uppmuntrar dig att utforska området själv.
Det finns många listor över populära IDE-tillägg på nätet, till exempel [Vim Awesome](https://vimawesome.com/) för Vim-insticksmoduler och [VS Code-tillägg sorterade efter popularitet](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Installs).

- [Utvecklingscontainrar](https://containers.dev/): stöds av populära IDE:er (t.ex. [stöds av VS Code](https://code.visualstudio.com/docs/devcontainers/containers)), och låter dig använda en container för att köra utvecklingsverktyg.
Detta kan vara hjälpsamt för portabilitet eller isolering.
[Föreläsningen om paketering och leverans av kod]({{ '/2026/shipping-code/' | relative_url }}) går djupare på containrar.
- Fjärrutveckling: utveckla på en fjärrmaskin via SSH (t.ex. med [Remote SSH-insticksmodulen för VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)).
Det kan vara praktiskt om du till exempel vill utveckla och köra kod på en kraftig GPU-maskin i molnet.
- Samarbetsredigering: redigera samma fil i Google Docs-stil (t.ex. med [Live Share-insticksmodulen för VS Code](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)).

# Övningar

1. Aktivera Vim-läge i all programvara du använder som stöder det, till exempel redigeraren och skalet, och använd Vim-läge för all textredigering den kommande månaden.
När något känns ineffektivt, eller när du tänker "det måste finnas ett bättre sätt", prova att googla.
Det finns sannolikt ett bättre sätt.
1. Genomför en utmaning från [VimGolf](https://www.vimgolf.com/).
1. Konfigurera ett IDE-tillägg och en språkserver för ett projekt du arbetar med.
Säkerställ att förväntad funktionalitet, till exempel hoppa till definition även för biblioteksberoenden, fungerar som den ska.
Om du inte har kod att använda till övningen kan du ta ett öppen källkod-projekt från GitHub (som [det här](https://github.com/spf13/cobra)).
1. Bläddra i en lista över IDE-tillägg och installera ett som verkar användbart för dig.
