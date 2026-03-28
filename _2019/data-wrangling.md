---
layout: lecture
title: "Datahantering"
presenter: Jon
date: 2019-01-17
order: 2
video:
  aspect: 56.25
  id: VW2jn9Okjhw
---

Har du någonsin haft en hög med text och velat göra något med den?
Bra.
Det är precis vad datahantering handlar om.
Mer specifikt handlar det om att anpassa data från ett format till ett annat, tills du får exakt det du vill ha.

Vi har redan sett grundläggande datahantering: `journalctl | grep -i intel`.
 - hitta alla systemloggrader som nämner Intel (skiftlägesokänsligt)
 - egentligen handlar mycket av datahantering om att veta vilka verktyg du har, och hur du kombinerar dem

Vi börjar från början.
Vi behöver en datakälla och något att göra med den.
Loggar är ofta ett bra användningsfall, eftersom man ofta vill undersöka saker i dem, och att läsa allt är inte rimligt.
Låt oss ta reda på vem som försöker logga in på min server genom att titta i serverloggen:

```bash
ssh myserver journalctl
```

Det är alldeles för mycket.
Låt oss begränsa det till ssh-relaterat innehåll:

```bash
ssh myserver journalctl | grep sshd
```

Notera att vi använder ett rör för att strömma en _fjärrfil_ genom `grep` på vår lokala dator.
`ssh` är magiskt.
Det här är fortfarande mycket mer än vi vill ha, och dessutom svårt att läsa.
Låt oss göra bättre:

```bash
ssh myserver journalctl | grep sshd | grep "Disconnected from"
```

Det finns fortfarande mycket brus här.
Det finns _många_ sätt att minska det, men låt oss titta på ett av de kraftfullaste verktygen i verktygslådan: `sed`.

`sed` är en strömredigerare som bygger vidare på den äldre editorn `ed`.
I `sed` ger du korta kommandon för hur en fil ska ändras, i stället för att manipulera innehållet direkt (även om du också kan göra det).
Det finns massor av kommandon, men ett av de vanligaste är `s`: substitution.
Vi kan till exempel skriva:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

Det vi just skrev var ett enkelt _reguljärt uttryck_, en kraftfull konstruktion som låter dig matcha text mot mönster.
Kommandot `s` skrivs i formen `s/REGEX/SUBSTITUTION/`, där `REGEX` är det reguljära uttryck du vill söka efter, och `SUBSTITUTION` är texten du vill ersätta matchningen med.

## Reguljära uttryck

Reguljära uttryck är så vanliga och användbara att det är värt att lägga tid på att förstå dem.
Låt oss börja med uttrycket ovan: `/.*Disconnected from /`.
Reguljära uttryck omges ofta (men inte alltid) av `/`.
De flesta ASCII-tecken har sin vanliga betydelse, men vissa tecken har särskild matchningsfunktion.
Exakt vilka tecken som betyder vad varierar mellan olika regex-implementationer, vilket är en källa till frustration.
Mycket vanliga mönster är:

 - `.` betyder "vilket enskilt tecken som helst" utom radbrytning
 - `*` noll eller fler av föregående matchning
 - `+` en eller fler av föregående matchning
 - `[abc]` något av tecknen `a`, `b` eller `c`
 - `(RX1|RX2)` antingen något som matchar `RX1` eller `RX2`
 - `^` början av raden
 - `$` slutet av raden

