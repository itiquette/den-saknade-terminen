---
layout: lecture
title: "Kommandoradsmiljön"
presenter: Jose
date: 2019-01-17
order: 1
video:
  aspect: 62.5
  id: i0rf1gpKL1E
---

## Alias och funktioner

Som du kan tänka dig kan det bli tröttsamt att skriva långa kommandon med många flaggor eller utförliga alternativ.
De flesta skal stöder dock **alias**.
I bash har ett alias till exempel följande struktur (notera att det inte finns några mellanslag runt `=`):

```bash
alias alias_name="command_to_alias"
```

<!-- Vi kan lägga vanliga flaggor i alias, t.ex. `alias ll=ls -ltAh`. Alias kan också byggas på varandra. -->

Alias har många praktiska egenskaper.

```bash
# Alias kan samla bra standardflaggor
alias ll="ls -lh"

# Spara mycket knappande för vanliga kommandon
alias gc="git commit"

# Alias kan skriva över befintliga kommandon
alias mv="mv -i"
alias mkdir="mkdir -p"

# Alias kan byggas på varandra
alias la="ls -A"
alias lla="la -l"

# För att ignorera ett alias, inled kommandot med \
\ls
# Eller stäng av aliaset med unalias
unalias la

```
<!--
För att ta bort ett alias kan du köra `unalias alias_name`. Om du tillfälligt vill ignorera ett alias när du kör ett kommando kan du skriva ett omvänt snedstreck före kommandot: `\alias_name`. Det är praktiskt när ett alias skriver över ett befintligt kommando. -->


I många situationer kan alias däremot vara begränsande,
särskilt när du vill skriva kedjade kommandon som tar samma argument.
Ett alternativ är **funktioner**,
som är en mellanväg mellan alias och egna skalskript.

Här är en exempel­funktion som skapar en katalog och går in i den.

```bash
mcd () {
    mkdir -p $1
    cd $1
}
```

Alias och funktioner sparas inte mellan skalsessioner som standard.
För att göra ett alias permanent behöver du lägga till det i någon av skalets uppstartsfiler,
som `.bashrc` eller `.zshrc`.
Mitt förslag är att skriva dem separat i en `.aliases`-fil och sedan `source` den från dina olika skalkonfigurationsfiler.

<!-- Om du aliasar ett verktyg till en "förbättrad" variant, t.ex. `alias bat=cat`, är det bra att känna till att bash kan ignorera alias med `\cat` och ignorera både alias och funktioner med `command cat`. -->

## Skal och ramverk

I föreläsningen om skal och skriptning täckte vi `bash`,
eftersom det är det mest utbredda skalet och standard på de flesta system.
Det är dock inte det enda alternativet.

Till exempel är `zsh` en övermängd av `bash` och erbjuder många praktiska funktioner direkt,
som:

- Smartare globbing, `**`
- Inbyggd globbing/wildcard-expansion
- Stavningskorrigering
- Bättre tab-komplettering/val
- Sökvägsexpansion (`cd /u/lo/b` expanderar till `/usr/local/bin`)

