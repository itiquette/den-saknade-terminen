---
layout: lecture
title: "Datahantering"
description: >
  Lär dig manipulera och transformera data med kommandoradsverktyg som sed, awk och reguljära uttryck.
thumbnail: /static/assets/thumbnails/2020/lec4.png
date: 2020-01-16
ready: true
video:
  aspect: 56.25
  id: sz_dsktIjt4
special: true
---

Har du någon gång velat ta data i ett format och göra om den till ett annat format?
Självklart har du det.
Det är, mycket generellt uttryckt, vad den här föreläsningen handlar om.
Mer specifikt handlar det om att bearbeta data, oavsett om den är i text- eller binärformat, tills du får exakt det du vill ha.

Vi har redan sett grundläggande datahantering i tidigare föreläsningar.
Nästan varje gång du använder operatorn `|` utför du någon form av datahantering.
Betrakta ett kommando som `journalctl | grep -i intel`.
Det hittar alla poster i systemloggen som nämner Intel (skiftlägesokänsligt).
Du kanske inte tänker på det som datahantering, men det går från ett format (hela systemloggen) till ett format som är mer användbart för dig (bara intel-relaterade loggrader).
Mycket av datahantering handlar om att veta vilka verktyg du har tillgängliga, och hur du kombinerar dem.

Låt oss börja från början.
För att hantera data behöver vi två saker: data att hantera, och något att göra med den.
Loggar är ofta ett bra användningsfall, eftersom man ofta vill undersöka saker i dem, och att läsa allt är inte rimligt.
Låt oss ta reda på vem som försöker logga in på min server genom att titta i serverloggen:

```bash
ssh myserver journalctl
```

Det är alldeles för mycket.
Låt oss begränsa till ssh-relaterat innehåll:

```bash
ssh myserver journalctl | grep sshd
```

Notera att vi använder en pipe för att strömma en _fjärrfil_ genom `grep` på vår lokala dator.
`ssh` är magiskt, och vi pratar mer om det i nästa föreläsning om kommandoradsmiljön.
Det här är fortfarande mycket mer än vi vill ha.
Och ganska svårt att läsa.
Låt oss göra det bättre:

```bash
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```

Varför den extra citeringen?
Våra loggar kan vara stora, och det är slösaktigt att strömma allt till vår dator och filtrera efteråt.
I stället kan vi filtrera på fjärrservern och bearbeta datan lokalt.
`less` ger oss en "pager" som låter oss rulla upp och ned i lång utdata.
För att spara ytterligare trafik medan vi felsöker kommandoraden kan vi till och med lägga den filtrerade loggen i en fil, så att vi slipper nätåtkomst under utvecklingen:

```console
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

Det är fortfarande mycket brus här.
Det finns _många_ sätt att bli av med det, men låt oss titta på ett av de kraftfullaste verktygen i verktygslådan: `sed`.

`sed` är en "stream editor" som bygger på den äldre redigeraren `ed`.
I `sed` ger du i princip korta kommandon för hur filen ska ändras, i stället för att manipulera innehållet direkt (även om du kan göra det också).
Det finns mängder av kommandon, men ett av de vanligaste är `s`: substitution.
Vi kan till exempel skriva:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

Det vi just skrev var ett enkelt _reguljärt uttryck_; en kraftfull konstruktion som låter dig matcha text mot mönster.
Kommandot `s` skrivs i formen `s/REGEX/SUBSTITUTION/`, där `REGEX` är det reguljära uttryck du vill söka efter, och `SUBSTITUTION` är texten du vill ersätta matchningen med.

(Du kanske känner igen syntaxen från avsnittet "Search and replace" i våra Vim-[föreläsningsanteckningar]({{ '/2020/editors/#advanced-vim' | relative_url }}).
Vim använder faktiskt en sök- och ersättningssyntax som liknar `sed`-kommandot för substitution.
Att lära sig ett verktyg hjälper ofta med andra.)

## Reguljära uttryck

Reguljära uttryck är så vanliga och användbara att det är värt att lägga tid på att förstå hur de fungerar.
Låt oss börja med uttrycket vi använde ovan: `/.*Disconnected from /`.
Reguljära uttryck omges ofta (men inte alltid) av `/`.
De flesta ASCII-tecken har sin vanliga betydelse, men vissa tecken har "specialbeteende" vid matchning.
Exakt vilka tecken som gör vad varierar mellan implementationer av reguljära uttryck, vilket ofta är frustrerande.
Mycket vanliga mönster är:

 - `.` betyder "valfritt enskilt tecken" utom radbrytning
 - `*` noll eller fler av föregående matchning
 - `+` en eller fler av föregående matchning
 - `[abc]` valfritt ett tecken av `a`, `b` och `c`
 - `(RX1|RX2)` antingen något som matchar `RX1` eller `RX2`
 - `^` början av raden
 - `$` slutet av raden

`sed`-regex är lite märkliga och kräver ofta att du sätter `\` framför dessa för att ge dem specialbetydelse.
Eller så kan du skicka `-E`.

Tittar vi tillbaka på `/.*Disconnected from /` ser vi att det matchar valfri text som börjar med valfritt antal tecken, följt av den bokstavliga strängen "Disconnected from &rdquo;.
Det var vad vi ville.
Men var försiktig: reguljära uttryck är knepiga.
Vad händer om någon försöker logga in med användarnamnet "Disconnected from"?
Då får vi:

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

Vad får vi då ut?
`*` och `+` är som standard "giriga".
De matchar så mycket text de kan.
I fallet ovan skulle vi alltså bara få:

```
46.97.239.16 port 55920 [preauth]
```

Vilket kanske inte är vad vi ville.
I vissa regeximplementationer kan du lägga till `?` efter `*` eller `+` för att göra dem icke-giriga, men tyvärr stöds det inte av `sed`.
Vi _kan_ dock byta till perls kommandoradsläge, som _stödjer_ det:

```bash
perl -pe 's/.*?Disconnected from //'
```

Vi håller oss till `sed` i resten, eftersom det är det klart vanligaste verktyget för sådana jobb.
`sed` kan också göra andra praktiska saker som att skriva ut rader efter en viss matchning, göra flera substitutioner per körning, söka efter saker och mer.
Men vi går inte in så mycket på det här.
`sed` är i princip ett helt ämne i sig, och ofta finns bättre verktyg.

Okej, vi har också ett suffix vi vill bli av med.
Hur kan vi göra det?
Det är lite knepigt att matcha just texten efter användarnamnet, särskilt om användarnamnet kan innehålla blanksteg och liknande.
Det vi behöver göra är att matcha _hela_ raden:

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

Låt oss titta på vad som händer med en [regex-
felsökare](https://regex101.com/r/qqbZqh/2).
Starten är som tidigare.
Sedan matchar vi någon av varianterna av "user" (det finns två prefix i loggarna).
Därefter matchar vi en godtycklig teckensträng där användarnamnet finns.
Sedan matchar vi ett enskilt ord (`[^ ]+`; en icke-tom sekvens av tecken som inte är blanksteg).
Sedan ordet "port" följt av en sekvens siffror.
Sedan eventuellt suffixet `[preauth]`, och därefter radslut.

Notera att ett användarnamn som "Disconnected from" inte längre förvirrar oss med den här tekniken.
Ser du varför?

Det finns dock ett problem: hela loggraden blir tom.
Vi vill ju _behålla_ användarnamnet.
För det kan vi använda "capture groups".
All text som matchas av regex inom parenteser lagras i en numrerad fångstgrupp.
Dessa finns tillgängliga i substitutionen (och i vissa motorer även i mönstret självt) som `\1`, `\2`, `\3` osv.
Alltså:

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

Som du säkert anar kan man skapa _väldigt_ komplexa reguljära uttryck.
Till exempel finns en artikel om hur man kan matcha en [e-post-
adress](https://www.regular-expressions.info/email.html).
Det är [inte
lätt](https://web.archive.org/web/20221223174323/http://emailregex.com/).
Och det finns [mycket
diskussion](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982).
Och folk har [skrivit
tester](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php).
Och [testmatriser](https://mathiasbynens.be/demo/url-regex).
Du kan till och med skriva ett regex som avgör om ett tal [är ett primtal](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/).

Reguljära uttryck är ökända för att vara svåra att få rätt, men de är också mycket användbara att ha i verktygslådan.

## Tillbaka till datahantering

Okej, så nu har vi

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed` kan göra många andra intressanta saker, som att injicera text (med kommandot `i`), skriva ut rader explicit (med kommandot `p`), välja rader via index och mycket mer.
Kolla `man sed`.