`sed` har lite egen regex-syntax, och kräver ofta `\` framför många av dessa för att ge dem specialbetydelse.
Eller så kan du ange `-E`.

Om vi går tillbaka till `/.*Disconnected from /` ser vi att det matchar valfri text som börjar med valfritt antal tecken, följt av den bokstavliga strängen "Disconnected from ".
Det var vad vi ville.
Men se upp, reguljära uttryck är luriga.
Vad händer om någon försöker logga in med användarnamnet "Disconnected from"?
Då får vi:

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

Vad skulle vi få ut?
`*` och `+` är som standard "giriga".
De matchar så mycket text som möjligt.
Så i fallet ovan skulle vi få:

```
46.97.239.16 port 55920 [preauth]
```

Det kanske inte var det vi ville.
I vissa regex-implementationer kan du lägga till `?` efter `*` eller `+` för att göra dem icke-giriga, men `sed` stöder tyvärr inte det.
Vi _kan_ byta till perls kommandoradsläge, som _stöder_ konstruktionen:

```bash
perl -pe 's/.*?Disconnected from //'
```

Vi håller oss ändå till `sed` i resten av genomgången, eftersom det är klart vanligast för den typen av jobb.
`sed` kan också göra annat användbart, som att skriva ut rader efter en given träff, göra flera substitutioner per körning, söka, och mycket mer.
Vi går dock inte in så djupt här.
`sed` är i princip ett helt ämne i sig, men det finns ofta bättre verktyg.

Okej, vi har också ett suffix vi vill bli av med.
Hur gör vi det?
Det är lite knepigt att matcha just texten efter användarnamnet, särskilt om användarnamnet kan innehålla mellanslag och liknande.
Det vi behöver göra är att matcha _hela_ raden:

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

Låt oss titta på vad som händer med en [regex-felsökare](https://regex101.com/r/qqbZqh/2).
Starten är samma som tidigare.
Sedan matchar vi olika varianter av "user" (två prefix förekommer i loggarna).
Därefter matchar vi vilken teckensträng som helst där användarnamnet finns.
Sedan matchar vi ett enstaka ord (`[^ ]+`, alltså en icke-tom följd av icke-blanktecken).
Sedan ordet "port" följt av siffror.
Sedan eventuellt suffixet ` [preauth]`, och till sist radslut.

Notera att ett användarnamn som "Disconnected from" inte längre förvirrar oss med den här tekniken.
Ser du varför?

Det finns dock ett problem.
Hela loggraden blir tom.
Vi vill ju _behålla_ användarnamnet.
För det kan vi använda "capture groups".
Text som matchas av regex inom parentes lagras i en numrerad fångstgrupp.
Dessa finns tillgängliga i substitutionen (och i vissa motorer även i mönstret självt) som `\1`, `\2`, `\3`, osv. Alltså:

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

Som du kan tänka dig går det att skapa _mycket_ komplicerade regexar.
Här är till exempel en artikel om hur man kan matcha en [e-postadress](https://www.regular-expressions.info/email.html).
Det är [inte enkelt](https://web.archive.org/web/20221223174323/http://emailregex.com/).
Det finns [mycket diskussion](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982).
Folk har [skrivit tester](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php).
Och [testmatriser](https://mathiasbynens.be/demo/url-regex).
Du kan till och med skriva en regex för att avgöra om ett tal [är ett primtal](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/).

Reguljära uttryck är ökända för att vara svåra att få rätt, men de är också väldigt användbara verktyg.

## Tillbaka till datahantering

Okej, nu har vi:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

Vi skulle kunna göra allt med bara `sed`, men varför skulle vi det?
För att det är kul.

```bash
ssh myserver journalctl
 | sed -E
   -e '/Disconnected from/!d'
   -e 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

Det här visar upp några av `sed`:s möjligheter.
`sed` kan också injicera text (med kommandot `i`), skriva ut rader explicit (med `p`), välja rader efter index, och mycket annat.
Se `man sed`.

Hur som helst. Det vi har nu ger en lista över alla användarnamn som försökt logga in.
Men det är inte så hjälpsamt.
Låt oss leta vanliga namn:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` sorterar sin indata.
`uniq -c` slår ihop intilliggande identiska rader till en rad, prefixed med antal förekomster.
Vi vill sannolikt sortera även det, och bara behålla de vanligaste inloggningarna:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n` sorterar numeriskt i stället för lexikografiskt.
`-k1,1` betyder "sortera bara på första blankteckensseparerade kolumnen".
`,n`-delen betyder "sortera fram till fält nummer n", där standard är radslut.
I just _det här_ exemplet spelar det ingen roll om vi sorterar på hela raden, men vi är här för att lära oss.

Om vi i stället vill ha de _minst_ vanliga kan vi använda `head` i stället för `tail`.
Det finns också `sort -r`, som sorterar omvänt.

Okej, det här är ganska coolt, men vi vill kanske bara ha användarnamnen, och kanske inte en per rad.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

Låt oss börja med `paste`.
Det låter dig slå ihop rader (`-s`) med ett valfritt enstaka avgränsningstecken (`-d`).
Men vad är `awk`?

## awk -- ännu en editor

`awk` är ett programmeringsspråk som råkar vara väldigt bra på att bearbeta textströmmar.
Det finns _mycket_ att säga om `awk` om man vill lära sig det ordentligt, men som med mycket annat här går vi igenom grunderna.

Först, vad gör `{print $2}`?
`awk`-program har formen av ett valfritt mönster plus ett block som anger vad som ska göras om mönstret matchar en rad.
Standardmönstret (som vi använde ovan) matchar alla rader.
Inuti blocket är `$0` hela radens innehåll, och `$1` till `$n` är fält `n` i raden, enligt `awk`:s fältavgränsare (blanktecken som standard, kan ändras med `-F`).
Här säger vi att för varje rad skriva ut andra fältet, som råkar vara användarnamnet.

