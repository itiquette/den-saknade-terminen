---
layout: lecture
title: "Kommandoradsmiljön"
description: >
  Lär dig hur kommandoradsprogram fungerar, inklusive in-/utdataströmmar, miljövariabler och fjärrmaskiner med SSH.
thumbnail: /static/assets/thumbnails/2026/lec2.png
date: 2026-01-13
ready: true
video:
  aspect: 56.25
  id: ccBGsPedE9Q
---

Som vi tog upp i förra föreläsningen är de flesta skal inte bara en startare för andra program.
I praktiken erbjuder de ett helt programmeringsspråk med välkända mönster och abstraktioner.
Till skillnad från de flesta andra programmeringsspråk är allt i skalskriptning designat kring att köra program och låta dem kommunicera enkelt och effektivt.

Skalskriptning är dessutom starkt kopplad till _konventioner_.
För att ett kommandoradsprogram (CLI) ska fungera väl i den större skalmiljön finns några mönster det bör följa.
Vi kommer nu att täcka centrala begrepp för hur kommandoradsprogram fungerar såväl som vedertagna konventioner för hur de används och konfigureras.

# Kommandoradsgränssnittet

Att skriva en funktion i de flesta programmeringsspråk ser ut ungefär så här:

```
def add(x: int, y: int) -> int:
    return x + y
```

Här kan vi tydligt se programmets in- och utdata.
Skalskript kan däremot se ganska annorlunda ut vid en första anblick.

```shell
#!/usr/bin/env bash

if [[ -f $1 ]]; then
    echo "Målfilen finns redan"
    exit 1
else
    if $DEBUG; then
        grep 'error' - | tee $1
    else
        grep 'error' - > $1
    fi
    exit 0
fi
```

För att förstå vad som händer i sådana skript behöver vi några begrepp som ofta dyker upp när skalprogram kommunicerar med varandra eller med skalmiljön:

- Argument
- Strömmar
- Miljövariabler
- Returkoder
- Signaler

## Argument

Skalprogram tar emot en lista med argument när de körs.
Argument är vanliga strängar i skalet, och det är upp till programmet hur de ska tolkas.
När vi kör `ls -l folder/` kör vi till exempel programmet `/bin/ls` med argumenten `['-l', 'folder/']`.

Inifrån ett skalskript når vi dessa via särskild skalsyntax.
Första argumentet är `$1`, andra `$2` och så vidare till `$9`.
Alla argument som lista fås med `$@` och antal argument med `$#`.
Vi kan också läsa programmets namn via `$0`.

För de flesta program består argumenten av en blandning av _flaggor_ och vanliga strängar.
Flaggor känns igen på inledande bindestreck (`-`) eller dubbelt bindestreck (`--`).
Flaggor är oftast valfria och ändrar programmets beteende.
Till exempel ändrar `ls -l` hur `ls` formaterar utdata.

Du ser långa flaggor som `--all` och korta som `-a`, oftast med en bokstav.
Samma val kan ofta anges i båda formerna, `ls -a` och `ls --all` är ekvivalenta.
Korta flaggor kan ofta grupperas, så `ls -l -a` och `ls -la` är också ekvivalenta.
Ordningen på flaggor spelar vanligtvis ingen roll, `ls -la` och `ls -al` ger samma resultat.
Vissa flaggor återkommer ofta, till exempel `--help`, `--verbose` och `--version`.

> Flaggor är ett bra första exempel på skalets konventioner.
> Skalspråket kräver inte att program använder `-` eller `--` på det här sättet.
> Inget hindrar syntax som `myprogram +myoption myfile`,
> men det skapar förvirring eftersom förväntningen är bindestreck.
> I praktiken erbjuder de flesta språk bibliotek för CLI-flaggparsning (t.ex. `argparse` i Python).

En annan vanlig CLI-konvention är att ta ett varierande antal argument av samma typ.
När kommandot ges argument på det här sättet så utförs samma operation för varje argument.

```shell
mkdir src
mkdir docs
# motsvarar
mkdir src docs
```

Det kan först se ut som onödigt syntaktiskt socker, men blir väldigt kraftfullt i kombination med _mönstermatchning_ (globbing).
Mönstermatchning innebär att skalet expanderar speciella mönster innan programmet körs.

Säg att vi utan rekursion vill ta bort alla `.py`-filer i aktuell katalog.
Utifrån det vi lärde oss i förra föreläsningen kan vi köra:

```shell
for file in $(ls | grep -P '\.py$'); do
    rm "$file"
done
```

Men det går att ersätta med bara `rm *.py`.

När vi skriver `rm *.py` i terminalen kommer skalet inte anropa `/bin/rm` med argument `['*.py']`.
I stället letar skalet efter filer i aktuell katalog som matchar mönstret `*.py`, där `*` kan matcha vilken sträng som helst av noll eller fler tecken.
Om katalogen innehåller `main.py` och `utils.py` får `rm` alltså argumenten `['main.py', 'utils.py']`.

De vanligaste jokertecknen är `*` (noll eller fler av vad som helst), `?` (exakt ett av vad som helst) och klamrar.
Klamrar `{}` expanderar en kommaseparerad lista av mönster till flera argument.

Mönstermatchning förstås i praktiken bäst med exempel:

