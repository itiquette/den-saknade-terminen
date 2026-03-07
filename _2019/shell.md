---
layout: lecture
title: "Skal och skriptning"
presenter: Jon
date: 2019-01-15
order: 3
video:
  aspect: 56.25
  id: dbDRfmH5uSI
---

Skalet är ett effektivt, textbaserat gränssnitt till din dator.

Skalprompten är det som möter dig när du öppnar en terminal.
Den låter dig köra program och kommandon.
Vanliga kommandon är:

 - `cd` för att byta katalog
 - `ls` för att lista filer och kataloger
 - `mv` och `cp` för att flytta och kopiera filer

Men skalet låter dig göra _så_ mycket mer.
Du kan anropa vilket program som helst på datorn,
och det finns kommandoradsverktyg för i princip allt du kan vilja göra.
De är ofta effektivare än grafiska motsvarigheter.
Vi går igenom många av dem i den här kursen.

Skalet erbjuder också ett interaktivt programmeringsspråk ("skriptning").
Det finns många skal:

 - Du har troligen använt `sh` eller `bash`.
 - Det finns också skal som följer språk, som `csh`.
 - Eller "bättre" skal som `fish`, `zsh` och `ksh`.

I den här kursen fokuserar vi på de allmänt förekommande `sh` och `bash`,
men prova gärna andra.
Jag gillar `fish`.

Skalprogrammering är ett *mycket* användbart verktyg i din verktygslåda.
Du kan antingen skriva program direkt i prompten,
eller i en fil.
`#!/bin/sh` + `chmod +x` gör ett skalskript körbart.

## Arbeta med skalet

Kör ett kommando flera gånger:

```bash
for i in $(seq 1 5); do echo hello; done
```

Det finns mycket att packa upp här:

 - `for x in list; do BODY; done`
   - `;` avslutar ett kommando, motsvarar radbrytning
   - delar upp `list`, tilldelar varje element till `x`, och kör body
   - uppdelningen är "whitespace splitting", som vi återkommer till
   - skalet använder inte klammerparenteser här, därför `do` + `done`
 - `$(seq 1 5)`
   - kör programmet `seq` med argumenten `1` och `5`
   - ersätter hela `$()` med programmets utdata
   - motsvarar
     ```bash
     for i in 1 2 3 4 5
     ```
 - `echo hello`
   - allt i ett skalskript är ett kommando
   - här kör vi kommandot `echo`, som skriver ut sina argument
     med argumentet `hello`.
   - alla kommandon söks i `$PATH` (kolonseparerad)

Vi har variabler:
```bash
for f in $(ls); do echo $f; done
```

Det skriver ut varje filnamn i aktuell katalog.
Du kan också sätta variabler med `=` (ingen blanktecken):

```bash
foo=bar
echo $foo
```

Det finns också en mängd "specialvariabler":

 - `$1` till `$9`: argument till skriptet
 - `$0`: namnet på själva skriptet
 - `$#`: antal argument
 - `$$`: process-ID för nuvarande skal

För att bara skriva ut kataloger:

```bash
for f in $(ls); do if test -d $f; then echo dir $f; fi; done
```

Här finns mer att packa upp:

 - `if CONDITION; then BODY; fi`
   - `CONDITION` är ett kommando.
     Om det avslutas med statuskod 0 (lyckat), körs `BODY`.
   - du kan också använda `else` eller `elif`
   - återigen inga klammerparenteser, därför `then` + `fi`
 - `test` är ett annat program som erbjuder olika kontroller och jämförelser,
   och avslutas med 0 om villkoret är sant (`$?`)
   - `man COMMAND` är din vän: `man test`
   - kan också anropas med `[` + `]`: `[ -d $f ]`
     - se `man test` och `which "["`

Men vänta.
Det här är fel.
Vad händer om en fil heter "My Documents"?

 - `for f in $(ls)` expanderar till `for f in My Documents`
 - först körs testet på `My`, sedan på `Documents`
 - inte alls vad vi ville
 - det här är en av de största felkällorna i skalskript

## Argumentsplittring

