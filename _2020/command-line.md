---
layout: lecture
title: "Kommandoradsmiljön"
description: >
  Lär dig om jobbstyrning, terminalmultiplexrar, dotfiles och fjärrmaskiner med SSH.
thumbnail: /static/assets/thumbnails/2020/lec5.png
date: 2020-01-21
ready: true
video:
  aspect: 56.25
  id: e8BO_dYxk5c
---

I föreläsningen går vi igenom flera sätt att förbättra ditt arbetssätt när du använder skalet.
Vi har arbetat med skalet ett tag nu, men främst fokuserat på att köra olika kommandon.
Nu ska vi se hur man kör flera processer samtidigt och ändå håller ordning på dem, hur man stoppar eller pausar en viss process och hur man låter en process köra i bakgrunden.

Vi ska också lära oss olika sätt att förbättra skalet och andra verktyg genom att definiera alias och konfigurera dem med dotfiles.
Båda delarna kan spara tid, till exempel genom att använda samma konfigurationer på alla maskiner utan att skriva långa kommandon varje gång.
Vi tittar också på hur man arbetar med fjärrmaskiner via SSH.


# Jobbstyrning

I vissa fall behöver du avbryta ett jobb medan det körs, till exempel om ett kommando tar för lång tid att bli klart (som en `find` över en stor katalogstruktur).
Oftast räcker det att trycka `Ctrl-C` så stoppas kommandot.
Men hur fungerar det egentligen, och varför misslyckas det ibland att stoppa processen?

## Döda en process

Ditt skal använder en UNIX-mekanism för kommunikation som kallas _signal_ för att skicka information till processen.
När en process får en signal avbryter den körningen, hanterar signalen och kan ändra körflödet utifrån informationen i signalen.
Därför är signaler _programvaruavbrott_.

I vårt fall innebär `Ctrl-C` att skalet skickar signalen `SIGINT` till processen.

Här är ett minimalt exempel på ett Python-program som fångar `SIGINT` och ignorerar den, så att det inte längre stannar.
För att döda programmet kan vi i stället använda signalen `SIGQUIT` genom att trycka `Ctrl-\`.

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nJag fick en SIGINT, men jag tänker inte sluta")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

Här är vad som händer om vi skickar `SIGINT` två gånger till programmet, följt av `SIGQUIT`.
Observera att `^` är hur `Ctrl` visas när det skrivs i terminalen.

```
$ python sigint.py
24^C
Jag fick en SIGINT, men jag tänker inte sluta
26^C
Jag fick en SIGINT, men jag tänker inte sluta
30^\[1]    39913 quit       python sigint.py
```

Även om `SIGINT` och `SIGQUIT` båda vanligtvis är kopplade till terminalrelaterade begäranden är en mer allmän signal för att be en process avsluta snyggt `SIGTERM`.
För att skicka den kan vi använda kommandot [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html), med syntaxen `kill -TERM <PID>`.

## Pausa och bakgrundsköra processer

Signaler kan göra annat än att döda en process.
Till exempel pausar `SIGSTOP` en process.
I terminalen gör `Ctrl-Z` att skalet skickar signalen `SIGTSTP`, kort för Terminal Stop (dvs. terminalens variant av `SIGSTOP`).

Vi kan sedan fortsätta det pausade jobbet i förgrunden eller i bakgrunden med [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) respektive [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html).

Kommandot [`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) listar de ofärdiga jobben kopplade till den aktuella terminalsessionen.
Du kan referera till jobben med deras PID (du kan använda [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) för att ta reda på den).
Mer intuitivt kan du också referera till en process med procenttecken följt av jobbnumret (som visas av `jobs`).
För att referera till det senast bakgrundskörda jobbet kan du använda den särskilda parametern `$!`.

Ytterligare en sak att känna till är att suffixet `&` i ett kommando kör kommandot i bakgrunden och ger tillbaka prompten, men kommandot använder fortfarande skalets STDOUT vilket kan vara störande (använd skalomdirigeringar i så fall).