Många skal kan dessutom förbättras med **ramverk**.
Några populära allmänna ramverk är [prezto](https://github.com/sorin-ionescu/prezto) och [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh),
och mindre ramverk fokuserar på specifika funktioner,
till exempel [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) eller [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search).
Andra skal, som [fish](https://fishshell.com/), har många av dessa användarvänliga funktioner som standard.
Några sådana funktioner är:

- Högerprompt
- Syntaxmarkering av kommandon
- Delsträngssökning i historiken
- Flaggkompletteringar baserade på man-sidor
- Smartare autokomplettering
- Prompt-teman

En viktig sak med sådana ramverk är att skalet kan bli långsamt
om koden inte är väloptimerad eller om det blir för mycket kod.
Du kan alltid profilera och stänga av funktioner som du sällan använder,
eller som du värderar lägre än snabbhet.

## Terminalemulatorer och multiplexerare

Utöver att anpassa skalet är det värt att lägga tid på valet av **terminalemulator** och dess inställningar.
Det finns väldigt många terminalemulatorer (här är en [jämförelse](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)).

Eftersom du kan komma att tillbringa hundratals eller tusentals timmar i terminalen,
lönar det sig att utforska inställningarna.
Exempel på saker du kan vilja justera:

- Fontval
- Färgschema
- Kortkommandon
- Stöd för flikar/paneler
- Scrollback-konfiguration
- Prestanda (vissa nyare terminaler som [Alacritty](https://github.com/jwilm/alacritty) erbjuder GPU-acceleration)

Det är också värt att nämna **terminalmultiplexrar** som [tmux](https://github.com/tmux/tmux).
`tmux` låter dig ha flera skalsessioner i paneler och flikar.
Det stöder också attach/detach,
vilket är ett vanligt användningsfall när du arbetar på en fjärrserver och vill hålla skalet igång utan att behöva oroa dig för att `disown` dina aktuella processer
(som standard avslutas processer när du loggar ut).
Med `tmux` kan du därför hoppa in och ut ur komplexa terminal-layouter.
Precis som terminalemulatorer stöder `tmux` omfattande anpassning via filen `~/.tmux.conf`.


## Kommandoradsverktyg

Kommandoradsverktygen som finns som standard i de flesta UNIX-baserade operativsystem
räcker mer än väl till 99 % av det du vanligtvis behöver göra.


I nästa del går jag igenom alternativa verktyg för väldigt vanliga skaloperationer,
som ofta är smidigare att använda.
Vissa av dem tillför ny och förbättrad funktionalitet,
medan andra främst erbjuder enklare och mer intuitiva gränssnitt med bättre standardvärden.

### `fasd` vs `cd`

Även med förbättrad sökvägsexpansion och tab-komplettering kan katalogbyten bli repetitiva.
[Fasd](https://github.com/clvv/fasd) (eller [autojump](https://github.com/wting/autojump)) löser detta
genom att hålla reda på nyligen och ofta använda mappar och göra fuzzy matching.

Om jag till exempel har besökt sökvägen `/home/user/awesome_project/code`,
så kommer `z code` att `cd`:a dit.
Om jag har flera mappar som heter code kan jag avgränsa med `z awe code`,
vilket ger en närmare träff.
Till skillnad från autojump erbjuder fasd också kommandon som,
i stället för att köra `cd`,
bara expanderar ofta/nyligen använda filer, mappar eller båda.


### `bat` vs `cat`

Även om `cat` gör sitt jobb perfekt,
förbättrar [bat](https://github.com/sharkdp/bat) det med syntaxmarkering,
sidvisning,
radnummer
och git-integration.


### `exa`/`ranger` vs `ls`

`ls` är ett bra kommando,
men vissa standardval kan vara irriterande,
till exempel att storlekar visas i råa byte.
[exa](https://github.com/ogham/exa) ger bättre standardvärden.

Om du behöver navigera många mappar och/eller förhandsvisa många filer,
kan [ranger](https://github.com/ranger/ranger) vara mycket effektivare än `cd` och `cat` tack vare sitt gränssnitt.
Det är ganska anpassningsbart,
och med rätt konfiguration kan du till och med [förhandsvisa bilder](https://github.com/ranger/ranger/wiki/Image-Previews) i terminalen.

### `fd` vs `find`

[fd](https://github.com/sharkdp/fd) är ett enkelt,
snabbt
och användarvänligt alternativ till `find`.
`find` har standardbeteenden,
till exempel att du ofta måste ange `--name` (det du vill göra 99 % av gångerna),
som gör det mindre smidigt till vardags.
`fd` är också `git`-medvetet
och hoppar över filer i `.gitignore` och `.git` som standard.
Dessutom har det bra färgkodning direkt.

### `rg/fzf` vs `grep`

`grep` är ett utmärkt verktyg,
men om du vill söka i många filer samtidigt finns bättre alternativ för just det.
[ack](https://github.com/beyondgrep/ack3), [ag](https://github.com/ggreer/the_silver_searcher) och [rg](https://github.com/BurntSushi/ripgrep)
söker rekursivt i aktuell katalog efter ett regex-mönster samtidigt som de respekterar dina gitignore-regler.
Alla fungerar ganska likt,
men jag föredrar `rg` för att det kan söka igenom hela min hemkatalog mycket snabbt.

På samma sätt är det lätt att hamna i att skriva `CMD | grep PATTERN` om och om igen.
[fzf](https://github.com/junegunn/fzf) är en fuzzy finder för kommandoraden
som låter dig filtrera utdata från i princip vilket kommando som helst interaktivt.

### `rsync` vs `cp/scp`

`mv` och `scp` är perfekta i många lägen,
men när du kopierar/flyttar stora mängder filer,
stora enskilda filer,
eller när viss data redan finns på målet,
är `rsync` en stor förbättring.
`rsync` hoppar över filer som redan har överförts,
och med `--partial` kan det återuppta en tidigare avbruten kopiering.

### `trash` vs `rm`

`rm` är ett farligt kommando i den meningen att när du raderar en fil finns ingen enkel väg tillbaka.
Moderna operativsystem beter sig dock inte så i filhanteraren,
utan flyttar i stället filer till papperskorgen,
som töms periodiskt.

Eftersom hanteringen av papperskorg varierar mellan operativsystem finns inget enhetligt CLI-verktyg.
I macOS finns [trash](https://hasseg.org/trash/),
och i Linux finns bland annat [trash-cli](https://github.com/andreafrancia/trash-cli/).

### `mosh` vs `ssh`

`ssh` är ett mycket praktiskt verktyg,
men med långsam anslutning kan fördröjningen bli störande,
och om anslutningen bryts måste du ansluta igen.
[mosh](https://mosh.org/) är ett praktiskt verktyg som tillåter roaming,
stödjer intermittent uppkoppling,
och ger intelligent lokal ekohantering.

### `tldr` vs `man`

Du kan oftast ta reda på vad ett kommando gör och vilka alternativ det har med `man` och flaggan `-h`/`--help`.
I vissa fall kan det dock vara svårt att snabbt hitta rätt i detaljerad dokumentation.

Kommandot [tldr](https://github.com/tldr-pages/tldr) är ett community-drivet dokumentationssystem i kommandoraden,
som ger några enkla, illustrativa exempel på vad kommandot gör och de vanligaste argumenten.


### `aunpack` vs `tar/unzip/unrar`

Som [den här xkcd](https://xkcd.com/1168/) visar,
kan det vara knepigt att komma ihåg alternativ för `tar`,
och ibland behöver du helt andra verktyg,
som `unrar` för .rar-filer.
Paketet [atool](https://www.nongnu.org/atool/) innehåller kommandot `aunpack`,
som listar ut rätt alternativ och alltid lägger det extraherade arkivet i en ny mapp.


## Övningar

1. Kör `cat .bash_history | sort | uniq -c | sort -rn | head -n 10` (eller `cat .zhistory | sort | uniq -c | sort -rn | head -n 10` för zsh)
   för att få dina 10 mest använda kommandon,
   och överväg att skriva kortare alias för dem.
1. Välj en terminalemulator och ta reda på hur du ändrar följande egenskaper:
    - Fontval
    - Färgschema.
      Hur många färger har ett standardschema?
      Varför?
    - Storlek på scrollback-historik

1. Installera `fasd` eller liknande verktyg och skriv en bash/zsh-funktion `v`
   som gör fuzzy matching på argumenten och öppnar bästa träffen i valfri editor.
   Ändra sedan funktionen så att du kan välja med `fzf` när det finns flera träffar.
1. Eftersom `fzf` är bekvämt för fuzzy-sökningar och skalhistorik passar bra för sådana sökningar,
   undersök hur du binder `fzf` till `^R`.
   Du hittar information [här](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings).
1. Vad gör alternativet `--bar` i `ack`?
