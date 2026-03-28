---
layout: lecture
title: "Redigerare (Vim)"
description: >
  Lär dig använda Vim, en kraftfull textredigerare för effektiv kodredigering.
thumbnail: /static/assets/thumbnails/2020/lec3.png
date: 2020-01-15
ready: true
video:
  aspect: 56.25
  id: a6Q8Na575qc
---

Att skriva engelska ord och att skriva kod är två väldigt olika aktiviteter.
När du programmerar lägger du mer tid på att byta filer, läsa, navigera och redigera kod än på att skriva långa sammanhängande textstycken.
Det är rimligt att det finns olika typer av program för att skriva engelska ord respektive kod (t.ex. Microsoft Word jämfört med Visual Studio Code).

Som programmerare tillbringar vi största delen av tiden med att redigera kod, så det är värt att investera tid i att behärska en redigerare som passar dina behov.
Så här lär du dig en ny redigerare:

- Börja med en handledning (dvs. föreläsningen plus resurserna vi hänvisar till)
- Håll fast vid att använda redigeraren för all textredigering (även om det gör dig långsammare i början)
- Slå upp saker längs vägen: om det känns som att det borde finnas ett bättre sätt att göra något finns det förmodligen det

Om du följer metoden ovan och verkligen förbinder dig till att använda det nya programmet för all textredigering brukar inlärningskurvan för en avancerad textredigerare se ut ungefär så här.
På en eller två timmar lär du dig grundfunktioner som att öppna och redigera filer, spara/avsluta och navigera mellan buffertar.
Efter ungefär 20 timmar bör du vara lika snabb som du var i din gamla redigerare.
Efter det kommer vinsterna: du har tillräcklig kunskap och muskelminne för att den nya redigeraren faktiskt sparar tid.
Moderna textredigerare är avancerade och kraftfulla verktyg, så lärandet tar aldrig slut: du blir ännu snabbare ju mer du lär dig.

# Vilken redigerare ska man lära sig?