För att bakgrundsköra ett redan körande program kan du trycka `Ctrl-Z` följt av `bg`.
Observera att bakgrundsprocesser fortfarande är barnprocesser till terminalen och dör om du stänger terminalen (då skickas ännu en signal, `SIGHUP`).
För att förhindra det kan du köra programmet med [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (ett omslag som ignorerar `SIGHUP`), eller använda `disown` om processen redan har startats.
Alternativt kan du använda en terminalmultiplexer, som vi ser i nästa avsnitt.

Nedan är en exempelsession som visar några av dessa begrepp.

```
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs

```

En särskild signal är `SIGKILL`, eftersom processen inte kan fånga den och den alltid dödar processen omedelbart.
Den kan dock ge otrevliga bieffekter, till exempel att efterlämna föräldralösa barnprocesser.

Du kan läsa mer om dessa och andra signaler [här](https://en.wikipedia.org/wiki/Signal_(IPC)) eller genom att skriva [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) eller `kill -l`.


# Terminalmultiplexrar

När du använder kommandoraden vill du ofta köra mer än en sak samtidigt.
Du kanske till exempel vill ha redigeraren och programmet igång sida vid sida.
Det går att lösa genom att öppna nya terminalfönster, men en terminalmultiplexer är en mer flexibel lösning.

Terminalmultiplexrar som [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) låter dig dela upp terminalfönster i paneler och flikar så att du kan interagera med flera skalsessioner.
Dessutom låter terminalmultiplexrar dig koppla loss en aktiv terminalsession och återansluta senare.
Det kan göra arbetsflödet betydligt bättre när du arbetar med fjärrmaskiner eftersom du slipper `nohup` och liknande knep.

Den mest populära terminalmultiplexern i dag är [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html).
`tmux` är i hög grad konfigurerbar, och med tillhörande tangentbindningar kan du skapa flera flikar och paneler och snabbt navigera mellan dem.

`tmux` förutsätter att du kan dess tangentbindningar, och de har formen `<C-b> x` vilket betyder (1) tryck `Ctrl+b`, (2) släpp `Ctrl+b`, och (3) tryck `x`.
`tmux` har följande hierarki av objekt:
- **Sessioner** - en session är en oberoende arbetsyta med ett eller flera fönster
    + `tmux` startar en ny session.
    + `tmux new -s NAME` startar den med det namnet.
    + `tmux ls` listar aktuella sessioner
    + Inuti `tmux` kopplar `<C-b> d` loss den aktuella sessionen
    + `tmux a` ansluter till senaste sessionen.
      Du kan använda flaggan `-t` för att ange vilken

- **Fönster** - motsvarar flikar i redigerare eller webbläsare; visuellt separata delar av samma session
    + `<C-b> c` skapar ett nytt fönster.
      För att stänga det kan du helt enkelt avsluta skalet med `<C-d>`
    + `<C-b> N` går till fönster nummer _N_.
      Observera att de är numrerade
    + `<C-b> p` går till föregående fönster
    + `<C-b> n` går till nästa fönster
    + `<C-b> ,` byter namn på aktuellt fönster
    + `<C-b> w` listar aktuella fönster

- **Paneler** - likt splits i vim låter paneler dig ha flera skal i samma vy.
    + `<C-b> "` delar aktuell panel horisontellt
    + `<C-b> %` delar aktuell panel vertikalt
    + `<C-b> <direction>` flyttar till panelen i angiven riktning (piltangenter).
    + `<C-b> z` växlar zoom för aktuell panel
    + `<C-b> [` startar scrollback.
      Du kan sedan trycka `<space>` för att starta en markering och `<enter>` för att kopiera markeringen.
    + `<C-b> <space>` växlar mellan panelarrangemang.

För vidare läsning finns [här](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) en snabb introduktion till `tmux`, och [här](https://linuxcommand.org/lc3_adv_termmux.php) en mer detaljerad förklaring som även täcker det ursprungliga kommandot `screen`.
Du kan också vilja bekanta dig med [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html), eftersom det är installerat i de flesta UNIX-system.

# Alias

Det kan bli tröttsamt att skriva långa kommandon med många flaggor eller utförliga alternativ.
Därför stöder de flesta skal _alias_.
Ett skalalias är en kortform för ett annat kommando som skalet ersätter automatiskt.
Ett alias i bash har till exempel följande struktur:

```bash
alias alias_name="command_to_alias arg1 arg2"
```

Observera att det inte får finnas några blanksteg runt likhetstecknet `=`, eftersom [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) är ett skalkommando som tar ett enda argument.

Alias har många praktiska användningsområden:

```bash
# Skapa kortformer för vanliga flaggor
alias ll="ls -lh"

# Spara mycket skrivande för vanliga kommandon
alias gs="git status"
alias gc="git commit"
alias v="vim"

# Hjälp dig undvika felstavningar
alias sl=ls

# Skriv över befintliga kommandon för bättre standardvärden
alias mv="mv -i"           # -i frågar före överskrivning
alias mkdir="mkdir -p"     # -p skapar föräldrakataloger vid behov
alias df="df -h"           # -h skriver ut i läsbart format

# Alias kan byggas på varandra
alias la="ls -A"
alias lla="la -l"

# För att ignorera ett alias, kör kommandot med \ först
\ls
# Eller stäng av aliaset helt med unalias
unalias la

# För att visa aliasdefinitionen, anropa det med alias
alias ll
# Skriver ut ll='ls -lh'
```

Observera att alias inte är beständiga mellan skalsessioner som standard.
För att göra ett alias beständigt behöver du lägga det i skalets uppstartsfil, som `.bashrc` eller `.zshrc`, vilket vi introducerar i nästa avsnitt.


# Dotfiles

Många program konfigureras med vanlig text i filer som kallas _dotfiles_ (eftersom filnamnen börjar med `.`, t.ex. `~/.vimrc`, så att de är dolda i kataloglistningen `ls` som standard).

Skal är ett exempel på program som konfigureras med sådana filer.
Vid uppstart läser skalet många filer för att ladda sin konfiguration.
Beroende på skal, och om du startar en inloggningssession och/eller en interaktiv session, kan hela processen vara ganska komplex. [Här](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) finns en utmärkt resurs om ämnet.

För `bash` fungerar det i de flesta system att redigera `.bashrc` eller `.bash_profile`.
Här kan du lägga in kommandon du vill köra vid uppstart, som aliasen vi just beskrev eller ändringar av miljövariabeln `PATH`.
Faktum är att många program ber dig lägga till en rad som `export PATH="$PATH:/path/to/program/bin"` i din skalkonfigurationsfil så att deras binärer kan hittas.

Några andra verktyg som kan konfigureras via dotfiles är:

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` och katalogen `~/.vim`
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

Hur bör du organisera dina dotfiles?
De bör ligga i en egen katalog, vara versionshanterade och **symboliskt länkas** på plats med ett skript.
Det ger fördelar som:

- **Enkel installation**: om du loggar in på en ny maskin tar det bara en minut att få in dina anpassningar.
- **Portabilitet**: dina verktyg fungerar likadant överallt.
- **Synkronisering**: du kan uppdatera dotfiles var som helst och hålla dem synkroniserade.
- **Ändringsspårning**: du kommer sannolikt att underhålla dina dotfiles under hela din programmeringskarriär, och versionshistorik är värdefull för långlivade projekt.

Vad ska du lägga i dina dotfiles?
Du kan lära dig verktygets inställningar genom att läsa dokumentation på nätet eller [manualsidor](https://en.wikipedia.org/wiki/Man_page).
Ett annat bra sätt är att söka efter blogginlägg om specifika program där författare berättar om sina favoritinställningar.
Ytterligare ett sätt är att titta i andras dotfiles: det finns mängder av [dotfiles-kodförråd](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) på GitHub --- se det mest populära [här](https://github.com/mathiasbynens/dotfiles) (vi rekommenderar dock att du inte kopierar konfigurationer blint).
[Här](https://dotfiles.github.io/) finns ännu en bra resurs om ämnet.

Alla kursens lärare har sina dotfiles publikt tillgängliga på GitHub: [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles).


## Portabilitet

Ett vanligt problem med dotfiles är att konfigurationerna kanske inte fungerar när du arbetar på flera maskiner, t.ex. om de har olika operativsystem eller skal.
Ibland vill du också att viss konfiguration bara ska gälla på en viss maskin.

Det finns några knep som gör detta enklare.
Om konfigurationsfilen stöder det, använd motsvarigheten till if-satser för att tillämpa maskinspecifika anpassningar.
Till exempel kan ditt skal innehålla något i stil med:

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Kontrollera innan du använder skalspecifika funktioner
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# Du kan också göra det maskinspecifikt
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

Om konfigurationsfilen stöder det, använd includes.
Till exempel kan `~/.gitconfig` ha inställningen:

```
[include]
    path = ~/.gitconfig_local
```

Och på varje maskin kan `~/.gitconfig_local` innehålla maskinspecifika inställningar.
Du kan till och med versionshantera dem i ett separat kodförråd för maskinspecifika inställningar.

Samma idé är användbar om du vill att olika program ska dela viss konfiguration.
Till exempel, om du vill att både `bash` och `zsh` ska dela samma uppsättning alias kan du skriva dem i `.aliases` och ha följande block i båda:

```bash
# Testa om ~/.aliases finns och läs in den
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

<span id="remote-machines"></span>
# Fjärrmaskiner

Det har blivit allt vanligare att programmerare använder fjärrservrar i det dagliga arbetet.
Om du behöver fjärrservrar för att driftsätta backend-programvara, eller behöver en server med högre beräkningskapacitet, kommer du att använda Secure Shell (SSH).
Som de flesta verktyg vi tar upp är SSH i hög grad konfigurerbart, så det är värt att lära sig.

För att logga in med `ssh` på en server kör du ett kommando enligt följande:

```bash
ssh foo@bar.mit.edu
```

Här försöker vi logga in via ssh som användaren `foo` på servern `bar.mit.edu`.
Servern kan anges med en URL (som `bar.mit.edu`) eller en IP-adress (något som `foobar@192.168.1.42`).
Senare ser vi att om vi ändrar ssh-konfigurationsfilen kan du ansluta med något i stil med `ssh bar`.

## Köra kommandon

En ofta förbisedd funktion i `ssh` är möjligheten att köra kommandon direkt.
`ssh foobar@server ls` kör `ls` i hemkatalogen för foobar.
Det fungerar med rör, så `ssh foobar@server ls | grep PATTERN` kör `grep` lokalt på fjärrutdata från `ls`, och `ls | ssh foobar@server grep PATTERN` kör `grep` på servern på lokal utdata från `ls`.


## SSH-nycklar

Nyckelbaserad autentisering använder publik nyckelkryptografi för att bevisa för servern att klienten äger den hemliga privata nyckeln utan att avslöja nyckeln.
På så sätt behöver du inte skriva lösenord varje gång.
Den privata nyckeln (ofta `~/.ssh/id_rsa` och på senare tid `~/.ssh/id_ed25519`) är i praktiken ditt lösenord, så hantera den därefter.

### Generera nycklar

För att generera ett nyckelpar kan du köra [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html).
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```
Du bör välja en lösenfras för att förhindra att någon som får tag i din privata nyckel kommer åt auktoriserade servrar.
Använd [`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) eller [`gpg-agent`](https://linux.die.net/man/1/gpg-agent) så att du inte behöver skriva lösenfrasen varje gång.

Om du någon gång har konfigurerat push till GitHub med SSH-nycklar har du sannolikt redan gjort stegen som beskrivs [här](https://help.github.com/articles/connecting-to-github-with-ssh/) och har ett giltigt nyckelpar.
För att kontrollera om du har en lösenfras och verifiera den kan du köra `ssh-keygen -y -f /path/to/key`.

### Nyckelbaserad autentisering

`ssh` tittar i `.ssh/authorized_keys` för att avgöra vilka klienter som ska släppas in.
För att kopiera över en publik nyckel kan du använda:

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

En enklare lösning där det finns stöd är `ssh-copy-id`:

```bash
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

## Kopiera filer över SSH

Det finns många sätt att kopiera filer över ssh:

- `ssh+tee`, det enklaste är att använda kommandokörning via `ssh` och indata från STDIN med `cat localfile | ssh remote_server tee serverfile`.
  Kom ihåg att [`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) skriver utdata från STDIN till en fil.
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) när du kopierar stora mängder filer/kataloger är kommandot secure copy `scp` smidigare eftersom det enkelt kan gå rekursivt över sökvägar.
  Syntaxen är `scp path/to/local_file remote_host:path/to/remote_file`
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) förbättrar `scp` genom att upptäcka identiska filer lokalt och på fjärrsidan och undvika att kopiera dem igen.
  Det ger också mer finmaskig kontroll över symlänkar, rättigheter och har extra funktioner som flaggan `--partial`, som kan återuppta en tidigare avbruten kopiering.
  `rsync` har liknande syntax som `scp`.

## Portvidarebefordran

I många scenarier stöter du på program som lyssnar på specifika portar på en maskin.
När det sker på din lokala maskin kan du skriva `localhost:PORT` eller `127.0.0.1:PORT`, men vad gör du med en fjärrserver som inte exponerar sina portar direkt via nätverket/internet?

Det kallas portvidarebefordran (_port forwarding_) och finns i två varianter: lokal portvidarebefordran (_local port forwarding_) och fjärrportvidarebefordran (_remote port forwarding_) (se bilderna för mer detaljer; bildkredit från [detta StackOverflow-inlägg](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)).

**Lokal portvidarebefordran** ![Lokal portvidarebefordran]({{ '/static/media/images/local-port-forwarding.png' | relative_url }})

**Fjärrportvidarebefordran** ![Fjärrportvidarebefordran]({{ '/static/media/images/remote-port-forwarding.png' | relative_url }})

Det vanligaste scenariot är lokal portvidarebefordran, där en tjänst på fjärrmaskinen lyssnar på en port och du vill koppla en port på din lokala maskin till den fjärrporten.
Om vi till exempel kör `jupyter notebook` på fjärrservern och den lyssnar på port `8888`, kan vi vidarebefordra den till lokal port `9999` med `ssh -L 9999:localhost:8888 foobar@remote_server` och sedan öppna `localhost:9999` på vår lokala maskin.


## SSH-konfiguration

Vi har gått igenom många argument som kan skickas till `ssh`.
Ett lockande alternativ är att skapa skalalias som ser ut så här:
```bash
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server"
```

Det finns dock ett bättre alternativ: `~/.ssh/config`.

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Konfigurationen kan också använda jokertecken
Host *.mit.edu
    User foobaz
```

En ytterligare fördel med `~/.ssh/config` jämfört med alias är att andra program som `scp`, `rsync`, `mosh`, &c också kan läsa filen och omvandla inställningarna till motsvarande flaggor.


Observera att `~/.ssh/config` kan ses som en dotfile, och i allmänhet är det helt okej att ha den tillsammans med resten av dina dotfiles.
Men om du gör den publik bör du tänka på vilken information du potentiellt ger främlingar på internet: adress till servrar, användare, öppna portar, &c.
Det kan underlätta vissa typer av attacker, så var eftertänksam med att dela din SSH-konfiguration.

Konfiguration på serversidan anges vanligtvis i `/etc/ssh/sshd_config`.
Här kan du göra ändringar som att stänga av lösenordsautentisering, ändra ssh-portar, aktivera X11 forwarding, &c.
Du kan ange konfigurationsinställningar per användare.

## Övrigt

Ett vanligt problem vid anslutning till fjärrserver är avbrott när datorn stängs av, går i vila eller byter nätverk.
Dessutom kan ssh bli frustrerande om anslutningen har märkbar fördröjning.
[Mosh](https://mosh.org/), mobile shell, förbättrar ssh genom att tillåta roaming-anslutningar, ojämn uppkoppling och intelligent lokalt eko.

Ibland är det praktiskt att montera en fjärrkatalog.
[sshfs](https://github.com/libfuse/sshfs) kan montera en katalog på en fjärrserver lokalt, så att du kan använda en lokal redigerare.


# Skal och ramverk

I avsnitten om skalverktyg och skriptning använde vi skalet `bash` eftersom det är överlägset mest utbrett och standardval på de flesta system.
Det är dock inte det enda alternativet.

Till exempel är skalet `zsh` en övermängd av `bash` och erbjuder många praktiska funktioner direkt:

- Smartare mönstermatchning, `**`
- Inline-expansion av mönstermatchning/jokertecken
- Stavningskorrigering
- Bättre tab completion/selection
- Sökvägsexpansion (`cd /u/lo/b` expanderar till `/usr/local/bin`)

**Ramverk** kan också förbättra ditt skal.
Några populära generella ramverk är [prezto](https://github.com/sorin-ionescu/prezto) och [oh-my-zsh](https://ohmyz.sh/), samt mindre ramverk som fokuserar på specifika funktioner, till exempel [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) eller [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search).
Skal som [fish](https://fishshell.com/) har många av dessa användarvänliga funktioner som standard.
Några sådana funktioner är:

- Högerprompt
- Syntaxmarkering av kommandon
- Delsträngssökning i historik
- Flaggkomplettering baserad på manpages
- Smartare autokomplettering
- Promptteman

En sak att notera när du använder sådana ramverk är att de kan göra skalet långsammare, särskilt om koden de kör inte är väloptimerad eller om det blir för mycket kod.
Du kan alltid profilera och stänga av funktioner du sällan använder eller inte prioriterar högre än hastighet.

# Terminalemulatorer

Utöver att anpassa skalet är det värt att lägga lite tid på val av **terminalemulator** och dess inställningar.
Det finns väldigt många terminalemulatorer där ute (här är en [jämförelse](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)).

Eftersom du kan komma att tillbringa hundratals till tusentals timmar i terminalen lönar det sig att undersöka inställningarna.
Några aspekter du kan vilja ändra i terminalen är:

- Typsnittsval
- Färgschema
- Kortkommandon
- Stöd för flikar/paneler
- Scrollback-konfiguration
- Prestanda (vissa nyare terminaler som [Alacritty](https://github.com/jwilm/alacritty) eller [kitty](https://sw.kovidgoyal.net/kitty/) erbjuder GPU-acceleration).

# Övningar

## Jobbstyrning

1. Av det vi har sett kan vi använda kommandon som `ps aux | grep` för att få fram PID för jobb och sedan döda dem, men det finns bättre sätt.
   Starta ett jobb `sleep 10000` i en terminal, lägg det i bakgrunden med `Ctrl-Z` och fortsätt körningen med `bg`.
   Använd sedan [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) för att hitta dess PID och [`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) för att döda det utan att någonsin skriva PID direkt.
   (Tips: använd flaggorna `-af`).

