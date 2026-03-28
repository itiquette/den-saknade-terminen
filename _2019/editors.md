---
layout: lecture
title: "Redigerare"
presenter: Anish
date: 2019-01-22
order: 1
video:
  aspect: 62.5
  id: 1vLcusYSrI4
---

# Varför redigerare är viktiga

Som programmerare tillbringar vi större delen av tiden med att redigera vanliga textfiler.
Det är värt att investera tid i att lära sig en redigerare som passar ens behov.

Hur lär man sig en ny redigerare?
Man tvingar sig själv att använda den under en period, även om produktiviteten tillfälligt sjunker.
Det lönar sig snart.
Två veckor räcker för att lära sig grunderna.

Vi kommer att lära ut Vim, men vi uppmuntrar dig att testa andra redigerare.
Det är ett högst personligt val, och folk har [starka åsikter](https://en.wikipedia.org/wiki/Editor_war).

Vi kan inte lära ut en kraftfull redigerare på 50 minuter, så vi fokuserar på grunderna, visar mer avancerad funktionalitet, och ger resurser för att bemästra verktyget.
Vi undervisar i Vim-kontext, men de flesta idéer går att överföra till andra kraftfulla redigerare.
Om de inte gör det bör du kanske inte använda den redigeraren.

![Inlärningskurvor för textredigerare]({{ '/2019/files/editor-learning-curves.jpg' | relative_url }})

<!-- source: https://blogs.msdn.microsoft.com/steverowe/2004/11/17/code-editor-learning-curves/ -->

Grafen över inlärningskurvor för redigerare är en myt.
Att lära sig grunderna i en kraftfull redigerare är ganska enkelt, även om det kan ta år att bemästra allt.

Vilka redigerare är populära i dag?
Se den här [Stack Overflow-undersökningen](https://insights.stackoverflow.com/survey/2018/#development-environments-and-tools) (det kan finnas viss bias eftersom Stack Overflow-användare inte nödvändigtvis representerar programmerare i stort).

## Kommandoradsredigerare

Även om du i slutändan väljer en GUI-redigerare, är det värt att kunna en kommandoradsredigerare för att enkelt redigera filer på fjärrmaskiner.

# Nano

Nano är en enkel kommandoradsredigerare.

- Flytta med piltangenterna
- Alla andra kortkommandon (spara, avsluta) visas längst ner

# Vim

Vi/Vim är en kraftfull textredigerare.
Det är ett kommandoradsprogram som vanligtvis finns installerat överallt, vilket gör det praktiskt för redigering på fjärrmaskiner.

Vim har också grafiska versioner, som GVim och [MacVim](https://macvim-dev.github.io/macvim/).
De erbjuder extra funktioner, som 24-bitarsfärg, menyer och popup-fönster.

## Vims filosofi

- När man programmerar lägger man mer tid på att läsa/redigera än på att skriva
    - Vim är en **modal** redigerare: olika lägen för att infoga text respektive manipulera text
- Vim är programmerbar (med Vimscript och även språk som Python)
- Vims gränssnitt i sig fungerar som ett programmeringsspråk
    - Tangenttryckningar (med minnesvänliga namn) är kommandon
    - Kommandon är komponerbara
- Använd inte musen: för långsamt
- Redigeraren ska fungera i samma hastighet som du tänker

## Introduktion till Vim

### Lägen

Vim visar aktuellt läge längst ner till vänster.

- Normalläge: för att röra dig i en fil och redigera
    - Tillbringa mesta tiden här
- Infogningsläge: för att skriva in text
- Visuellt läge (tecken-, rad- eller blockläge): för att markera textblock

Du byter läge genom att trycka `<ESC>` för att gå tillbaka till normalläge från vilket läge som helst. Från normalläge går du till infogningsläge med `i`, visuellt läge med `v`, visuellt radläge med `V`, och visuellt blockläge med `<C-v>`.

Du använder `<ESC>` mycket i Vim, så överväg att mappa om Caps Lock till Escape.

### Grunder

Vim ex-kommandon körs via `:{command}` i normalläge.

- `:q` avsluta (stäng fönster)
- `:w` spara
- `:wq` spara och avsluta
- `:e {name of file}` öppna fil för redigering
- `:ls` visa öppna buffrar
- `:help {topic}` öppna hjälp
    - `:help :w` öppnar hjälp för ex-kommandot `:w`
    - `:help w` öppnar hjälp för rörelsen `w`

### Förflyttning

Vim handlar om effektiv förflyttning.
Navigera i filen i normalläge.

- Inaktivera piltangenterna för att undvika dåliga vanor
```vim
nnoremap <Left> :echoe "Use h"<CR>
nnoremap <Right> :echoe "Use l"<CR>
nnoremap <Up> :echoe "Use k"<CR>
nnoremap <Down> :echoe "Use j"<CR>
```
- Grundrörelser: `hjkl` (vänster, ner, upp, höger)
- Ord: `w` (nästa ord), `b` (början av ord), `e` (slutet av ord)
- Rader: `0` (början av rad), `^` (första icke-blanktecken), `$` (slutet av rad)
- Skärm: `H` (överst), `M` (mitten), `L` (nederst)
- Fil: `gg` (början av fil), `G` (slutet av fil)
- Radnummer: `:{number}<CR>` eller `{number}G` (rad {number})
- Övrigt: `%` (matchande objekt)
- Sök i rad: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - hitta/till framåt/bakåt {character} på aktuell rad
- Upprepa N gånger: `{number}{movement}`, t.ex. `10j` går ner 10 rader
- Sök: `/{regex}`, `n` / `N` för att navigera träffar

### Markering

Visuella lägen:

- Visual
- Visual Line
- Visual Block

Du kan använda förflyttningskommandon för att göra markeringar.

### Manipulera text

Allt du tidigare gjorde med musen gör du nu med tangentbordet (och kraftfulla, komponerbara kommandon).

- `i` gå till infogningsläge
    - men för manipulation/radering vill du använda mer än backsteg
- `o` / `O` infoga rad under / över
- `d{motion}` radera {motion}
    - t.ex. `dw` raderar ord, `d$` raderar till radslut, `d0` raderar till radbörjan
- `c{motion}` ändra {motion}
    - t.ex. `cw` ändrar ord
    - motsvarar ungefär `d{motion}` följt av `i`
- `x` radera tecken (motsvarar `dl`)
- `s` ersätt tecken (motsvarar `xi`)
- visuellt läge + manipulation
    - markera text, `d` för att radera eller `c` för att ändra
- `u` för ångra, `<C-r>` för gör om
- Mycket mer att lära: t.ex. `~` växlar versalisering på ett tecken

### Resurser

- `vimtutor` är ett kommandoradsprogram som lär dig vim
- [Vim Adventures](https://vim-adventures.com/) är ett spel för att lära sig Vim

## Anpassa Vim

Vim anpassas via en textbaserad konfigurationsfil i `~/.vimrc` (som innehåller Vimscript-kommandon).
Det finns troligen många grundinställningar du vill slå på.

Titta på andras dotfiles på GitHub för inspiration, men undvik att kopiera hela konfigurationer rakt av.
Läs, förstå, och ta det du behöver.

Några anpassningar att överväga:

- Syntaxmarkering: `syntax on`
- Färgscheman
- Radnummer: `set nu` / `set rnu`
- Backsteg genom allt: `set backspace=indent,eol,start`

## Avancerad Vim

Här är några exempel som visar redigerarens kraft.
Vi kan inte lära ut alla sådana tekniker här, men du lär dig dem över tid.
En bra tumregel är: när du tänker "det måste finnas ett bättre sätt att göra det här", så finns det ofta det.
Sök upp det.

### Sök och ersätt

Kommandot `:s` (substitute) ([dokumentation](https://vim.fandom.com/wiki/Search_and_replace)).

- `%s/foo/bar/g`
    - ersätt foo med bar globalt i filen
- `%s/\[.*\](\(.*\))/\1/g`
    - ersätt namngivna Markdown-länkar med rena URL:er

### Flera fönster

- `sp` / `vsp` för att dela fönster
- Du kan ha flera vyer av samma buffert.

### Musstöd

- `set mouse+=a`
    - du kan klicka, skrolla och markera

### Makron

- `q{character}` för att börja spela in ett makro i register `{character}`
- `q` för att stoppa inspelning
- `@{character}` spelar upp makrot
- Makrokörning stoppar vid fel
- `{number}@{character}` kör makrot {number} gånger
- Makron kan vara rekursiva
    - rensa först makrot med `q{character}q`
    - spela in makrot och använd `@{character}` för att anropa makrot rekursivt
  (det gör inget förrän inspelningen är klar)
- Exempel: konvertera xml till json ([fil]({{ '/2019/files/example-data.xml' | relative_url }}))
    - Array av objekt med nycklarna "name" / "email"
    - Använda ett Python-program?
    - Använda sed / regex
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - ...
    - Vim-kommandon / makron
        - `Gdd`, `ggdd` raderar första och sista raden
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

## Bygg ut Vim

Det finns massor av insticksmoduler för att bygga ut Vim.

Börja med en hanterare för insticksmoduler som [vim-plug](https://github.com/junegunn/vim-plug), [Vundle](https://github.com/VundleVim/Vundle.vim), eller [pathogen.vim](https://github.com/tpope/vim-pathogen).

Några insticksmoduler att överväga:

- [ctrlp.vim](https://github.com/kien/ctrlp.vim): ungefärlig filsökare
- [vim-fugitive](https://github.com/tpope/vim-fugitive): git-integration
- [vim-surround](https://github.com/tpope/vim-surround): manipulera "omgivning"-tecken
- [gundo.vim](https://github.com/sjl/gundo.vim): navigera i ångra-trädet
- [nerdtree](https://github.com/scrooloose/nerdtree): filutforskare
- [syntastic](https://github.com/vim-syntastic/syntastic): syntaxkontroll
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): smarta rörelser
- [vim-over](https://github.com/osyo-manga/vim-over): förhandsvisning av ersättning

Listor med insticksmoduler:

- [Vim Awesome](https://vimawesome.com/)

## Vim-läge i andra program

För många populära redigerare (t.ex. vim och emacs) finns emuleringsstöd i andra verktyg.

- Skal
    - bash: `set -o vi`
    - zsh: `bindkey -v`
    - `export EDITOR=vim` (miljövariabel som används av program som `git`)
- `~/.inputrc`
    - `set editing-mode vi`

Det finns även tillägg med vim-tangentbindningar för webbläsare.
Några populära är [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) för Google Chrome och [Tridactyl](https://github.com/tridactyl/tridactyl) för Firefox.


## Resurser

- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2018/): olika Vim-tips
- [Neovim](https://neovim.io/) är en modern omimplementation av Vim med mer aktiv utveckling.
- [Vim Golf](https://www.vimgolf.com/): olika Vim-utmaningar

{% comment %}
# Resurser

TODO: resurser för andra redigerare?
{% endcomment %}

# Övningar

1. Experimentera med några redigerare.
   Prova minst en kommandoradsredigerare (t.ex. Vim) och minst en GUI-redigerare (t.ex. Atom).
   Lär via handledningar som `vimtutor` (eller motsvarande för andra redigerare).
   För att verkligen få känsla för en ny redigerare, använd den exklusivt i ett par dagar i ditt vanliga arbete.

1. Anpassa din redigerare.
   Titta på tips och tricks på nätet, och gå igenom andras konfigurationer (de är ofta väldokumenterade).

1. Experimentera med insticksmoduler för din redigerare.

1. Bestäm dig för att använda en kraftfull redigerare i minst ett par veckor.
   Då bör du börja se fördelarna.
   Vid någon punkt bör redigeraren kunna arbeta i samma hastighet som du tänker.

1. Installera en linter (t.ex. pyflakes för python), koppla den till din redigerare, och testa att den fungerar.