```shell
touch katalog/{a,b,c}.py
# Expanderar till
touch katalog/a.py katalog/b.py katalog/c.py

convert bild.{png,jpg}
# Expanderar till
convert bild.png bild.jpg

cp /sökväg/till/projekt/{setup,build,deploy}.sh /nysökväg
# Expanderar till
cp /sökväg/till/projekt/setup.sh /sökväg/till/projekt/build.sh /sökväg/till/projekt/deploy.sh /nysökväg

# Mönster kan kombineras
mv *{.py,.sh} katalog
# Flyttar alla *.py- och *.sh-filer
```

> Vissa skal (t.ex. zsh) har en mer avancerad mönstermatchning som `**` för rekursiva sökvägar.
> `rm **/*.py` tar då bort alla `.py`-filer rekursivt.

## Strömmar

När vi kör en programrörledning som

```shell
cat myfile | grep -P '\d+' | uniq -c
```

ser vi att `grep` kommunicerar både med `cat` och `uniq`.

En viktig observation är att alla tre program kör samtidigt.
Skalet kör alltså inte först `cat`, sedan `grep`, sedan `uniq` i sekvens.
I stället startas alla tre processer och skalet kopplar utdata från `cat` till indata för `grep`, och utdata från `grep` till indata för `uniq`.
Med operatorn `|` arbetar skalet med dataströmmar som flyter från ett program till nästa i kedjan.

Vi kan demonstrera samtidigheten:

```console
$ (sleep 15 && cat tal.txt) | grep -P '^\d$' | sort | uniq  &
[1] 12345
$ ps | grep -P '(sleep|cat|grep|sort|uniq)'
  32930 pts/1    00:00:00 sleep
  32931 pts/1    00:00:00 grep
  32932 pts/1    00:00:00 sort
  32933 pts/1    00:00:00 uniq
  32948 pts/1    00:00:00 grep
```

Vi ser att alla processer utom `cat` kör direkt.
Skalet startar processerna och kopplar deras strömmar innan någon av dem är klara.
`cat` börjar först när `sleep` är klar, och dess utdata skickas vidare till `grep` och så vidare.

Varje program har en inström, stdin (standard in).
När du använder rör kopplas stdin automatiskt.
I skript accepterar många program `-` som filnamn för "läs från stdin":

```shell
# Dessa är likvärdiga när data kommer från ett rör
echo "hej" | grep "hej"
echo "hej" | grep "hej" -
```

På motsvarande sätt har varje program två utströmmar: stdout och stderr.
Standard ut är den vanligaste och används för att skicka vidare genom ett rör till nästa kommando.
Standard error är en separat ström för varningar och fel, så att den utdata inte tolkas av nästa kommando i kedjan.

```console
$ ls /ickeexisterande
ls: cannot access '/ickeexisterande': No such file or directory
$ ls /ickeexisterande | grep "mönster"
ls: cannot access '/ickeexisterande': No such file or directory
# Felmeddelandet syns fortfarande eftersom stderr inte går genom röret
$ ls /nonexistent 2>/dev/null
# Ingen utdata - stderr omdirigerades till /dev/null
```

Skalet har syntax för att omdirigera strömmar.
Här några exempel:

```shell
# Omdirigera stdout till en fil (skriv över)
echo "hej" > utdata.txt

# Omdirigera stdout till en fil (lägg till)
echo "världen" >> utdata.txt

# Omdirigera stderr till en fil
ls foobar 2> fel.txt

# Omdirigera både stdout och stderr till samma fil
ls foobar &> all_utdata.txt

# Omdirigera stdin från en fil
grep "mönster" < indata.txt

# Kasta utdata genom att omdirigera till /dev/null
cmd > /dev/null 2>&1
```