Bash delar argument på blanktecken,
vilket inte alltid är det du vill.

 - du behöver citattecken för att hantera mellanslag i argument
   `for f in "My Documents"` skulle fungera korrekt
 - samma problem finns någon annanstans, ser du var?
   `test -d $f`: om `$f` innehåller blanktecken får `test` fel
 - `echo` råkar vara okej,
   eftersom split + join med mellanslag
 - men vad händer om ett filnamn innehåller radbrytning?
   då blir det ett blanksteg
 - citera alla variabler som du inte vill ska splittras
 - men hur fixar vi skriptet ovan?
   vad tror du att `for f in "$(ls)"` gör?

Globbing är svaret.

 - bash kan hitta filer med mönster:
   - `*` vilken teckensträng som helst
   - `?` ett godtyckligt enskilt tecken
   - `{a,b,c}` något av dessa tecken
 - `for f in *`: alla filer i den här katalogen
 - vid globbing blir varje matchad fil ett eget argument
   - du måste fortfarande citera vid _användning_: `test -d "$f"`
 - du kan skapa avancerade mönster:
   - `for f in a*`: alla filer i aktuell katalog som börjar på `a`
   - `for f in foo/*.txt`: alla `.txt`-filer i `foo`
   - `for f in foo/*/p??.txt`
     alla textfiler på tre bokstäver som börjar på p i underkataloger till `foo`

Problem med blanktecken slutar inte där:

 - `if [ $foo = "bar" ]; then` -- ser du problemet?
 - vad händer om `$foo` är tom?
   argumenten till `[` blir `=` och `bar`...
 - det _går_ att kringgå med `[ x$foo = "xbar" ]`, men usch
 - använd i stället `[[`:
   bash-inbyggd jämförare med särskild parsning
   - den tillåter också `&&` i stället för `-a`, `||` i stället för `-o`, osv.

<!-- TODO: arrays? $@. ${array[@]} vs "${array[@]}". -->

## Komponerbarhet

Skalet är kraftfullt delvis tack vare komponerbarhet.
Du kan kedja flera program i stället för att ha ett enda program som gör allt.

Nyckeltecknet är `|` (pipe).

 - `a | b` betyder att både `a` och `b` körs
   och att all utdata från `a` skickas som indata till `b`
   och att utdata från `b` skrivs ut

Alla program du startar ("processer") har tre "strömmar":

 - `STDIN`: när programmet läser indata kommer den härifrån
 - `STDOUT`: när programmet skriver ut något går det hit
 - `STDERR`: en andra utström som programmet kan välja att använda
 - som standard är `STDIN` ditt tangentbord,
   och `STDOUT` och `STDERR` går båda till terminalen.
   Men det kan du ändra.
   - `a | b` kopplar `STDOUT` för `a` till `STDIN` för `b`.
   - du har också:
     - `a > foo` (`STDOUT` från `a` går till filen `foo`)
     - `a 2> foo` (`STDERR` från `a` går till filen `foo`)
     - `a < foo` (`STDIN` till `a` läses från filen `foo`)
     - tips: `tail -f` skriver ut en fil medan den skrivs
 - varför är detta användbart?
   för att du kan bearbeta ett programs utdata.
   - `ls | grep foo`: alla filer som innehåller ordet `foo`
   - `ps | grep foo`: alla processer som innehåller ordet `foo`
   - `journalctl | grep -i intel | tail -n5`:
     de senaste 5 systemloggraderna med ordet intel (skiftlägesokänsligt)
   - `who | sendmail -t me@example.com`
     skicka listan över inloggade användare till `me@example.com`
   - detta är grunden för mycket datahantering,
     som vi tar upp senare

Bash har också flera andra sätt att komponera program.

Du kan gruppera kommandon med `(a; b) | tac`.
Det kör `a`, sedan `b`, och skickar all deras utdata till `tac`,
som skriver ut indata i omvänd ordning.

Ett mindre känt men mycket användbart sätt är _process substitution_.
`b <(a)` kör `a`,
skapar ett temporärt filnamn för dess utström,
och skickar det filnamnet till `b`.
Till exempel:

```bash
diff <(journalctl -b -1 | head -n20) <(journalctl -b -2 | head -n20)
```
visar skillnaden mellan de första 20 raderna i senaste bootloggen och bootloggen före den.

<!-- TODO: exit codes? -->

## Jobb- och processkontroll

Vad gör du om du vill köra långvariga saker i bakgrunden?

 - suffixet `&` kör ett program "i bakgrunden"
   - du får tillbaka prompten direkt
   - praktiskt om du vill köra två program samtidigt,
     som server och klient: `server & client`
   - notera att programmet fortfarande har din terminal som `STDOUT`
     prova: `server > server.log & client`
 - se alla sådana processer med `jobs`
   - notera att det står "Running"
 - ta tillbaka ett jobb i förgrunden med `fg %JOB` (utan argument = senaste)
 - om du vill bakgrundssätta aktuellt program: `^Z` + `bg` (`^Z` betyder `Ctrl+Z`)
   - `^Z` stoppar aktuell process och gör den till ett "jobb"
   - `bg` kör senaste jobbet i bakgrunden (som om du hade skrivit `&`)
 - bakgrundsjobb är fortfarande knutna till din nuvarande session,
   och avslutas när du loggar ut.
   `disown` låter dig bryta den kopplingen.
   Eller använd `nohup`.
 - `$!` är pid för senaste bakgrundsprocessen

<!-- TODO: process output control (^S and ^Q)? -->