1. Säg att du inte vill starta en process förrän en annan är klar.
   Hur skulle du göra det?
   I övningen är den begränsande processen alltid `sleep 60 &`.
Ett sätt att lösa det är att använda kommandot [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html).
Testa att starta sleep-kommandot och låta ett `ls` vänta tills bakgrundsprocessen är klar.

    Den här strategin misslyckas dock om vi startar i en annan bash-session, eftersom `wait` bara fungerar för barnprocesser. En funktion som vi inte tog upp i anteckningarna är att `kill`-kommandots slutstatus är noll vid framgång och skild från noll annars. `kill -0` skickar ingen signal men ger en slutstatus skild från noll om processen inte finns.
    Skriv en bash-funktion `pidwait` som tar en PID och väntar tills den processen är klar. Du bör använda `sleep` för att undvika onödig CPU-förbrukning.

## Terminalmultiplexrar

1. Följ den här `tmux`-[guiden](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) och lär dig sedan några grundläggande anpassningar enligt [dessa steg](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/).

## Alias

1. Skapa ett alias `dc` som blir `cd` för när du skriver fel.

1.  Kör `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` för att få dina 10 mest använda kommandon och överväg att skriva kortare alias för dem.
    Obs: detta fungerar för Bash; om du använder ZSH, använd `history 1` i stället för bara `history`.