Ett annat kraftfullt verktyg i sann Unix-anda är [`fzf`](https://github.com/junegunn/fzf), ett program för ungefärlig sökning.
Det läser rader från stdin med ett interaktivt gränssnitt för filtrering och val:

```console
$ ls | fzf
$ cat ~/.bash_history | fzf
```

`fzf` kan integreras med många skaloperationer.
Vi kommer att se fler användningsfall när vi pratar skalanpassning.

## Miljövariabler

För att tilldela variabler i bash använder vi `foo=bar`, och kommer sedan åt värdet med `$foo`.
Observera att `foo = bar` är ogiltig syntax, eftersom skalet då kommer att tolka det som att programmet `foo` anropas med argument `['=', 'bar']`.
I skalskriptning används blanktecken för argumentsplittring, vilket kan vara förvirrande tills man vant sig.

Skalvariabler har inga typer, de är alla strängar.
Observera också att enkla och dubbla citationstecken inte är utbytbara.
Strängar i `'` är bokstavliga och expanderar inte variabler, gör inte kommandosubstitution (_command substitution_) och tolkar inte kontrollsekvenser, medan strängar i `"` gör detta.

```shell
foo=bar
echo "$foo"
# skriver ut bar
echo '$foo'
# skriver ut $foo
```

För att fånga utdata från ett kommando i en variabel använder vi _kommandosubstitution_.
När vi kör:

```shell
files=$(ls)
echo "$files" | grep README
echo "$files" | grep ".py"
```

placeras stdout från `ls` i variabeln `$files`.
Innehållet i `$files` innehåller radbrytningar från `ls`, vilket gör att program som `grep` kan behandla varje post separat.

En mindre känd närliggande funktion är processsubstitution (_process substitution_).
`<( CMD )` kör `CMD`, placerar utdata i en temporär fil och ersätter `<()` med filnamnet.
Det är användbart när kommandon väntar sig filer i stället för stdin.
Till exempel visar `diff <(ls src) <(ls docs)` skillnader mellan filerna i `src` och `docs`.

När ett skalprogram anropar ett annat skickar det med en uppsättning variabler som ofta kallas _miljövariabler_.
I skalet kan du se nuvarande miljövariabler med `printenv`.
För att skicka en miljövariabel explicit kan vi prefixa ett kommando med tilldelning.

> Miljövariabler skrivs normalt med VERSALER (t.ex. `HOME`, `PATH`, `DEBUG`).
> Det är en konvention, inte ett tekniskt krav,
> men det hjälper att skilja dem från lokala skalvariabler som oftast är gemener.

```shell
TZ=Asia/Tokyo date  # skriver ut aktuell tid i Tokyo
echo $TZ  # blir tomt, eftersom TZ bara sattes för barnkommandot
```

Alternativt kan vi använda den inbyggda funktionen `export`, som ändrar nuvarande miljö så att alla barnprocesser ärver variabeln:

```shell
export DEBUG=1
# Alla program från och med nu får DEBUG=1 i sin miljö
bash -c 'echo $DEBUG'
# skriver ut 1
```

För att ta bort en variabel använder du `unset`, till exempel `unset DEBUG`.

> Miljövariabler är ännu en skalkonvention.
> De kan implicit ändra beteendet hos många program.
> Skalet sätter till exempel `$HOME` till nuvarande användares hemkatalog,
> och program kan läsa den i stället för att kräva ett explicit `--home /home/alice`.
> Ett annat vanligt exempel är `$TZ`,
> som många program använder för datum/tid i en viss tidszon.

## Returkoder

Som vi såg tidigare förmedlas huvudutdata från skalprogram via stdout/stderr och sidoeffekter i filsystemet.

Som standard returnerar ett skalskript slutkod noll.
Konventionen är att noll betyder att allt gick bra, medan icke-noll betyder att något gick fel.
För att returnera icke-noll använder vi den inbyggda funktionen `exit NUM`.
Returkoden från senaste kommandot finns i specialvariabeln `$?`.

Skalet har booleska operatorer `&&` och `||` för AND respektive OR.
Till skillnad från vanliga programmeringsspråk verkar operatorerna i skalet på programmens returkoder.
Båda är [kortslutande](https://en.wikipedia.org/wiki/Short-circuit_evaluation).
Det betyder att de kan användas för villkorlig körning baserat på om tidigare kommandon lyckades eller misslyckades.
Lyckat betyder här att returkoden är noll.
Exempel:

```shell
# echo körs bara om grep lyckas (hittar en träff)
grep -q "mönster" en_fil.txt && echo "Mönster hittat"

# echo körs bara om grep misslyckas (ingen träff)
grep -q "mönster" en_fil.txt || echo "Mönster saknas"

# true är ett skalprogram som alltid lyckas
true && echo "Det här skrivs alltid ut"

# och false är ett skalprogram som alltid misslyckas
false || echo "Det här skrivs alltid ut"
```

Samma princip gäller för `if` och `while`, som båda använder returkoder för beslut:

```shell
# if använder returvärdet från villkorskommandot (0 = sant, icke-noll = falskt)
if grep -q "mönster" en_fil.txt; then
    echo "Hittat"
fi

# while-slingor fortsätter så länge kommandot returnerar 0
while read line; do
    echo "$line"
done < en_fil.txt
```

## Signaler

Ibland behöver du avbryta ett program medan det kör, till exempel om ett kommando tar för lång tid.
Det enklaste är att trycka `Ctrl-C`, och då stoppas kommandot oftast. Men hur fungerar det egentligen, och varför misslyckas det ibland?

```console
$ sleep 100
^C
$
```

> Observera att `^C` är hur `Ctrl-C` visas i terminalen.

Under ytan händer detta:

1. Vi trycker `Ctrl-C`.
2. Skalet känner igen den särskilda tangentkombinationen.
3. Skalprocessen skickar signalen SIGINT till `sleep`-processen.
4. Signalen avbryter körningen i `sleep`-processen.

Signaler är en särskild kommunikationsmekanism.
När en process tar emot en signal stoppar den körningen, hanterar signalen, och kan ändra kontrollflödet utifrån informationen i signalen.
Därför är signaler _programvaruavbrott_.

När du trycker `Ctrl-C` får alltså skalet anledning att leverera `SIGINT` till processen.
Här är ett minimalt Python-program som fångar `SIGINT` och ignorerar den.
För att döda programmet kan vi då använda `SIGQUIT` genom att trycka `Ctrl-\`.

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

Så här ser det ut om vi skickar `SIGINT` två gånger, följt av `SIGQUIT`.
Notera att `^` är hur `Ctrl` visas i terminalen.

```console
$ python sigint.py
24^C
Jag fick en SIGINT, men jag tänker inte sluta
26^C
Jag fick en SIGINT, men jag tänker inte sluta
30^\[1]    39913 quit       python sigint.py
```

`SIGINT` och `SIGQUIT` kopplas ofta till terminalhändelser, men en mer generell signal för att be en process avsluta snyggt är `SIGTERM`.
Den skickas med [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html): `kill -TERM <PID>`.

Signaler kan göra mer än att avsluta processer.
`SIGSTOP` pausar till exempel en process.
I terminalen gör `Ctrl-Z` att skalet skickar `SIGTSTP`, alltså terminalvarianten av stopp.

Du kan fortsätta ett pausat jobb i förgrund eller bakgrund med [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) respektive [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html).

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) listar ofärdiga jobb kopplade till aktuell terminalsesssion.
Du kan referera till jobben med PID (hitta med [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html)).
Mer intuitivt kan du också hänvisa med procenttecken och jobbnummer från `jobs`.
För senast bakgrundssatta jobb kan du använda specialparametern `$!`.

Ännu en sak: suffixet `&` kör ett kommando i bakgrunden och ger tillbaka prompten, men processen kan fortfarande skriva till skalets STDOUT vilket kan vara störande.
Använd omdirigeringar i sådana fall.
På motsvarande sätt kan du lägga ett redan körande program i bakgrunden med `Ctrl-Z` följt av `bg`.

Bakgrundsprocesser är fortfarande barnprocesser till terminalen, och dör om du stänger terminalen (skickar `SIGHUP`).
För att undvika det kan du köra programmet via [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (som ignorerar `SIGHUP`) eller använda `disown` om processen redan startat.
Alternativt kan du använda en terminalmultiplexer, vilket vi visar i nästa avsnitt.

Nedan är en exempelsession som visar några av koncepten.

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

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup skyddar mot SIGHUP

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000
```

En särskild signal är `SIGKILL`, som inte kan fångas av processen och därför alltid dödar den direkt.
Den kan dock ge oönskade bieffekter, till exempel föräldralösa barnprocesser.

Läs mer om signaler [här](https://en.wikipedia.org/wiki/Signal_(IPC)), eller via [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) eller `kill -l`.

I skalskript kan du använda inbyggda `trap` för att köra kommandon när signaler tas emot, vilket är användbart för uppstädning:

```shell
#!/usr/bin/env bash
cleanup() {
    echo "Städar tillfälliga filer..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT  # Kör städning när skriptet avslutas
trap cleanup SIGINT SIGTERM  # Kör också vid Ctrl-C eller kill
```
{% comment %}
### Användare, filer och rättigheter

Till sist finns ytterligare ett sätt för program att kommunicera indirekt med varandra: via filer.
För att ett program ska kunna läsa/skriva/ta bort filer och kataloger korrekt måste filrättigheterna tillåta operationen.

Att lista en specifik fil kan ge följande utdata

```console
$ ls -l anteckningar.txt
-rw-r--r--  1 alice  users  12693 Jan 11 23:05 anteckningar.txt
```

Här visar `ls` vem som äger filen, användaren `alice`, och gruppen `users`.
`rw-r--r--` är en kort notation för rättigheterna.
Här har filen `anteckningar.txt` läs/skriv-rättigheter för användaren alice `rw-`, och endast läsrättigheter för gruppen och övriga användare i filsystemet.

```console
$ ./script.sh
# behörighet saknas
$ chmod +x script.sh
$ ls -l script.sh
-rwxr-xr-x  1 alice  users  3125 Jan 11 23:07 script.sh
$ ./script.sh
```

För att ett skript ska vara körbart måste kör-rättighet vara satt, därför behövde vi använda `chmod` (change mode).
`chmod`-syntaxen är intuitiv när man väl kan den, men inte självklar första gången.
Om du, som jag, föredrar att lära dig med exempel är detta ett bra användningsfall för verktyget `tldr` (som du först behöver installera).

```console
❯ tldr chmod
  Change the access permissions of a file or directory.
  More information: <https://www.gnu.org/software/coreutils/chmod>.

  Give the [u]ser who owns a file the right to e[x]ecute it:

      chmod u+x path/to/file

  Give the [u]ser rights to [r]ead and [w]rite to a file/directory:

      chmod u+rw path/to/file_or_directory

  Give [a]ll users rights to [r]ead and e[x]ecute:

      chmod a+rx path/to/file
```

Kör `tldr chmod` för fler exempel, inklusive rekursiva operationer och grupprättigheter.

> Ditt skal kan visa något i stil med `command not found: tldr`.
> Det beror på att det är ett modernare verktyg som inte är förinstallerat på de flesta system.
> En bra referens för hur du installerar verktyg är [https://command-not-found.com](https://command-not-found.com).
> Sidan innehåller instruktioner för en stor mängd CLI-verktyg i vanliga OS-distributioner.

Varje program körs som en specifik användare i systemet.
Vi kan använda kommandot `whoami` för att hitta användarnamnet och `id -u` för att hitta vårt UID (user id), alltså heltalsvärdet som operativsystemet associerar med användaren.

När du kör `sudo command` körs `command` som root-användaren, som kan kringgå de flesta rättigheter i systemet.
Prova `sudo whoami` och `sudo id -u` för att se hur utdata ändras (du kan bli ombedd att ange lösenord).
För att ändra ägare på en fil eller katalog använder vi kommandot `chown`.

Du kan läsa mer om UNIX-filsystemrättigheter [här](https://en.wikipedia.org/wiki/File-system_permissions#Traditional_Unix_permissions)

Hittills har vi fokuserat på din lokala maskin, men många av dessa färdigheter blir ännu mer värdefulla när du arbetar med fjärrservrar.

{% endcomment %}

# Fjärrmaskiner

Det har blivit allt vanligare att programmerare arbetar mot fjärrservrar i vardagen.
Det vanligaste verktyget här är SSH (Secure Shell), som hjälper oss att ansluta till en fjärrserver och har ett välkänt skalgränssnitt.
Vi ansluter till en server med kommandot:

```bash
ssh alice@server.mit.edu
```

Här försöker vi ansluta som användaren `alice` till servern `server.mit.edu`.

En ofta förbisedd funktion i `ssh` är att köra kommandon icke-interaktivt.
`ssh` hanterar både stdin till kommandot och stdout tillbaka korrekt, så vi kan kombinera det med andra kommandon:

```shell
# Här körs ls på fjärrmaskinen och wc lokalt
ssh alice@server ls | wc -l

# Här körs både ls och wc på servern
ssh alice@server 'ls | wc -l'

```

> Testa gärna [Mosh](https://mosh.org/) som SSH-alternativ.
> Det hanterar avbrott, sömn/vakna, nätverksbyten och hög latens bättre.

För att `ssh` ska låta oss köra kommandon på servern måste vi bevisa att vi är behöriga.
Det kan göras med lösenord eller SSH-nycklar.
Nyckelbaserad autentisering använder publik nyckelkryptografi för att bevisa att klienten har den privata nyckeln utan att avslöja den.
Nyckelbaserad autentisering är både smidigare och säkrare, så den bör föredras.
Observera att den privata nyckeln (ofta `~/.ssh/id_rsa` och numera oftare `~/.ssh/id_ed25519`) i praktiken är ditt lösenord.
Behandla den därefter och dela aldrig dess innehåll.

För att generera ett nyckelpar kan du köra [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html).

```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

Om du har konfigurerat push till GitHub via SSH har du sannolikt redan följt stegen [här](https://help.github.com/articles/connecting-to-github-with-ssh/) och har ett giltigt nyckelpar.
För att kontrollera passphrase och verifiera nyckeln kan du köra `ssh-keygen -y -f /path/to/key`.

På serversidan tittar `ssh` i `.ssh/authorized_keys` för att avgöra vilka klienter som tillåts.
För att kopiera över en publik nyckel kan du använda:

```bash
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'

# eller enklare (om ssh-copy-id finns)

ssh-copy-id -i .ssh/id_ed25519 alice@remote
```

Utöver kommandokörning kan SSH-anslutningen användas för säker filöverföring till och från servern.
[`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) är det mest traditionella verktyget och syntaxen är `scp path/to/local_file remote_host:path/to/remote_file`.
[`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) förbättrar `scp` genom att känna igen identiska filer lokalt/fjärr och undvika att kopiera dem igen.
Det ger också finare kontroll över symlänkar och rättigheter och har extrafunktioner som `--partial`, som kan återuppta en avbruten kopiering.
`rsync` har liknande syntax som `scp`.

Konfiguration av SSH-klienten ligger i `~/.ssh/config` och låter oss deklarera värdar och standardinställningar.
Den här filen läses inte bara av `ssh` utan även av verktyg som `scp`, `rsync`, `mosh`, etc.

```bash
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Konfigurationen kan också använda jokertecken
Host *.mit.edu
    User alice
```

# Terminalmultiplexrar

När du arbetar i kommandoraden vill du ofta köra mer än en sak samtidigt.
Du kanske till exempel vill ha redigeraren och programmet sida vid sida.
Det går att lösa med flera terminalfönster, men en terminalmultiplexer är mer flexibel.

Terminalmultiplexrar som [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) låter dig dela upp terminalfönster i paneler och flikar, så att du kan arbeta effektivt med flera skalsessioner.
Dessutom kan du koppla från en pågående session och återansluta senare.
Det gör terminalmultiplexrar särskilt praktiska på fjärrmaskiner, eftersom du slipper `nohup` och liknande knep.

Den mest populära terminalmultiplexern i dag är [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html).
`tmux` är i hög grad konfigurerbart, och med rätt kortkommandon kan du skapa flera flikar och paneler och snabbt växla mellan dem.

`tmux` bygger på att du kan dess kortkommandon.
De har formen `<C-b> x`, vilket betyder: (1) tryck `Ctrl+b`, (2) släpp, (3) tryck `x`.
`tmux` har följande objekt-hierarki:

- **Sessioner** - en session är en separat arbetsyta med ett eller flera fönster.
    + `tmux` startar en ny session.
    + `tmux new -s NAME` startar med angivet namn.
    + `tmux ls` listar aktuella sessioner.
    + Inne i `tmux` kopplar `<C-b> d` från aktuell session.
    + `tmux a` ansluter till senaste sessionen.
      Du kan använda `-t` för att välja vilken.

- **Fönster** - motsvarar flikar i editorer/webbläsare.
    + `<C-b> c` skapar ett nytt fönster.
      För att stänga det kan du avsluta skalet med `<C-d>`.
    + `<C-b> N` går till fönster nummer _N_.
    + `<C-b> p` går till föregående fönster.
    + `<C-b> n` går till nästa fönster.
    + `<C-b> ,` byter namn på aktuellt fönster.
    + `<C-b> w` listar aktuella fönster.

- **Paneler** - likt splits i vim låter paneler dig ha flera skal i samma vy.
    + `<C-b> "` delar aktuell panel horisontellt.
    + `<C-b> %` delar aktuell panel vertikalt.
    + `<C-b> <direction>` flyttar till panel i angiven riktning (piltangenter).
    + `<C-b> z` växlar zoom för aktuell panel.
    + `<C-b> [` startar scrollback.
      Där kan du trycka `<space>` för markering och `<enter>` för kopiering.
    + `<C-b> <space>` cyklar panelarrangemang.

> För mer om tmux, läs gärna [den här](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) snabba guiden och [den här](https://linuxcommand.org/lc3_adv_termmux.php) mer detaljerade genomgången.

Med tmux och SSH i verktygslådan vill du snart få miljön att kännas som hemma på alla maskiner.
Där kommer skalanpassning in.

# Anpassa skalet

Många kommandoradsprogram konfigureras med textfiler som kallas _dotfiles_ (eftersom filnamnen börjar med `.`, t.ex. `~/.vimrc`, och därför döljs i `ls` som standard).

> Dotfiles är ännu en skalkonvention.
> Punkten i början används för att "dölja" filen i listningar.

Skal är ett exempel på program som konfigureras med sådana filer.
Vid uppstart läser skalet flera filer för att ladda konfiguration.
Beroende på skal och om du startar login-/interaktiv session kan processen vara ganska komplex. [Här](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) finns en utmärkt resurs.

För `bash` fungerar det på de flesta system att redigera `.bashrc` eller `.bash_profile`.
Andra verktyg som kan konfigureras via dotfiles:

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` och katalogen `~/.vim`
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

En vanlig ändring är att lägga till nya sökvägar där skalet ska hitta program.
Du ser mönstret ofta vid installation av programvara:

```shell
export PATH="$PATH:path/to/append"
```

Här sätter vi `$PATH` till nuvarande värde plus en ny sökväg, och låter barnprocesser ärva det.
Då kan de hitta program under `path/to/append`.

Att anpassa skalet betyder ofta att installera nya CLI-verktyg.
Pakethanterare gör detta enkelt.
De hanterar nedladdning, installation och uppdateringar.
Olika operativsystem har olika pakethanterare: macOS använder [Homebrew](https://brew.sh/), Ubuntu/Debian använder `apt`, Fedora använder `dnf`, och Arch använder `pacman`.
Vi går djupare i detta i föreläsningen om att distribuera kod.

Så här installerar du två användbara verktyg med Homebrew på macOS:

```shell
# ripgrep: ett snabbare grep med bättre standardvärden
brew install ripgrep

# fd: ett snabbare och mer användarvänligt alternativ till `find`
brew install fd
```

Efter installation kan du använda `rg` i stället för `grep` och `fd` i stället för `find`.

> **Varning för `curl | bash`**: Du ser ofta installationskommandon som `curl -fsSL https://example.com/install.sh | bash`.
> Det här laddar ner ett skript och kör det direkt,
> vilket är bekvämt men riskabelt eftersom du kör kod du inte granskat.
> Säkrare är att ladda ner först, granska och sedan köra:
> ```shell
> curl -fsSL https://example.com/install.sh -o install.sh
> less install.sh  # granska skriptet
> bash install.sh
> ```
> Vissa installationer använder en något säkrare variant: `/bin/bash -c "$(curl -fsSL https://url)"`.

När du försöker köra ett kommando som inte är installerat visar skalet `command not found`.
Webbplatsen [command-not-found.com](https://command-not-found.com) är en bra resurs för att hitta installationsinstruktioner i olika pakethanterare och distributioner.

Ett annat användbart verktyg är [`tldr`](https://tldr.sh/), som ger förenklade man-sidor med fokus på exempel.
I stället för att läsa igenom mängder av dokumentation ser du snabbt vanligt förekommande användningsmönster:

```console
$ tldr fd
  An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt
```

Ibland behöver du inte ett nytt program, utan bara en genväg till ett befintligt kommando med vissa flaggor.
Där kommer alias in.

Vi kan skapa egna alias med inbyggda `alias`.
Ett skalalias är en kortform som skalet ersätter automatiskt innan uttrycket evalueras.
I bash ser strukturen ut så här:

```bash
alias alias_name="command_to_alias arg1 arg2"
```

> Notera att det inte ska vara mellanslag runt `=`,
> eftersom [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) är ett skalkommando som tar ett enda argument.

Alias har många praktiska användningar:

```bash
# Skapa kortformer för vanliga flaggor
alias ll="ls -lh"

# Spara mycket skrivande för vanliga kommandon
alias gs="git status"
alias gc="git commit"

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

Alias har begränsningar: de kan inte ta argument i mitten av ett kommando.
För mer avancerat beteende bör du använda skalfunktioner.

De flesta skal stöder `Ctrl-R` för omvänd historiksökning.
Tryck `Ctrl-R` och börja skriva för att söka bland tidigare kommandon.
Tidigare introducerade vi `fzf` som ungefärlig sökare.
Med fzf:s skalintegration blir `Ctrl-R` en interaktiv ungefärlig sökning i hela historiken, betydligt kraftfullare än standardläget.

Hur bör du organisera dina dotfiles?
De bör ligga i en egen katalog, under versionshantering, och **symboliskt länkas** in på plats med ett skript.
Det ger:

- **Enkel installation**: på en ny maskin tar det bara någon minut att få allt på plats.
- **Portabilitet**: dina verktyg fungerar likadant överallt.
- **Synkronisering**: du kan uppdatera dotfiles var som helst och hålla allt synkroniserat.
- **Historik**: du kommer sannolikt att underhålla dotfiles länge, och då är versionshistorik värdefullt.

Vad ska ligga i dotfiles?
Lär dig verktygens inställningar via dokumentation på nätet eller [man-sidor](https://en.wikipedia.org/wiki/Man_page).
Ett annat bra sätt är blogginlägg om specifika program, där författare beskriver sina favoritinställningar.
Du kan också läsa andras dotfiles: det finns mängder av [dotfiles-kodförråd](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) på GitHub.
Se det mest populära [här](https://github.com/mathiasbynens/dotfiles) (kopiera inte blint).
[Här](https://dotfiles.github.io/) finns ytterligare en bra resurs.

Alla kursens lärare har sina dotfiles offentliga på GitHub: [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles).

**Ramverk och insticksmoduler** kan också förbättra skalet.
Populära ramverk är [prezto](https://github.com/sorin-ionescu/prezto) och [oh-my-zsh](https://ohmyz.sh/), plus mindre insticksmoduler för specifika funktioner:

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - färgar giltiga/ogiltiga kommandon medan du skriver
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - föreslår kommandon från historik medan du skriver
- [zsh-completions](https://github.com/zsh-users/zsh-completions) - fler kompletteringsdefinitioner
- [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - fish-liknande historiksökning
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) - snabbt, anpassningsbart prompttema

Skal som [fish](https://fishshell.com/) har många av dessa funktioner som standard.

> Du behöver inte ett stort ramverk som oh-my-zsh för att få funktionerna.
> Att installera enskilda insticksmoduler är ofta snabbare och ger bättre kontroll.
> Stora ramverk kan sakta ner skalets uppstart avsevärt,
> så installera helst bara det du faktiskt använder.

# AI i skalet

Det finns många sätt att använda AI-verktyg i skalet.
Här är några exempel med olika integrationsnivå:

**Kommandogenerering**: verktyg som [`simonw/llm`](https://github.com/simonw/llm) kan hjälpa till att generera skalkommandon från naturligt språk:

```console
$ llm cmd "hitta alla Python-filer som ändrats senaste veckan"
find . -name "*.py" -mtime -7
```

**Pipelineintegration**: LLM:er kan integreras i pipelines för att bearbeta och transformera data.
De är särskilt användbara när du vill extrahera information ur inkonsekventa format där regex vore besvärligt:

```console
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor
$ INSTRUCTIONS="Extrahera bara användarnamnet från varje rad, en per rad, inget annat"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```

Notera att vi använder `"$INSTRUCTIONS"` (citerat) eftersom variabeln innehåller blanksteg, och `< users.txt` för att omdirigera filens innehåll till stdin.

**AI-skal**: verktyg som [Claude Code](https://docs.anthropic.com/en/docs/claude-code) fungerar som ett meta-skal som tar engelska instruktioner och översätter dem till skaloperationer, filändringar och mer komplexa flerstegsuppgifter.

# Terminalemulatorer

Utöver att anpassa skalet är det värt att lägga lite tid på valet av **terminalemulator** och dess inställningar.
En terminalemulator är ett GUI-program som tillhandahåller det textbaserade gränssnitt där skalet körs.
Det finns många alternativ.

Eftersom du sannolikt tillbringar hundratals till tusentals timmar i terminalen lönar det sig att utforska inställningarna.
Exempel på saker du kan vilja justera:

- Typsnittsval
- Färgschema
- Tangentbordsgenvägar
- Flik-/panelstöd
- Scrollback-konfiguration
- Prestanda (vissa nyare terminaler som [Alacritty](https://github.com/alacritty/alacritty) eller [Ghostty](https://ghostty.org/) erbjuder GPU-acceleration)

# Övningar

## Argument och mönstermatchning

1. Du kan se kommandon som `cmd --flag -- --notaflag`.
   `--` är ett specialargument som säger åt programmet att sluta tolka flaggor.
   Allt efter `--` behandlas som positionsargument.
   Varför kan det vara användbart?
   Testa `touch -- -myfile` och ta sedan bort filen utan `--`.

1. Läs [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) och skriv ett `ls`-kommando som listar filer på följande sätt:
    - Visar alla filer, även dolda.
    - Storlekar visas i läsbart format för människor (t.ex. 454M i stället för 454279954).
    - Filer sorteras efter nyast först.
    - Utdata är färglagd.

   Exempelutdata:

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. Processsubstitution (_process substitution_) `<(command)` låter dig använda ett kommandos utdata som om det vore en fil.
   Använd `diff` med processsubstitution för att jämföra utdata från `printenv` och `export`.
   Varför skiljer de sig?
   (Tips: testa `diff <(printenv | sort) <(export | sort)`.)

## Miljövariabler

1. Skriv bash-funktionerna `marco` och `polo` som gör följande: när du kör `marco` ska nuvarande arbetskatalog sparas, och när du kör `polo` ska du, oavsett var du befinner dig, `cd` tillbaka till katalogen där `marco` kördes.
   För enklare felsökning kan du skriva koden i en fil `marco.sh` och (om)ladda definitionerna med `source marco.sh`.

{% comment %}
marco() { export MARCO=$(pwd) }

polo() { cd "$MARCO" }
{% endcomment %}

## Returkoder

1. Anta att du har ett kommando som sällan misslyckas.
   För felsökning vill du fånga utdata, men det kan ta lång tid att få ett felkörningstillfälle.
   Skriv ett bash-skript som kör följande skript tills det misslyckas, fångar stdout och stderr till filer, och skriver ut allt i slutet.
   Bonus om du också rapporterar hur många körningar som krävdes innan fel.

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Något gick fel"
       >&2 echo "Felet var användning av magiska tal"
       exit 1
    fi

    echo "Allt gick enligt plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0 until [[ "$?" -ne 0 ]]; do count=$((count+1)) ./random.sh &> out.txt done

echo "hittade fel efter $count körningar" cat out.txt
{% endcomment %}

## Signaler och jobbstyrning

1. Starta ett jobb `sleep 10000` i en terminal, lägg det i bakgrunden med `Ctrl-Z` och fortsätt körningen med `bg`.
   Använd sedan [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) för att hitta PID och [`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) för att döda processen utan att skriva PID manuellt.
   (Tips: använd flaggorna `-af`.)

1. Säg att du inte vill starta en process förrän en annan har avslutats.
   I övningen är den begränsande processen alltid `sleep 60 &`.
   Ett sätt är kommandot [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html).
   Testa att starta sleep och låt ett `ls` vänta tills bakgrundsprocessen är klar.

   Den strategin misslyckas dock om du startar i en annan bash-session, eftersom `wait` bara fungerar för barnprocesser.
   En funktion vi inte nämnde tidigare är att `kill` returnerar noll vid framgång och icke-noll annars.
   `kill -0` skickar ingen signal men ger icke-noll om processen inte finns.
   Skriv en bash-funktion `pidwait` som tar en PID och väntar tills processen avslutas.
   Du bör använda `sleep` för att undvika onödig CPU-förbrukning.

## Filer och rättigheter

1. (Avancerad) Skriv ett kommando eller skript som rekursivt hittar den senast modifierade filen i en katalog.
   Mer allmänt, kan du lista alla filer sorterade efter senaste?

## Terminalmultiplexrar

1. Följ denna `tmux`-[guide](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/), och lär dig sedan några grundläggande anpassningar via [de här stegen](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/).

## Alias och dotfiles

1. Skapa ett alias `dc` som expanderar till `cd` för när du skriver fel.

1. Kör `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` för att få dina 10 mest använda kommandon och överväg kortare alias för dem.
   Notera: detta gäller Bash.
   Om du använder ZSH, använd `history 1` i stället för bara `history`.

1. Skapa en katalog för dina dotfiles och lägg den under versionshantering.

1. Lägg till konfiguration för minst ett program, t.ex. ditt skal, med någon anpassning.
   För att komma igång kan det räcka att ändra skalets prompt genom att sätta `$PS1`.

1. Sätt upp ett sätt att installera dina dotfiles snabbt och utan manuellt arbete på en ny maskin.
   Det kan vara så enkelt som ett skalskript som kör `ln -s` för varje fil, eller ett [specialiserat verktyg](https://dotfiles.github.io/utilities/).

1. Testa installationsskriptet på en ren virtuell maskin.

1. Migrera alla dina nuvarande verktygskonfigurationer till ditt dotfiles-kodförråd.

1. Publicera dina dotfiles på GitHub.

## Fjärrmaskiner (SSH)

Installera en Linux-VM (eller använd en befintlig) för övningarna.
Om du inte är bekant med virtuella maskiner, se [den här](https://hibbard.eu/install-ubuntu-virtual-box/) guiden.

1. Gå till `~/.ssh/` och kontrollera om du har ett SSH-nyckelpar.
   Om inte, skapa ett med `ssh-keygen -a 100 -t ed25519`.
   Rekommendationen är att använda lösenfras och `ssh-agent`, mer info [här](https://www.ssh.com/ssh/agent).

1. Redigera `.ssh/config` så att den har en post som:

    ```bash
    Host vm
        User username_goes_here
        HostName ip_goes_here
        IdentityFile ~/.ssh/id_ed25519
        LocalForward 9999 localhost:8888
    ```

1. Använd `ssh-copy-id vm` för att kopiera din SSH-nyckel till servern.

1. Starta en webbserver i din VM med `python -m http.server 8888`.
   Gå till `http://localhost:9999` på din egen maskin för att nå webbservern i VM.

1. Redigera SSH-serverkonfigurationen med `sudo vim /etc/ssh/sshd_config` och stäng av lösenordsautentisering genom att ändra `PasswordAuthentication`.
   Stäng också av root-inloggning genom att ändra `PermitRootLogin`.
   Starta om SSH-tjänsten med `sudo service sshd restart`.
   Testa att SSH:a in igen.

1. (Utmaning) Installera [`mosh`](https://mosh.org/) i VM:n och upprätta en anslutning.
   Koppla sedan bort nätverksadaptern för servern/VM:n.
   Kan mosh återhämta sig korrekt?

1. (Utmaning) Ta reda på vad flaggorna `-N` och `-f` gör i `ssh` och hitta ett kommando för portvidarebefordran i bakgrunden.