Vad gäller annan processaktivitet på datorn?

 - `ps` är din vän: listar körande processer
   - `ps -A`: skriv ut processer från alla användare (också `ps ax`)
   - `ps` har *många* argument: se `man ps`
 - `pgrep`: hitta processer via sökning (som `ps -A | grep`)
   - `pgrep -af`: sök och visa med argument
 - `kill`: skicka en _signal_ till en process via ID (`pkill` söker + `-f`)
   - signaler säger åt en process att "göra något"
   - vanligast: `SIGKILL` (`-9` eller `-KILL`): säg åt den att avsluta *nu*
     motsvarar `^\`
   - även `SIGTERM` (`-15` eller `-TERM`): be den avsluta kontrollerat
     motsvarar `^C`


## Flaggor

De flesta kommandoradsverktyg tar parametrar via **flaggor**.
Flaggor finns oftast i kort form (`-h`) och lång form (`--help`).
Vanligen ger `CMD -h` eller `man CMD` en lista över flaggor som programmet stöder.
Korta flaggor kan oftast kombineras,
så `rm -r -f` är samma som `rm -rf` eller `rm -fr`.
Vissa vanliga flaggor är i praktiken en standard,
och du ser dem i många program:

* `-a` syftar ofta på alla filer (inklusive de som börjar med punkt)
* `-f` syftar ofta på att tvinga något, som i `rm -f`
* `-h` visar hjälp för de flesta kommandon
* `-v` slår ofta på utförlig utdata
* `-V` skriver oftast ut kommandots version

Ett dubbelt bindestreck `--` används också i inbyggda kommandon och många andra kommandon
för att markera slutet på kommandoalternativ,
varefter endast positionsargument accepteras.
Om du därför har en fil som heter `-v` (det går) och vill köra `grep` på den,
fungerar `grep pattern -- -v` medan `grep pattern -v` inte gör det.
Ett sätt att skapa en sådan fil är faktiskt `touch -- -v`.

## Övningar

1. Om du är helt ny i skalet kan du läsa en mer heltäckande guide,
   till exempel [BashGuide](https://mywiki.wooledge.org/BashGuide).
   Om du vill ha en djupare introduktion är [The Linux Command Line](https://linuxcommand.org/tlcl.php) en bra resurs.

1. **PATH, which, type**

    Vi pratade kort om att miljövariabeln `PATH` används för att hitta programmen
    du kör från kommandoraden.
    Låt oss utforska det lite mer.
    - Kör `echo $PATH` (eller `echo $PATH | tr -s ':' '\n'` för snyggare utskrift) och granska innehållet.
      Vilka sökvägar listas?
    - Kommandot `which` letar upp ett program i användarens PATH.
      Prova `which` för vanliga kommandon som `echo`, `ls` eller `mv`.
      Notera att `which` är lite begränsat eftersom det inte förstår skalalias.
      Testa `type` och `command -v` för samma kommandon.
      Hur skiljer sig utdata?
    - Kör `PATH=` och testa de tidigare kommandona igen.
      Vissa fungerar och vissa inte.
      Kan du lista ut varför?

1. **Specialvariabler**
    - Till vad expanderar `~`?
      Vad sägs om `.`?
      Och `..`?
    - Vad gör variabeln `$?`?
    - Vad gör variabeln `$_`?
    - Till vad expanderar `!!`?
      Och `!!*`?
      Och `!l`?
    - Leta upp dokumentation för dessa och bekanta dig med dem

1. **xargs**

    Ibland fungerar piping inte riktigt,
    eftersom kommandot som tar emot data inte förväntar sig ett radseparerat format.
    Till exempel visar kommandot `file` egenskaper för en fil.

    Kör `ls | file` och `ls | xargs file`.
    Vad gör `xargs`?


1. **Shebang**

    När du skriver ett skript kan du ange vilket program som ska tolka skriptet,
    via en [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))-rad.
    Skriv ett skript som heter `hello` med innehållet nedan,
    gör det körbart med `chmod +x hello`,
    och kör det sedan med `./hello`.
    Ta därefter bort första raden och kör igen.
    Hur använder skalet den första raden?


    ```bash
      #! /usr/bin/python

      print("Hello World!")
    ```

    Du kommer ofta att se program med en shebang som ser ut så här: `#! usr/bin/env bash`.
    Det är en mer portabel lösning med egna [för- och nackdelar](https://unix.stackexchange.com/questions/29608/why-is-it-better-to-use-usr-bin-env-name-instead-of-path-to-name-as-my).
    Hur skiljer sig `env` från `which`?
    Vilken miljövariabel använder `env` för att avgöra vilket program som ska köras?


1. **Pipes, process substitution, subshell**

    Skapa ett skript som heter `slow_seq.sh` med innehållet nedan,
    och kör `chmod +x slow_seq.sh` för att göra det körbart.

    ```bash
      #! /usr/bin/env bash

      for i in $(seq 1 10); do
              echo $i;
              sleep 1;
      done
    ```

    Pipes (och process substitution) skiljer sig från att använda subshell-körning,
    alltså `$()`.
    Kör följande kommandon och observera skillnaderna:

    - `./slow_seq.sh | grep -P "[3-6]"`
    - `grep -P "[3-6]" <(./slow_seq.sh)`
    - `echo $(./slow_seq.sh) | grep -P "[3-6]"`


1. **Övrigt**
    - Kör `touch {a,b}{a,b}` och sedan `ls`.
      Vad dök upp?
    - Ibland vill du behålla STDIN och samtidigt skriva till fil.
      Kör `echo HELLO | tee hello.txt`.
    - Kör `cat hello.txt > hello.txt`.
      Vad tror du händer?
      Vad händer faktiskt?
    - Kör `echo HELLO > hello.txt` och sedan `echo WORLD >> hello.txt`.
      Vad innehåller `hello.txt`?
      Hur skiljer sig `>` från `>>`?
    - Kör `printf "\e[38;5;81mfoo\e[0m\n"`.
      Hur blev utdata annorlunda?
      Om du vill veta mer, sök på ANSI color escape sequences.
    - Kör `touch a.txt` och sedan `^txt^log`.
      Vad gjorde bash åt dig?
      Kör i samma anda `fc`.
      Vad gör det?

{% comment %}

TODO

1. **parallel**
- set -e, set -x
- traps

{% endcomment %}

1. **Kortkommandon**

    Precis som med alla program du använder ofta är det värt att lära sig kortkommandon.
    Skriv in följande och försök förstå vad de gör,
    och i vilka situationer de är praktiska.
    För vissa kan det vara enklast att söka online.
    (Kom ihåg att `^X` betyder `Ctrl+X`.)

    - `^A`, `^E`
    - `^R`
    - `^L`
    - `^C`, `^\` och `^D`
    - `^U` och `^Y`