Programmerare har [starka åsikter](https://en.wikipedia.org/wiki/Editor_war) om sina textredigerare.

Vilka redigerare är populära i dag?
Se den här [Stack Overflow- undersökningen](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools) (det kan finnas viss snedvridning eftersom Stack Overflow-användare kanske inte är representativa för programmerare i stort).
[Visual Studio Code](https://code.visualstudio.com/) är den mest populära redigeraren.
[Vim](https://www.vim.org/) är den mest populära kommandoradsbaserade redigeraren.

## Vim

Alla lärare i kursen använder Vim som redigerare.
Vim har en rik historia; den härstammar från redigeraren Vi (1976) och utvecklas fortfarande i dag.
Vim bygger på flera riktigt smarta idéer, och därför stöder många verktyg ett Vim-emuleringsläge (till exempel har 1,4 miljoner personer installerat [Vim-emulering för VS code](https://github.com/VSCodeVim/Vim)).
Vim är sannolikt värt att lära sig även om du i slutänden byter till en annan textredigerare.

Det går inte att lära ut all funktionalitet i Vim på 50 minuter, så vi fokuserar på att förklara filosofin bakom Vim, lära ut grunderna, visa mer avancerade funktioner och resurser för att bemästra verktyget.

# Vims filosofi

När du programmerar lägger du största delen av tiden på att läsa och redigera, inte skriva.
Därför är Vim en _modal_ redigerare: den har olika lägen för att skriva in text respektive manipulera text.
Vim är programmerbar (med Vimscript och även andra språk som Python), och själva gränssnittet i Vim är ett programmeringsspråk: tangenttryckningar (med minnesvänliga namn) är kommandon, och kommandona är komponerbara.
Vim undviker musen eftersom den är för långsam; Vim undviker till och med piltangenterna eftersom de kräver för mycket handrörelse.

Slutresultatet är en redigerare som kan matcha hastigheten i ditt tänkande.

# Modal redigering

Vims design bygger på idén att en stor del av programmerartiden går till att läsa, navigera och göra små redigeringar, snarare än att skriva långa textströmmar.
Därför har Vim flera arbetslägen.

- **Normal**: för att röra dig i en fil och göra redigeringar
- **Infoga** (Insert): för att skriva in text
- **Ersätt** (Replace): för att ersätta text
- **Visuellt** (Visual) (vanligt, rad eller block): för att markera textblock
- **Kommandorad** (Command-line): för att köra ett kommando

Tangenttryckningar har olika betydelse i olika lägen.
Till exempel kommer bokstaven `x` i Infoga-läge bara att skriva in tecknet "x", men i Normal-läge tar den bort tecknet under markören, och i Visuellt-läge tar den bort markeringen.

I standardkonfigurationen visar Vim aktuellt läge nere till vänster.
Start-/standardläget är Normal-läge.
Du kommer i allmänhet att tillbringa mest tid mellan Normal-läge och Infoga-läge.

Du byter läge genom att trycka `<ESC>` (escape) för att gå från vilket läge som helst tillbaka till Normal-läge.
Från Normal-läge går du till Infoga med `i`, Ersätt med `R`, Visuellt med `v`, Visuell rad med `V`, Visuellt block med `<C-v>` (Ctrl-V, ibland också skrivet `^V`) och Kommandorad med `:`.

Du använder ofta `<ESC>` i Vim: överväg att mappa om Caps Lock till Escape ([macOS- instruktioner](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)) eller att skapa en [alternativ mappning](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings) för `<ESC>` med en enkel tangentsekvens.

# Grunder

## Skriva in text

Från Normal-läge trycker du `i` för att gå till Infoga-läge.
Nu beter sig Vim som vilken annan textredigerare som helst tills du trycker `<ESC>` för att återgå till Normal-läge.
Detta, tillsammans med grunderna ovan, är allt du behöver för att börja redigera filer i Vim (dock inte särskilt effektivt om du gör all redigering från Infoga-läge).

## Buffertar, flikar och fönster

Vim håller en uppsättning öppna filer som kallas "buffertar".
En Vim-session har ett antal flikar, där varje flik har ett antal fönster (delade paneler).
Varje fönster visar en enda buffert.
Till skillnad från andra program du känner till, som webbläsare, finns ingen 1-till-1-koppling mellan buffertar och fönster; fönster är bara vyer.
En given buffert kan vara öppen i _flera_ fönster, även inom samma flik.
Det kan vara mycket praktiskt, till exempel för att se två olika delar av samma fil samtidigt.

Som standard öppnar Vim med en enda flik som innehåller ett enda fönster.

## Kommandorad

Kommandoläget nås genom att skriva `:` i Normal-läge.
Markören hoppar då till kommandoraden längst ned på skärmen.
Detta läge har många funktioner, bland annat att öppna, spara och stänga filer samt [avsluta Vim](https://twitter.com/iamdevloper/status/435555976687923200).

- `:q` avsluta (stäng fönster)
- `:w` spara ("write")
- `:wq` spara och avsluta
- `:e {name of file}` öppna fil för redigering
- `:ls` visa öppna buffertar
- `:help {topic}` öppna hjälp
    - `:help :w` öppnar hjälp för kommandot `:w`
    - `:help w` öppnar hjälp för rörelsen `w`

# Vims gränssnitt är ett programmeringsspråk

Den viktigaste idén i Vim är att Vims gränssnitt i sig är ett programmeringsspråk.
Tangenttryckningar (med minnesvänliga namn) är kommandon, och kommandona _komponeras_.
Det möjliggör effektiv navigering och redigering, särskilt när kommandona sitter i muskelminnet.

## Rörelse

Du bör tillbringa största delen av tiden i Normal-läge och använda rörelsekommandon för att navigera i bufferten.
Rörelser i Vim kallas också "substantiv", eftersom de syftar på textenheter.

- Grundrörelse: `hjkl` (vänster, ned, upp, höger)
- Ord: `w` (nästa ord), `b` (början av ord), `e` (slutet av ord)
- Rader: `0` (början av rad), `^` (första icke-blanktecken), `$` (slutet av rad)
- Skärm: `H` (överkant), `M` (mitten), `L` (underkant)
- Rulla: `Ctrl-u` (upp), `Ctrl-d` (ned)
- Fil: `gg` (början av fil), `G` (slutet av fil)
- Radnummer: `:{number}<CR>` eller `{number}G` (rad {number})
- Övrigt: `%` (motsvarande objekt)
- Hitta: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - find/to framåt/bakåt {character} på aktuell rad
    - `,` / `;` för att navigera mellan träffar
- Sök: `/{regex}`, `n` / `N` för att navigera mellan träffar

## Markering

Visuella lägen:

- Visuellt: `v`
- Visuell rad: `V`
- Visuellt block: `Ctrl-v`

Du kan använda rörelsetangenter för att göra markering.

## Redigeringar

Allt du tidigare gjorde med musen gör du nu med tangentbordet, med redigeringskommandon som kan kombineras med rörelsekommandon.
Det är här Vims gränssnitt börjar likna ett programmeringsspråk.
Vims redigeringskommandon kallas också "verb", eftersom verb agerar på substantiv.

- `i` gå till Infoga-läge
    - men för att manipulera/ta bort text vill du använda något mer än backsteg
- `o` / `O` infoga rad under / över
- `d{motion}` ta bort {motion}
    - t.ex. `dw` är ta bort ord, `d$` är ta bort till radslut, `d0` är ta bort till radbörjan
- `c{motion}` ändra {motion}
    - t.ex. `cw` är ändra ord
    - som `d{motion}` följt av `i`
- `x` ta bort tecken (motsvarar `dl`)
- `s` ersätt tecken (motsvarar `cl`)
- Visuellt-läge + manipulation
    - markera text, `d` för att ta bort eller `c` för att ändra
- `u` för ångra, `<C-r>` för göra om
- `y` för kopiera / "yank" (vissa andra kommandon som `d` kopierar också)
- `p` för klistra in
- Mycket mer att lära: t.ex. `~` växlar versal/gemen för ett tecken

## Antal

Du kan kombinera substantiv och verb med ett antal, vilket gör att en åtgärd utförs flera gånger.

- `3w` flytta 3 ord framåt
- `5j` flytta 5 rader nedåt
- `7dw` ta bort 7 ord

## Modifierare

Du kan använda modifierare för att ändra innebörden av ett substantiv.
Några modifierare är `i`, som betyder "inner" eller "inside", och `a`, som betyder "around".

- `ci(` ändra innehållet inuti det aktuella parentesparet
- `ci[` ändra innehållet inuti det aktuella hakparentesparet
- `da'` ta bort en enkelciterad sträng inklusive omgivande enkla citationstecken

# Demo

Här är en trasig implementation av [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz):

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10)
```

Vi kommer att rätta följande problem:

- Main anropas aldrig
- Startar på 0 i stället för 1
- Skriver "fizz" och "buzz" på separata rader för multiplar av 15
- Skriver "fizz" för multiplar av 5
- Använder hårdkodat argument 10 i stället för att ta ett kommandoradsargument

{% comment %}
- main is never called
  - `G` end of file
  - `o` open new line below
  - type in "if __name__ ..." thing
- starts at 0 instead of 1
  - search for `/range`
  - `ww` to move forward 2 words
  - `i` to insert text, "1, "
  - `ea` to insert after limit, "+1"
- newline for "fizzbuzz"
  - `jj$i` to insert text at end of line
  - add ", end=''"
  - `jj.` to repeat for second print
  - `jjo` to open line below if
  - add "else: print()"
- fizz fizz
  - `ci'` to change fizz
- command-line argument
  - `ggO` to open above
  - "import sys"
  - `/10`
  - `ci(` to "int(sys.argv[1])"
{% endcomment %}

Se föreläsningsvideon för demonstrationen.
Jämför hur ändringarna ovan utförs med Vim med hur du hade gjort samma redigeringar i ett annat program.
Notera hur få tangenttryckningar som krävs i Vim, vilket låter dig redigera i den hastighet du tänker.

# Anpassa Vim

Vim anpassas via en textbaserad konfigurationsfil i `~/.vimrc` (som innehåller Vimscript-kommandon).
Det finns sannolikt många grundinställningar som du vill slå på.

Vi tillhandahåller en väl dokumenterad grundkonfiguration som du kan använda som startpunkt.
Vi rekommenderar att du använder den eftersom den rättar till en del av Vims udda standardbeteende.
**Ladda ner vår konfiguration [här]({{ '/2020/files/vimrc' | relative_url }}) och spara den som `~/.vimrc`.**

Vim är mycket anpassningsbart, och det är värt att lägga tid på att utforska anpassningsmöjligheter.
Du kan titta på andras dotfiles på GitHub för inspiration, till exempel lärarnas Vim-konfigurationer ([Anish](https://github.com/anishathalye/dotfiles/blob/master/vimrc), [Jon](https://github.com/jonhoo/configs/blob/master/editor/.config/nvim/init.lua) (använder [neovim](https://neovim.io/)), [Jose](https://github.com/JJGO/dotfiles/blob/master/vim/.vimrc)).
Det finns också många bra blogginlägg om ämnet.
Försök att inte kopiera och klistra in någons fulla konfiguration, utan läs den, förstå den och plocka det du behöver.

# Utöka Vim

Det finns massor av insticksmoduler för att utöka Vim.
I motsats till föråldrade råd du kan hitta på internet behöver du _inte_ använda en hanterare för insticksmoduler i Vim (sedan Vim 8.0).
I stället kan du använda det inbyggda pakethanteringssystemet.
Skapa helt enkelt katalogen `~/.vim/pack/vendor/start/` och lägg insticksmoduler där (t.ex. via `git clone`).

Här är några av våra favoritinsticksmoduler:

- [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): ungefärlig filsökare
- [ack.vim](https://github.com/mileszs/ack.vim): kodsökning
- [nerdtree](https://github.com/scrooloose/nerdtree): filutforskare
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): magiska rörelser

Vi försöker undvika att ge en överväldigande lång lista med insticksmoduler här.
Du kan titta i lärarnas dotfiles ([Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/JJGO/dotfiles)) för att se vilka andra insticksmoduler vi använder.
Kolla också in [Vim Awesome](https://vimawesome.com/) för fler bra Vim-insticksmoduler.
Det finns även mängder av blogginlägg om ämnet: sök till exempel på `Vim-insticksmoduler`.

# Vim-läge i andra program

Många verktyg stöder Vim-emulering.
Kvaliteten varierar från bra till mycket bra; beroende på verktyg kanske de inte stöder de mer avancerade Vim-funktionerna, men de flesta täcker grunderna väl.

## Skal

Om du använder Bash, använd `set -o vi`.
Om du använder Zsh, `bindkey -v`.
För Fish, `fish_vi_key_bindings`.
Dessutom kan du, oavsett skal, sätta `export EDITOR=vim`.
Det är miljövariabeln som används för att avgöra vilken redigerare som startas när ett program vill öppna en redigerare.
Till exempel använder `git` denna redigerare för incheckningsmeddelanden.

## Readline

Många program använder biblioteket [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) för sitt kommandoradsgränssnitt.
Readline stöder också (grundläggande) Vim-emulering, som kan aktiveras genom att lägga till följande rad i filen `~/.inputrc`:

```
set editing-mode vi
```

Med den här inställningen får till exempel Python-REPL stöd för Vim-bindningar.

## Övriga

Det finns till och med vim-tangentbindningstillägg för webbläsare [browsers](https://vim.fandom.com/wiki/Vim_key_bindings_for_web_browsers) - några populära är [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) för Google Chrome och [Tridactyl](https://github.com/tridactyl/tridactyl) för Firefox.
Du kan till och med få Vim-bindningar i [Jupyter notebooks](https://github.com/jupyterlab-contrib/jupyterlab-vim).
Här är en [lång lista](https://reversed.top/2016-08-13/big-list-of-vim-like-software) över program med vim-liknande tangentbindningar.

<span id="advanced-vim"></span>
# Avancerad Vim

Här är några exempel som visar kraften i redigeraren.
Vi kan inte lära ut alla sådana saker, men du lär dig dem längs vägen.
En bra tumregel: varje gång du använder redigeraren och tänker "det måste finnas ett bättre sätt att göra det här" så finns det förmodligen det; slå upp det på nätet.

## Sök och ersätt

Kommandot `:s` (substitute) ([dokumentation](https://vim.fandom.com/wiki/Search_and_replace)).

- `%s/foo/bar/g`
    - ersätt foo med bar globalt i filen
- `%s/\[.*\](\(.*\))/\1/g`
    - ersätt namngivna Markdown-länkar med rena URL:er

## Flera fönster

- `:sp` / `:vsp` för att dela fönster
- Kan ha flera vyer av samma buffert.

<span id="macros"></span>
## Makron

- `q{character}` för att börja spela in ett makro i register `{character}`
- `q` för att stoppa inspelningen
- `@{character}` spelar upp makrot
- Makrouppspelning stoppar vid fel
- `{number}@{character}` kör ett makro {number} gånger
- Makron kan vara rekursiva
    - rensa först makrot med `q{character}q`
    - spela in makrot med `@{character}` för att anropa makrot rekursivt
  (det blir en no-op tills inspelningen är klar)
- Exempel: konvertera xml till json ([fil]({{ '/2020/files/example-data.xml' | relative_url }}))
    - Array av objekt med nycklarna "name" / "email"
    - Använd ett Python-program?
    - Använd sed / regex
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - ...
    - Vim-kommandon / makron
        - `Gdd`, `ggdd` tar bort första och sista raden
        - Makro för att formatera ett enskilt element (register `e`)
            - Gå till raden med `<name>`
            - `qe^r"f>s": "<ESC>f<C"<ESC>q`
        - Makro för att formatera en person
            - Gå till raden med `<person>`
            - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        - Makro för att formatera en person och gå till nästa person
            - Gå till raden med `<person>`
            - `qq@pjq`
        - Kör makrot till filslut
            - `999@q`
        - Ta manuellt bort sista `,` och lägg till avgränsarna `[` och `]`

# Resurser

- `vimtutor` är en handledning som följer med Vim - om Vim är installerat bör du kunna köra `vimtutor` från skalet
- [Vim Adventures](https://vim-adventures.com/) är ett spel för att lära sig Vim
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) har olika Vim-tips
- [Vim Golf](https://www.vimgolf.com/) är [code golf](https://en.wikipedia.org/wiki/Code_golf), men där programmeringsspråket är Vims UI
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (bok)

# Övningar

1. Slutför `vimtutor`.
   Obs: den ser bäst ut i ett [80x24](https://en.wikipedia.org/wiki/VT100) (80 kolumner och 24 rader) terminalfönster.
1. Ladda ner vår [grundläggande vimrc]({{ '/2020/files/vimrc' | relative_url }}) och spara den som `~/.vimrc`.
   Läs igenom den välkommenterade filen (med Vim!) och observera hur Vim ser ut och beter sig något annorlunda med den nya konfigurationen.
1. Installera och konfigurera en insticksmodul: [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).
   1. Skapa katalogen för insticksmoduler med `mkdir -p ~/.vim/pack/vendor/start`
   1. Ladda ner insticksmodulen: `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`
   1. Läs [dokumentationen](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md) för insticksmodulen.
      Prova att använda CtrlP för att hitta en fil genom att gå till en projektkatalog, öppna Vim och använda Vim-kommandoraden för att starta `:CtrlP`.
    1. Anpassa CtrlP genom att lägga till [konfiguration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) i din `~/.vimrc` så att CtrlP öppnas med Ctrl-P.
1. För att öva Vim, gör om [Demo](#demo) från föreläsningen på din egen maskin.
1. Använd Vim för _all_ textredigering under nästa månad.
   När något känns ineffektivt, eller när du tänker "det måste finnas ett bättre sätt", prova att googla - det finns förmodligen ett.
   Om du fastnar, kom till mottagningstid eller e-posta oss.
1. Konfigurera dina andra verktyg att använda Vim-bindningar (se instruktionerna ovan).
1. Anpassa din `~/.vimrc` vidare och installera fler insticksmoduler.
1. (Avancerad) Konvertera XML till JSON ([exempelfil]({{ '/2020/files/example-data.xml' | relative_url }})) med Vim-makron.
   Försök göra detta själv, men du kan titta i avsnittet [makron](#makron) ovan om du fastnar.
