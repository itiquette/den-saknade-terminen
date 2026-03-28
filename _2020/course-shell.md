---
layout: lecture
title: "Kursöversikt + skalet"
description: >
  Lär dig varför kursen finns och kom igång med skalet.
thumbnail: /static/assets/thumbnails/2020/lec1.png
date: 2020-01-13
ready: true
video:
  aspect: 56.25
  id: Z56Jmr9Z34Q
---

# Motivation

Som datavetare vet vi att datorer är fantastiska på att hjälpa till med repetitiva uppgifter.
Ofta glömmer vi att det här gäller lika mycket för _hur vi använder_ datorn som för de beräkningar vi vill att våra program ska utföra.
Vi har många verktyg nära till hands som gör oss mer produktiva och låter oss lösa mer komplexa problem i allt datorrelaterat arbete.
Trots det använder många av oss bara en liten del av verktygen; vi kan precis tillräckligt många magiska rader utantill för att klara oss, och kopierar blint kommandon från internet när vi kör fast.

Kursen är ett försök att rätta till det.

Vi vill lära dig att få ut mer av verktygen du redan känner till, visa nya verktyg att lägga i verktygslådan och förhoppningsvis väcka lust att utforska (och kanske bygga) fler verktyg själv.
Det är det vi menar med den saknade terminen i många datavetenskapliga utbildningar.

# Kursupplägg

Kursen består av 11 föreläsningar på en timme vardera, där varje föreläsning kretsar kring ett [särskilt ämne]({{ '/2020/' | relative_url }}).
Föreläsningarna är till stor del fristående, även om vi längre fram under terminen antar att du är bekant med innehållet från tidigare pass.
Vi har föreläsningsanteckningar på nätet, men en hel del av det som tas upp i klassrummet (t.ex. demonstrationer) kanske inte finns i anteckningarna.
Vi spelar in föreläsningarna och publicerar inspelningarna på nätet.