## Dotfiles

Nu ska vi få upp farten med dotfiles.
1. Skapa en katalog för dina dotfiles och sätt upp versionshantering.
1. Lägg till konfiguration för minst ett program, t.ex. ditt skal, med någon anpassning (för att börja enkelt kan det räcka att anpassa skalprompten genom att sätta `$PS1`).
1. Sätt upp en metod för att snabbt (och utan manuellt arbete) installera dina dotfiles på en ny maskin.
   Det kan vara så enkelt som ett skalskript som anropar `ln -s` för varje fil, eller så kan du använda ett [specialiserat verktyg](https://dotfiles.github.io/utilities/).
1. Testa installationsskriptet på en ny virtuell maskin.
1. Migrera alla dina nuvarande verktygskonfigurationer till ditt dotfiles-kodförråd.
1. Publicera dina dotfiles på GitHub.

## Fjärrmaskiner

Installera en Linux-virtuell maskin (eller använd en befintlig) för övningen.
Om du inte är bekant med virtuella maskiner, titta på [den här](https://hibbard.eu/install-ubuntu-virtual-box/) guiden för att installera en.

1. Gå till `~/.ssh/` och kontrollera om du har ett par SSH-nycklar där.
   Om inte, skapa dem med `ssh-keygen -a 100 -t ed25519`.
   Det rekommenderas att du använder ett lösenord och `ssh-agent`, mer info [här](https://www.ssh.com/ssh/agent).
1. Redigera `.ssh/config` så att den har en post enligt följande

    ```bash
    Host vm
        User username_goes_here
        HostName ip_goes_here
        IdentityFile ~/.ssh/id_ed25519
        LocalForward 9999 localhost:8888
    ```
1. Använd `ssh-copy-id vm` för att kopiera din ssh-nyckel till servern.
1. Starta en webbserver i din VM genom att köra `python -m http.server 8888`.
   Kom åt VM:ens webbserver genom att öppna `http://localhost:9999` på din maskin.
1. Redigera din SSH-serverkonfiguration med `sudo vim /etc/ssh/sshd_config` och stäng av lösenordsautentisering genom att ändra värdet på `PasswordAuthentication`.
   Stäng av root-inloggning genom att ändra värdet på `PermitRootLogin`.
   Starta om `ssh`-tjänsten med `sudo service sshd restart`.
   Försök logga in med ssh igen.
1. (Utmaning) Installera [`mosh`](https://mosh.org/) i VM:en och upprätta en anslutning.
   Koppla sedan från nätverksadaptern för servern/VM:en.
   Kan mosh återhämta sig korrekt?
1. (Utmaning) Ta reda på vad flaggorna `-N` och `-f` gör i `ssh` och hitta ett kommando för att åstadkomma portvidarebefordran i bakgrunden.