Hur som helst.
Det vi har nu ger en lista över alla användarnamn som har försökt logga in.
Men det är ganska oanvändbart.
Låt oss leta efter vanliga namn:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` sorterar, ja, sin indata.
`uniq -c` slår ihop intilliggande lika rader till en enda rad med ett antal före som visar antal förekomster.
Vi vill sannolikt också sortera detta och bara behålla de vanligaste användarnamnen:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n` sorterar numeriskt (i stället för lexikografiskt).
`-k1,1` betyder "sortera enbart på första blankteckensseparerade kolumnen".
Delen `,n` betyder "sortera till och med det `n`:te fältet, där standard är radslut".
I just _detta_ exempel spelar sortering på hela raden ingen roll, men vi är här för att lära oss.

Om vi ville ha de _minst_ vanliga kunde vi använda `head` i stället för `tail`.
Det finns också `sort -r`, som sorterar i omvänd ordning.

Okej, detta är ganska coolt, men vad om vi vill extrahera bara användarnamnen som en kommaseparerad lista i stället för en per rad, kanske för en konfigurationsfil?

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

Om du använder macOS: notera att kommandot som visas inte fungerar med BSD-`paste` som följer med macOS.
Se [övning 4 i föreläsningen om skalverktyg]({{ '/2020/shell-tools/#exercises' | relative_url }}) för mer om skillnaden mellan BSD och GNU coreutils samt hur du installerar GNU coreutils på macOS.

Låt oss börja med `paste`: det låter dig slå ihop rader (`-s`) med en given avgränsare på ett tecken (`-d`; här `,`).
Men vad är grejen med `awk`?

## awk -- ännu en redigerare

`awk` är ett programmeringsspråk som råkar vara väldigt bra på att bearbeta textströmmar.
Det finns _mycket_ att säga om `awk` om man vill lära sig det ordentligt, men som med mycket annat här går vi igenom grunderna.

Först: vad gör `{print $2}`?
`awk`-program har formen av ett valfritt mönster plus ett block som anger vad som ska göras om mönstret matchar en viss rad.
Standardmönstret (som vi använde ovan) matchar alla rader.
Inuti blocket sätts `$0` till hela radens innehåll, och `$1` till `$n` sätts till radens `n`:te _fält_, separerat av `awk`s fältseparator (blanktecken som standard, ändra med `-F`).
I detta fall säger vi alltså: för varje rad, skriv ut innehållet i andra fältet, vilket råkar vara användarnamnet.

Låt oss se om vi kan göra något mer avancerat.
Låt oss beräkna antalet engångsanvändarnamn som börjar med `c` och slutar med `e`:

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

Det finns mycket att packa upp här.
Först, notera att vi nu har ett mönster (delen före `{...}`).
Mönstret säger att radens första fält ska vara lika med 1 (antalet från `uniq -c`) och att det andra fältet ska matcha det givna reguljära uttrycket.
Blocket säger bara att skriva ut användarnamnet.
Därefter räknar vi antalet rader i utdata med `wc -l`.

Men `awk` är ju ett programmeringsspråk, kom ihåg?

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` är ett mönster som matchar början av indata (och `END` matchar slutet).
Nu adderar blocket per rad bara antalet från första fältet (även om det alltid blir 1 i just detta fall), och sedan skriver vi ut det i slutet.
Faktum är att vi _skulle_ kunna ta bort både `grep` och `sed` helt, eftersom `awk` [kan göra
allt](https://web.archive.org/web/20251210045942/https://backreference.org/2010/02/10/idiomatic-awk/), men vi lämnar det som övning till läsaren.

## Analysera data

Du kan göra matematik direkt i skalet med `bc`, en kalkylator som kan läsa från STDIN.
Till exempel kan du addera talen på varje rad genom att slå samman dem med `+` mellan:

```bash
 | paste -sd+ | bc -l