Vi försöker täcka mycket under bara 11 en-timmesföreläsningar, så föreläsningarna är ganska täta.
För att ge dig tid att ta till dig innehållet i din egen takt innehåller varje föreläsning en uppsättning övningar som leder dig genom föreläsningens nyckelpunkter.
Efter varje föreläsning har vi mottagningstid där vi finns på plats för att hjälpa till med frågor.
Om du deltar på distans kan du skicka frågor till [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

På grund av begränsad tid kan vi inte täcka alla verktyg på samma detaljnivå som en fullskalig kurs.
Där det går försöker vi hänvisa dig till resurser för att fördjupa dig i ett verktyg eller ämne, men om något särskilt fångar ditt intresse får du gärna höra av dig och be om tips.

# Ämne 1: Skalet

## Vad är skalet?

Datorer har i dag många gränssnitt för att ge dem kommandon; avancerade grafiska gränssnitt, röstgränssnitt och till och med AR/VR finns överallt.
De är utmärkta för 80 % av användningsfallen, men de är ofta i grunden begränsade i vad de låter dig göra.
Du kan inte trycka på en knapp som inte finns eller ge ett röstkommando som inte har programmerats.
För att fullt ut utnyttja verktygen i din dator behöver vi gå tillbaka till grunderna och använda ett textbaserat gränssnitt: skalet.

Nästan alla plattformar du kan få tag i har ett skal i någon form, och många av dem har flera skal att välja mellan.
Detaljerna skiljer sig, men i grunden är de ungefär lika: de låter dig köra program, ge dem indata och läsa deras utdata på ett semistrukturerat sätt.

I föreläsningen fokuserar vi på Bourne Again SHell, eller "bash".
Det är ett av de mest använda skalen, och dess syntax liknar det du ser i många andra skal.
För att öppna en skalprompt (där du kan skriva kommandon) behöver du först en terminal.
Din enhet har troligen en förinstallerad, annars är den enkel att installera.

## Använda skalet

När du startar terminalen ser du en _prompt_ som ofta ser ut så här:

```console
saknade:~$
```

Det här är skalets viktigaste textgränssnitt.
Det berättar att du är på maskinen `saknade` och att din "nuvarande arbetskatalog", alltså var du befinner dig just nu, är `~` (kort för "home").
`$` visar att du inte är administratörsanvändare (`root`) (mer om det senare).
Vid prompten kan du skriva ett _kommando_ som skalet tolkar.
Det mest grundläggande kommandot är att köra ett program:

```console
saknade:~$ date
fre 10 jan 2020 11:49:31 CET
saknade:~$
```

Här körde vi programmet `date`, som (inte oväntat) skriver ut aktuellt datum och tid.
Skalet ber oss sedan om nästa kommando.
Du kan också köra kommandon med _argument_:

```console
saknade:~$ echo hej
hej
```

Här bad vi skalet köra programmet `echo` med argumentet `hej`.
Programmet `echo` skriver helt enkelt ut sina argument.
Skalet tolkar kommandot genom att dela på blanktecken, kör programmet i första ordet och skickar varje efterföljande ord som ett argument programmet kan läsa.
Om du vill ge ett argument som innehåller blanksteg eller andra specialtecken (t.ex. en katalog med namnet "Mina bilder") kan du antingen citera argumentet med `'` eller `"` (`"Mina bilder"`), eller använda undantagstecken för enskilda tecken med `\` (`Mina\ bilder`).

Men hur vet skalet var program som `date` eller `echo` finns?
Skalet är en programmeringsmiljö, precis som Python eller Ruby, och har därför variabler, villkor, slingor och funktioner (nästa föreläsning!).
När du kör kommandon i skalet skriver du i praktiken små kodsnuttar som skalet tolkar.
Om skalet ombeds köra ett kommando som inte matchar något av dess egna nyckelord tittar det på en _miljövariabel_ som heter `$PATH`, som listar vilka kataloger skalet ska söka i efter program:


```console
saknade:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
saknade:~$ which echo
/bin/echo
saknade:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

När vi kör kommandot `echo` ser skalet att programmet `echo` ska köras och söker sedan i den `:`-separerade listan av kataloger i `$PATH` efter en fil med det namnet.
När det hittar filen kör det den (förutsatt att filen är _körbar_; mer om det senare).
Vi kan ta reda på vilken fil som körs för ett visst programnamn med programmet `which`.
Vi kan också kringgå `$PATH` helt genom att ange _sökvägen_ till filen vi vill köra.

## Navigera i skalet

En sökväg i skalet är en avgränsad lista av kataloger; separerade med `/` i Linux och macOS och `\` i Windows.
I Linux och macOS är sökvägen `/` filsystemets "rot", under vilken alla kataloger och filer finns, medan Windows har en rot per diskpartition (t.ex. `C:\`).
I kursen antar vi i allmänhet att du använder ett Linux-filsystem.
En sökväg som börjar med `/` kallas en _absolut_ sökväg.
Alla andra sökvägar är _relativa_.
Relativa sökvägar är relativa till nuvarande arbetskatalog, som vi kan se med kommandot `pwd` och ändra med kommandot `cd`.
I en sökväg betyder `.` nuvarande katalog och `..` dess föräldrakatalog:

```console
saknade:~$ pwd
/home/saknade
saknade:~$ cd /home
saknade:/home$ pwd
/home
saknade:/home$ cd ..
saknade:/$ pwd
/
saknade:/$ cd ./home
saknade:/home$ pwd
/home
saknade:/home$ cd saknade
saknade:~$ pwd
/home/saknade
saknade:~$ ../../bin/echo hej
hej
```

Notera att skalprompten hela tiden höll oss informerade om vad vår nuvarande arbetskatalog var.
Du kan konfigurera prompten så att den visar många olika sorters användbar information, vilket vi tar upp i en senare föreläsning.

I allmänhet gäller att när vi kör ett program arbetar det i nuvarande katalog om vi inte säger något annat.
Programmet kommer till exempel oftast att leta efter filer där och skapa nya filer där om det behöver.

För att se vad som finns i en viss katalog använder vi kommandot `ls`:

```console
saknade:~$ ls
saknade:~$ cd ..
saknade:/home$ ls
saknade
saknade:/home$ cd ..
saknade:/$ ls
bin
boot
dev
etc
home
...
```

Om ingen katalog anges som första argument skriver `ls` ut innehållet i nuvarande katalog.
De flesta kommandon accepterar flaggor och alternativ (flaggor med värden) som börjar med `-` för att ändra beteendet.
Vanligtvis skriver ett program ut hjälpinformation om tillgängliga flaggor och alternativ om du kör det med `-h` eller `--help`.
Till exempel säger `ls --help`:

```
  -l                         use a long listing format
```

```console
saknade:~$ ls -l /home
drwxr-xr-x 1 saknade  users  4096 Jun 15  2019 saknade
```

Det visar betydligt mer information om varje fil eller katalog som finns.
Först säger `d` i början av raden att `saknade` är en katalog.
Sedan följer tre grupper om tre tecken (`rwx`).
Dessa visar vilka rättigheter ägaren av filen (`saknade`), ägargruppen (`users`) respektive alla andra har på objektet.
Ett `-` betyder att den aktuella parten inte har den aktuella rättigheten.
Ovan är det bara ägaren som får ändra (`w`) katalogen `saknade` (dvs. lägga till/ta bort filer i den).
För att gå in i en katalog måste en användare ha "sök"-rättighet (representerad av "execute": `x`) på den katalogen (och dess föräldrar).
För att lista innehållet måste en användare ha läsrättighet (`r`) på katalogen.
För filer fungerar rättigheterna ungefär som du förväntar dig.
Notera att nästan alla filer i `/bin` har `x`-rättighet för den sista gruppen, "alla andra", så att vem som helst kan köra de programmen.

Några andra praktiska program att känna till här är `mv` (för att byta namn/flytta en fil), `cp` (för att kopiera en fil) och `mkdir` (för att skapa en ny katalog).

Om du någon gång vill ha _mer_ information om ett programs argument, indata, utdata eller hur det fungerar i allmänhet kan du prova programmet `man`.
Det tar ett programnamn som argument och visar dess _manualsida_.
Tryck `q` för att avsluta.

```console
saknade:~$ man ls
```

## Koppla samman program

I skalet har program två huvudsakliga "strömmar" kopplade till sig: en inström och en utström.
När programmet försöker läsa indata läser det från inströmmen, och när det skriver ut något skriver det till utströmmen.
Normalt är ett programs indata och utdata båda terminalen.
Det betyder tangentbordet som indata och skärmen som utdata.
Men vi kan också koppla om dessa strömmar.

Den enklaste formen av omdirigering är `< en_fil` och `> en_fil`.
Dessa låter dig koppla om ett programs in- respektive utström till en fil:

```console
saknade:~$ echo hej > hej.txt
saknade:~$ cat hej.txt
hej
saknade:~$ cat < hej.txt
hej
saknade:~$ cat < hej.txt > hej2.txt
saknade:~$ cat hej2.txt
hej
```

Som visat i exemplet ovan är `cat` ett program som con`cat`enerar filer.
När det får filnamn som argument skriver det ut innehållet i varje fil i följd till utströmmen.
Men när `cat` inte får några argument skriver det i stället innehåll från inströmmen till utströmmen (som i det tredje exemplet ovan).

Du kan också använda `>>` för att lägga till i en fil.
Det här slaget av in-/utdataomdirigering blir särskilt kraftfullt tillsammans med rör (`|`).
Operatorn `|` låter dig "kedja" program så att utdata från ett program blir indata till ett annat:

```console
saknade:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
saknade:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

Vi går in mycket mer i detalj på hur du utnyttjar rör i föreläsningen om datahantering.

## Ett mångsidigt och kraftfullt verktyg

På de flesta Unix-liknande system finns en särskild användare: "root".
Du kan ha sett den i fillistningarna ovan.
Root-användaren står över (nästan) alla åtkomstbegränsningar och kan skapa, läsa, uppdatera och ta bort vilken fil som helst i systemet.
Du loggar normalt inte in som `root`, eftersom det är för lätt att råka förstöra något.
I stället använder du kommandot `sudo`.
Som namnet antyder låter det dig "göra" något "som su" (kort för "superanvändare", alltså `root`).
När du får fel av typen "åtkomst nekad" ("permission denied") beror det ofta på att du behöver göra något som `root`.
Dubbelkolla dock först att det verkligen är vad du vill göra.

En sak som kräver `root` är att skriva till filsystemet `sysfs` som är monterat under `/sys`.
`sysfs` exponerar ett antal kärnparametrar som filer, så att du enkelt kan konfigurera om kärnan i farten utan specialverktyg.
**Observera att sysfs inte finns i Windows eller macOS.**

Till exempel exponeras ljusstyrkan på din bärbara skärm genom en fil som heter `brightness` under

```
/sys/class/backlight
```

Genom att skriva ett värde till den filen kan vi ändra skärmens ljusstyrka.
Din första instinkt kan vara att göra något i stil med:

```console
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

Det här felet kan komma som en överraskning.
Vi körde ju kommandot med `sudo`.
Det är en viktig sak att känna till om skalet.
Operationer som `|`, `>` och `<` utförs _av skalet_, inte av det enskilda programmet.
`echo` och liknande "känner" inte till `|`.
De läser bara från sin indata och skriver till sin utdata, vad de än råkar vara.
I fallet ovan försöker _skalet_ (som är autentiserat som din vanliga användare) öppna brightness-filen för skrivning innan den kopplas som utdata till `sudo echo`.
Det förhindras eftersom skalet inte körs som `root`.
Med den kunskapen kan vi arbeta runt problemet:

```console
$ echo 3 | sudo tee brightness
```

Eftersom programmet `tee` är det som öppnar `/sys`-filen för skrivning, och _det_ körs som `root`, fungerar rättigheterna.
Du kan styra många roliga och användbara saker via `/sys`, till exempel tillståndet för olika system-LED:ar (din sökväg kan skilja sig):

```console
$ echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness
```

# Nästa steg

Nu kan du tillräckligt mycket om skalet för att utföra grundläggande uppgifter.
Du bör kunna navigera för att hitta intressanta filer och använda basfunktioner i de flesta program.
I nästa föreläsning pratar vi om hur man utför och automatiserar mer komplexa uppgifter med skalet och de många praktiska kommandoradsprogram som finns.

# Övningar

Alla pass i kursen har tillhörande övningar.
Vissa är tydliga och konkreta, medan andra är öppnare, som "prova att använda programmen X och Y".
Vi uppmuntrar dig starkt att testa dem.

Vi har inte skrivit facit till övningarna.
Om du fastnar på något särskilt får du gärna e-posta oss och beskriva vad du har provat hittills, så försöker vi hjälpa till.

  1. För kursen behöver du använda ett Unix-skal som Bash eller ZSH.
     Om du använder Linux eller macOS behöver du inte göra något särskilt.
     Om du använder Windows måste du se till att du inte kör cmd.exe eller PowerShell; du kan använda [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) eller en Linux-virtuell maskin för Unix-liknande kommandoradsverktyg.
     För att kontrollera att du kör ett lämpligt skal kan du prova kommandot `echo $SHELL`.
     Om det står något som `/bin/bash` eller `/usr/bin/zsh` betyder det att du kör rätt program.
  1. Skapa en ny katalog med namnet `missing` under `/tmp`.
  1. Slå upp programmet `touch`.
     Programmet `man` är din vän.
  1. Använd `touch` för att skapa en ny fil som heter `semester` i `missing`.
  1. Skriv följande i filen, en rad i taget:
     ```
     #!/bin/sh
     curl --head --silent https://missing.csail.mit.edu
     ```
     Den första raden kan vara lite knepig att få att fungera.
     Det är bra att veta att `#` startar en kommentar i Bash, och att `!` har en specialbetydelse även i dubbelt citerade (`"`) strängar.
     Bash behandlar enkla citationstecken (`'`) annorlunda: de gör jobbet i det här fallet.
     Se Bash-manualsidan om [citering](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) för mer information.
  1. Försök att köra filen, dvs. skriv sökvägen till skriptet (`./semester`) i skalet och tryck Retur/Enter.
     Förstå varför det inte fungerar genom att titta på utdata från `ls` (tips: titta på filens rättighetsbitar).
  1. Kör kommandot genom att uttryckligen starta tolken `sh` och ge den filen `semester` som första argument, dvs. `sh semester`.
     Varför fungerar detta medan `./semester` inte gjorde det?
  1. Slå upp programmet `chmod` (t.ex. med `man chmod`).
  1. Använd `chmod` för att göra det möjligt att köra kommandot `./semester` i stället för att behöva skriva `sh semester`.
     Hur vet skalet att filen ska tolkas med `sh`?
     Se sidan om [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) för mer information.
  1. Använd `|` och `>` för att skriva datumet för "last modified" som `semester` skriver ut till en fil med namnet `last-modified.txt` i din hemkatalog.
  1. Skriv ett kommando som läser ut batterinivån på din bärbara dator eller CPU-temperaturen på din stationära dator från `/sys`.
     Obs: om du använder macOS har ditt operativsystem inte sysfs, så du kan hoppa över övningen.