Låt oss göra något mer avancerat.
Låt oss räkna antalet engångsanvända användarnamn som börjar på `c` och slutar på `e`:

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

Det finns mycket att ta in här.
Notera först att vi nu har ett mönster (delen före `{...}`).
Mönstret säger att första fältet ska vara lika med 1 (det är antalet från `uniq -c`), och att andra fältet ska matcha det givna regexet.
Blocket säger bara att skriva ut användarnamnet.
Sedan räknar vi antalet rader i utdata med `wc -l`.

Men `awk` är ju ett programmeringsspråk, kom ihåg.

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` är ett mönster som matchar början av indata (och `END` matchar slutet).
Nu adderar radblocket bara antalet i första fältet (även om det alltid är 1 i just det här fallet), och sedan skriver vi ut resultatet i slutet.
Faktum är att vi _skulle_ kunna ta bort `grep` och `sed` helt, eftersom `awk` [kan göra allt](https://web.archive.org/web/20251210045942/https://backreference.org/2010/02/10/idiomatic-awk/), men det lämnar vi som övning.

## Analysera data

Du kan räkna matematik.

```bash
 | paste -sd+ | bc -l
```

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

Du kan ta fram statistik på olika sätt.
[`st`](https://github.com/nferraz/st) är rätt trevligt, men om du redan har R:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R är ett annat (udda) programmeringsspråk som är utmärkt för dataanalys och [visualisering](https://ggplot2.tidyverse.org/).
Vi går inte in i detalj, men `summary` skriver sammanfattande statistik för en matris, och vi byggde en matris från indataflödet av tal, så R ger oss statistiken vi ville ha.

Om du bara vill ha enkla diagram är `gnuplot` en bra vän:

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

Ibland vill du använda datahantering för att hitta saker att installera eller ta bort, baserat på en längre lista.
Datahanteringen vi har pratat om, tillsammans med `xargs`, är en kraftfull kombination:

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86|01-17" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

# Övningar

1. Om du inte är bekant med reguljära uttryck finns [här](https://regexone.com/) en kort interaktiv handledning som täcker de flesta grunder.
1. Hur skiljer sig `sed s/REGEX/SUBSTITUTION/g` från vanlig `sed`?
   Vad gäller `/I` eller `/m`?
1. För in-place-substitution är det frestande att göra något i stil med `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`.
   Det är dock en dålig idé.
   Varför?
   Är detta specifikt för `sed`?
1. Implementera ett enkelt grep-liknande verktyg i ett språk du kan, med regex. Om du vill färgmarkera utdata som `grep`, sök efter ANSI-färgkontrollsekvenser.
1. Vissa operationer, som att byta namn på filer, kan vara svåra med råa kommandon som `mv`.
   `rename` är ett smidigt verktyg för detta med sed-liknande syntax.
   Skapa några filer med mellanslag i namnen och använd `rename` för att ersätta dem med understreck.
1. Leta efter boot-meddelanden som _inte_ delas mellan dina tre senaste omstarter (se `journalctl`-flaggan `-b`).
   Du kan vilja slå ihop bootloggarna till en enda fil, eftersom det kan göra uppgiften enklare.
1. Ta fram statistik över ditt systems boot-tid under de senaste tio omstarterna med tidsstämplarna i loggmeddelandena
   ```
   Logs begin at ...
   ```
   och
   ```
   systemd[577]: Startup finished in ...
   ```
1. Hitta antalet ord (i `/usr/share/dict/words`) som innehåller minst tre `a` och inte slutar på `'s`.
   Vilka är de tre vanligaste tvåbokstavssluten i dessa ord?
   `sed`:s `y`-kommando eller programmet `tr` kan hjälpa med skiftlägesokänslighet.
   Hur många sådana tvåbokstavskombinationer finns det?
   Och som utmaning: vilka kombinationer förekommer inte?
1. Hitta en online-datamängd, som [den här](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab) eller [den här](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1).
   Kanske en annan [härifrån](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/).
   Hämta den med `curl` och extrahera endast två numeriska kolumner.
   Om du hämtar HTML-data kan [`pup`](https://github.com/EricChiang/pup) vara användbart.
   För JSON-data, prova [`jq`](https://stedolan.github.io/jq/).
   Hitta min och max i en kolumn i ett enda kommando, och summan av skillnaden mellan de två kolumnerna i ett annat.