```

Eller skapa mer avancerade uttryck:

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

Du kan få statistik på olika sätt.
[`st`](https://github.com/nferraz/st) är rätt trevligt, men om du redan har [R](https://www.r-project.org/):

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R är ännu ett (märkligt) programmeringsspråk som är mycket bra för dataanalys och [plotting](https://ggplot2.tidyverse.org/).
Vi går inte in i detalj här, men det räcker att säga att `summary` skriver ut sammanfattande statistik för en vektor, och att vi skapade en vektor med indataflödet av tal, så R ger oss statistiken vi ville ha.

Om du bara vill ha enklare diagram är `gnuplot` din vän:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## Datahantering för att skapa argument

Ibland vill du använda datahantering för att hitta saker att installera eller ta bort utifrån en längre lista.
Datahanteringen vi har pratat om + `xargs` kan vara en mycket kraftfull kombination.

Som i föreläsningen kan jag till exempel använda följande kommando för att avinstallera gamla nightly-builds av Rust från mitt system genom att extrahera gamla buildnamn med datahanteringsverktyg och sedan skicka dem via `xargs` till avinstalleraren:

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## Hantera binärdata

Hittills har vi mest pratat om hantering av textdata, men rör är lika användbara för binärdata.
Vi kan till exempel använda ffmpeg för att ta en bild från kameran, konvertera den till gråskala, komprimera den, skicka den till en fjärrmaskin via SSH, dekomprimera den där, göra en kopia och sedan visa den.

```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

# Övningar

1. Gör den här [korta interaktiva regexhandledningen](https://regexone.com/).
2. Hitta antalet ord (i `/usr/share/dict/words`) som innehåller minst tre `a` och inte slutar på `'s`.
   Vilka är de tre vanligaste sista två bokstäverna i dessa ord?
   `sed`-kommandot `y`, eller programmet `tr`, kan hjälpa med skiftlägesokänslighet.
   Hur många sådana tvåbokstavskombinationer finns det?
   Och som utmaning: vilka kombinationer förekommer inte?
3. För att göra in-place-substitution är det frestande att göra något som `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`.
   Men detta är en dålig idé, varför?
   Gäller detta specifikt för `sed`?
   Använd `man sed` för att ta reda på hur man gör detta korrekt.
4. Hitta medelvärde, median och max för systemets uppstartstid över de senaste tio uppstarterna.
   Använd `journalctl` i Linux och `log show` i macOS, och leta efter tidsstämplar nära början och slutet av varje uppstart.
   I Linux kan de se ut ungefär så här:
   ```
   Logs begin at ...
   ```
   och
   ```
   systemd[577]: Startup finished in ...
   ```
   I macOS, [leta
   efter](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):
   ```
   === system boot:
   ```
   och
   ```
   Previous shutdown cause: 5
   ```
5. Leta efter uppstartsmeddelanden som _inte_ delas mellan dina tre senaste omstarter (se `journalctl`-flaggan `-b`).
   Dela upp uppgiften i flera steg.
   Hitta först ett sätt att få ut bara loggarna från de tre senaste uppstarterna.
   Det kan finnas en lämplig flagga i verktyget du använder för att extrahera uppstartsloggarna, eller så kan du använda `sed '0,/STRING/d'` för att ta bort alla rader före en rad som matchar `STRING`.
   Ta sedan bort delar av raden som _alltid_ varierar (som tidsstämpeln).
   Avdubbla därefter indata och behåll antal för varje rad (`uniq` är din vän).
   Och till sist, eliminera alla rader vars antal är 3 (eftersom de _delades_ av alla uppstarter).
6. Hitta en datamängd på nätet, som [den här](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab), [den här](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1),
   eller kanske en [härifrån](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/).
   Hämta den med `curl` och extrahera bara två kolumner med numeriska data.
   Om du hämtar HTML-data kan [`pup`](https://github.com/EricChiang/pup) vara hjälpsamt.
   För JSON-data, prova [`jq`](https://stedolan.github.io/jq/).
   Hitta min och max i en kolumn i ett enda kommando, och skillnaden mellan summorna av respektive kolumn i ett annat.
