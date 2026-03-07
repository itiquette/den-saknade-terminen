---
layout: lecture
title: "Dotfiles"
presenter: Anish
date: 2019-01-24
order: 1
video:
  aspect: 62.5
  id: YSZBWWJw3mI
---

Många program konfigureras med klartextfiler som kallas "dotfiles"
(eftersom filnamnen börjar med `.`, t.ex. `~/.gitconfig`, och därför
döljs i kataloglistningen `ls` som standard).

Många verktyg du använder har sannolikt många inställningar som går att finjustera.
Ofta anpassas verktyg med specialiserade språk,
t.ex. Vimscript för Vim eller skalets eget språk för ett skal.

Att anpassa dina verktyg efter ditt föredragna arbetssätt gör dig mer produktiv.
Vi rekommenderar att du lägger tid på att anpassa verktygen själv
snarare än att klona någon annans dotfiles från GitHub.

Du har troligen redan några dotfiles på plats.
Några ställen att titta på:

- `~/.bashrc`
- `~/.emacs`
- `~/.vim`
- `~/.gitconfig`

Vissa program lägger inte filerna direkt i hemkatalogen utan i en undermapp till `~/.config`.

Dotfiles är inte exklusivt för kommandoradsprogram.
Till exempel kan videospelaren [MPV](https://mpv.io/) konfigureras genom att redigera filer under `~/.config/mpv`.

# Lär dig anpassa verktyg

Du kan lära dig verktygens inställningar genom att läsa dokumentation på nätet eller
[man-sidor](https://en.wikipedia.org/wiki/Man_page).
Ett annat bra sätt är att söka efter blogginlägg om specifika program,
där författare beskriver sina favoritinställningar.
Ytterligare ett sätt att lära sig anpassningar är att titta igenom andras dotfiles.
Du hittar massor av
[dotfiles-kodförråd](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)
på GitHub.
Se det mest populära
[här](https://github.com/mathiasbynens/dotfiles) (vi rekommenderar dock att du inte kopierar konfiguration blint).

# Organisering

Hur bör du organisera dina dotfiles?
De bör ligga i en egen mapp,
under versionshantering,
och **symboliskt länkas** på plats med ett skript.
Det ger följande fördelar:

- **Enkel installation**: om du loggar in på en ny maskin tar det bara en minut att lägga in dina anpassningar
- **Portabilitet**: dina verktyg fungerar likadant överallt
- **Synkronisering**: du kan uppdatera dina dotfiles var som helst och hålla dem synkroniserade
- **Ändringshistorik**: du kommer sannolikt underhålla dina dotfiles under hela din programmeringskarriär, och versionshistorik är värdefullt i långlivade projekt

```shell
cd ~/src
mkdir dotfiles
cd dotfiles
git init
touch bashrc
# skapa en bashrc-fil med några inställningar, t.ex.:
#     PS1='\w > '
touch install
chmod +x install
# lägg in följande i installationsskriptet:
#     #!/usr/bin/env bash
#     BASEDIR=$(dirname $0)
#     cd $BASEDIR
#
#     ln -s ${PWD}/bashrc ~/.bashrc
git add bashrc install
git commit -m 'Initial commit'
```

# Avancerade ämnen

## Maskinspecifika anpassningar

För det mesta vill du ha samma konfiguration på alla maskiner,
men ibland vill du ha en liten avvikelse på en specifik maskin.
Här är ett par sätt att hantera det:

### En gren per maskin

Använd versionshantering för att underhålla en gren per maskin.
Det här angreppssättet är logiskt rakt på sak men kan bli ganska tungrott.

### If-satser

Om konfigurationsfilen stödjer det, använd motsvarigheten till if-satser för att
tillämpa maskinspecifika anpassningar.
Till exempel kan ditt skal ha något i stil med:

```shell
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Darwin är arkitekturnamnet för macOS-system
if [[ "$(uname)" == "Darwin" ]]; then {do_something}; fi

# Du kan också göra det maskinspecifikt
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

### Inkluderingar

Om konfigurationsfilen stödjer det, använd inkluderingar.
Till exempel kan en `~/.gitconfig` ha en inställning:

```
[include]
    path = ~/.gitconfig_local
```

Sedan kan `~/.gitconfig_local` på varje maskin innehålla maskinspecifika inställningar.
Du kan till och med versionshantera dessa i ett separat kodförråd för just maskinspecifika inställningar.

Den här idén är också användbar om du vill att olika program ska dela viss konfiguration.
Om du till exempel vill att både `bash` och `zsh` ska dela samma aliasuppsättning kan du skriva dem i `.aliases` och ha följande block i båda.

```bash
# Testa om ~/.aliases finns och läs in den
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

# Resurser

- Lärarnas dotfiles:
  [Anish](https://github.com/anishathalye/dotfiles),
  [Jon](https://github.com/jonhoo/configs),
  [Jose](https://github.com/jjgo/dotfiles)
- [GitHub does dotfiles](https://dotfiles.github.io/) (om dotfiles): dotfile-ramverk,
verktyg, exempel och guider
- [Shell startup
  scripts](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html): en
  förklaring av olika konfigurationsfiler som används av ditt skal

# Övningar

1. Skapa en mapp för dina dotfiles och sätt upp [versionshantering]({{ '/2019/version-control/' | relative_url }}).

1. Lägg till konfiguration för minst ett program, till exempel ditt skal, med någon anpassning (för att komma igång kan det vara så enkelt som att ändra prompten genom att sätta `$PS1`).

1. Sätt upp ett sätt att installera dina dotfiles snabbt (och utan manuellt arbete) på en ny maskin.
   Det kan vara så enkelt som ett skalskript som kör `ln -s` för varje fil,
   eller ett [specialiserat verktyg](https://dotfiles.github.io/utilities/).

1. Testa ditt installationsskript i en ny virtuell maskin.

1. Migrera alla dina nuvarande verktygskonfigurationer till ditt dotfiles-kodförråd.

1. Publicera dina dotfiles på GitHub.
