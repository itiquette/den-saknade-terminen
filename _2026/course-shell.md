---
layout: lecture
title: "Kursöversikt + introduktion till skalet"
description: >
  Lär dig varför kursen finns och kom igång med skalet.
thumbnail: /static/assets/thumbnails/2026/lec1.png
date: 2026-01-12
ready: true
video:
  aspect: 56.25
  id: MSgoeuMqUmU
---

# Vilka är vi?

Kursen undervisas gemensamt av [Anish](https://anish.io/), [Jon](https://thesquareplanet.com/) och [Jose](http://josejg.com/).
Vi är alla tidigare MIT-studenter som startade den här MIT IAP-kursen när vi själva studerade.
Du kan nå oss gemensamt på [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

Vi får inte betalt för att undervisa kursen och vi tjänar inte pengar på den på något sätt.
Vi gör allt [kursmaterial (här länk till engelska kursoriginalsidan)](https://missing.csail.mit.edu/) och alla [inspelningar av föreläsningarna](https://www.youtube.com/@MissingSemester) fritt tillgängliga på nätet.
Vill du stötta vårt arbete så är det bästa sättet att berätta för andra om kursen.
Representerar du ett företag, universitet eller annan organisation som använder innehållet för större grupper får du gärna e-posta dina erfarenheter eller omdömen så att det når oss :)

# Motivation

Som datavetare vet vi att datorer är fantastiska på att hjälpa till med repetitiva uppgifter.
Alltför ofta glömmer vi dock att detta gäller lika mycket för _hur vi använder_ datorn som för de beräkningar vi vill att våra program ska utföra.
Det finns en uppsjö av verktyg tillgängliga som gör oss produktivare och låter oss lösa mer komplexa problem i allt datorrelaterat arbete.
Trots det använder många av oss bara en liten del av verktygen.
Vi kan precis tillräckligt många magiska rader utantill för att komma vidare och kopierar blint kommandon från internet när vi kör fast.

Kursen är ett försök att [åtgärda det]({{ '/about/' | relative_url }}).

Vi vill lära dig att få ut mer av verktygen du redan känner till, visa nya verktyg att lägga i verktygslådan, och förhoppningsvis väcka din nyfikenhet i att utforska (och kanske bygga) flera verktyg själv.
Vi tror att det här är den saknade terminen i de flesta datavetenskapliga utbildningar.

# Kursupplägg

Den poängfria kursen består av nio föreläsningar på en timme vardera, där varje föreläsning fokuserar på ett [särskilt ämne]({{ '/2026/' | relative_url }}).
Föreläsningarna är till stor del fristående, men allt efter terminen fortgår, antar vi att du kan innehållet från tidigare pass.
Vi har föreläsningsanteckningar på nätet, men en del innehåll som tas upp på en föreläsning (t.ex. demos) kanske inte finns i anteckningarna.
Precis som för tidigare år spelar vi in föreläsningarna och publicerar dem [på nätet](https://www.youtube.com/@MissingSemester).

Vi försöker att täcka mycket på bara några få en-timmespass, vilket innebär att föreläsningarna är ganska informationstunga.
För att du ska kunna ta till dig innehållet i din egen takt innehåller varje föreläsning en mängd övningar som vägleder dig genom huvudpunkterna.
Vi kommer inte att ha särskilda mottagningstider, men vi uppmuntrar dig att ställa frågor på [OSSU Discord](https://ossu.dev/#community), i `#missing-semester-forum`, eller via e-post till [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

På grund av tidsbrist går det inte täcka alla verktyg på samma detaljnivå som en fullskalig kurs.
Vi kommer att försöka hänvisa dig till fördjupande resurser i ett verktyg eller ämne när det går, men om något verkligen fångar ditt intresse, tveka inte på höra av dig och be om tips.

Har du synpunkter på kursen får du gärna e-posta dem till [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

# Ämne 1: Skalet

{% comment %}
lecturer: Jon
{% endcomment %}

## Vad är skalet?

Datorer har i dag många gränssnitt för att ge dem kommandon: avancerade grafiska gränssnitt, röstgränssnitt, AR/VR, och på senare tid: LLM:er.
De är utmärkta i 80 % av användningsfallen, men de är ofta i grunden begränsade i vad de låter dig göra.
Du kan inte trycka på en knapp som inte finns, eller ge ett röstkommando som ingen har programmerat.
För att fullt ut utnyttja verktygen i din dator behöver vi gå tillbaka till grunderna och använda ett textbaserat gränssnitt: skalet.

Nästan alla plattformar du kan få tag i har ett skal i någon form, och många har flera skal att välja mellan.
Detaljerna skiljer sig, men i grunden är de ungefär lika: de låter dig köra program, ge dem indata och läsa deras utdata på ett semistrukturerat sätt.

För att öppna en skalprompt (där du kan skriva kommandon) behöver du först en terminal, som är det visuella gränssnittet till ett skal.
Din enhet har troligen ett skal installerat, annars är den enkel att installera:

- **Linux:** Tryck `Ctrl + Alt + T` (fungerar i de flesta distributioner).
  Eller sök efter "Terminal" i programmenyn.
- **Windows:** Tryck `Win + R`, skriv `cmd` eller `powershell` och tryck Retur/Enter.
  Du kan också söka efter "Terminal" eller "Kommandotolken" i startmenyn.
- **macOS:** Tryck `Cmd + Space` för att öppna Spotlight, skriv "Terminal" och tryck Retur/Enter.
  Eller hitta den i Applikationer -> Verktyg -> Terminal.

På Linux och macOS öppnas oftast Bourne Again SHell, eller "bash".
Det är ett av de mest använda skalen, och dess syntax liknar det du ser i många andra skal.
På Windows möts du av "batch" eller "powershell" beroende på vilket kommando du körde.
De är Windows-specifika och vi fokuserar inte på dem i kursen, även om de har motsvarigheter till det mesta vi lär ut.
Du vill i stället använda [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) eller en Linux-virtuell maskin.

Andra skal finns också, ofta med många ergonomiska förbättringar jämfört med bash (fish och zsh är vanliga exempel).
Även om de är populära (alla lärare använder något av dem), är de inte alls lika allmänt förekommande som bash, och bygger på samma grundkoncept, så vi fokuserar inte på dem i föreläsningen.

## Varför ska du bry dig?

Skalet är inte bara (oftast) betydligt snabbare än att "klicka runt".
Det ger också en uttryckskraft som du sällan hittar i ett enskilt grafiskt program.
Som vi ska se ger skalet dig möjlighet att _kombinera_ program på kreativa sätt för att automatisera nästan vilken uppgift som helst.

Att kunna skalet är också väldigt nyttigt när du navigerar i världen av fri programvara och öppen källkod, där installationsinstruktioner ofta kräver skalet, när du bygger kontinuerlig integration för projekt (som i [föreläsningen om kodkvalitet]({{ '/2026/code-quality/' | relative_url }})), och när du felsöker programfel när andra program kraschar.

## Navigera i skalet

När du startar terminalen ser du en _prompt_ som ofta liknar detta:

```console
saknade:~$
```

Det här är skalets huvudsakliga textgränssnitt.
Det berättar att du är på maskinen `saknade` och att din "nuvarande arbetskatalog" är `~` (kort för "home").
`$` visar att du inte är root-användare (mer om det senare).
Vid prompten kan du skriva ett _kommando_ som skalet tolkar.
Det mest grundläggande kommandot är att köra ett program:

```console
saknade:~$ date
fre 10 jan 2020 11:49:31 CET
saknade:~$
```

Här körde vi programmet `date`, som (inte oväntat) skriver ut aktuellt datum och tid.
Sedan ber skalet oss om nästa kommando.
Vi kan också köra kommandon med _argument_:

```console
saknade:~$ echo hej
hej
```

Här bad vi skalet köra programmet `echo` med argumentet `hej`.
`echo` skriver helt enkelt ut sina argument.
Skalet tolkar kommandot genom att dela på blanktecken, kör programmet i första ordet, och skickar efterföljande ord som argument programmet kan läsa.
Om du vill ge ett argument som innehåller blanksteg eller specialtecken (t.ex. en katalog med namnet "Mina bilder") kan du citera med `'` eller `"` (`"Mina bilder"`), eller använda undantagstecken för enskilda tecken med `\` (`Mina\ bilder`).

Det viktigaste kommandot i början är kanske `man`, kort för "manual".
`man` låter dig slå upp information om kommandon på ditt system.
Om du till exempel kör `man date` får du en beskrivning av `date` och alla argument som ändrar beteendet.
Du kan ofta också visa en kortare hjälpsida genom att ange `--help` till kommandot.

> Överväg att installera och använda [`tldr`](https://tldr.sh/) utöver `man`, eftersom det visar vanliga exempel direkt i terminalen.
> LLM:er är också ofta väldigt bra på att förklara hur kommandon fungerar och hur de kan anropas.

Det viktigaste kommandot efter `man` är `cd` ("change directory").
Det är inbyggt i skalet och inte ett separat program (dvs. `which cd` säger "no cd found").
Du skickar in en sökväg, och den blir din nuvarande arbetskatalog.
Du ser det också i prompten:

```console
saknade:~$ cd /bin
saknade:/bin$ cd /
saknade:/$ cd ~
saknade:~$
```

> Skalet har autokomplettering,
> så du kan ofta komplettera sökvägar snabbare med `<TAB>`.

Många kommandon arbetar i nuvarande arbetskatalog om inget annat anges.
Om du är osäker på var du är kan du köra `pwd` eller skriva ut miljövariabeln `$PWD` med `echo $PWD`.
Båda visar nuvarande arbetskatalog.

Nuvarande arbetskatalog är också viktig eftersom den låter oss använda _relativa_ sökvägar.
Alla sökvägar vi sett hittills har varit _absoluta_ --- de börjar med `/` och anger hela vägen från filsystemets rot (`/`).
I praktiken jobbar du oftare med relativa sökvägar, som är relativa till nuvarande arbetskatalog.
I en relativ sökväg (allt som _inte_ börjar med `/`) används sökvägen med början från nuvarande katalog.
Till exempel:

```console
saknade:~$ cd /
saknade:/$ cd bin
saknade:/bin$
```

Det finns också två "specialkomponenter" i varje katalog: `.` och `..`.
`.` är "den här katalogen" och `..` är "föräldrakatalogen".
Alltså:

```console
saknade:~$ cd /
saknade:/$ cd bin/../bin/../bin/././../bin/..
saknade:/$
```

Du kan oftast växla mellan att använda absoluta och relativa sökvägar.
Tänk dock på vad din aktuella arbetskatalog är när du använder relativa.

> Överväg att installera och använda [`zoxide`](https://github.com/ajeetdsouza/zoxide) för snabbare `cd` --- `z` minns sökvägar du ofta besöker.

## Vad finns tillgängligt i skalet?

Hur vet skalet var program som `date` och `echo` finns?
Om skalet ska köra ett kommando tittar det på en _miljövariabel_ kallad `$PATH`.
Den listar de kataloger som skalet ska söka efter program i:

```console
saknade:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
saknade:~$ which echo
/bin/echo
saknade:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

När vi kör `echo` ser skalet att programmet `echo` ska köras, och söker i den `:`-separerade listan i `$PATH` efter en fil med det namnet.
När den hittas körs den (om filen är _körbar_, mer om det senare).
Vi kan se exakt vilken fil som körs för ett namn med `which`.
Vi kan också kringgå `$PATH` helt genom att ange full _sökväg_ till filen.

Det visar också hur vi kan ta reda på _alla_ program vi kan köra: lista innehållet i alla kataloger i `$PATH`.
Det gör vi genom att ge en katalog till `ls`, som listar filer:

```console
saknade:~$ ls /bin
```

> Överväg att installera och använda [`eza`](https://eza.rocks/) för en mer användarvänlig `ls`.

Det skriver på de flesta datorer ut _väldigt_ många program, men här fokuserar vi på några av de viktigaste.
Några enkla kommandon:

- `cat en_fil`, skriver ut innehållet i `en_fil`.
- `sort en_fil`, skriver ut raderna i `en_fil` sorterade.
- `uniq en_fil`, tar bort intilliggande dubblettrader i `en_fil`.
- `head en_fil` och `tail en_fil`, visar de första respektive sista raderna.

> Överväg att installera och använda [`bat`](https://github.com/sharkdp/bat) i stället för `cat` för syntaxfärgning och skrollning.

Det finns också `grep mönster en_fil`, som hittar rader som matchar `mönster` i `en_fil`.
Det kommandot förtjänar lite extra uppmärksamhet eftersom det är _mycket_ användbart och mer kraftfullt än det ser ut.
`mönster` är ett _reguljärt uttryck_ som kan beskriva komplexa mönster.
Vi går igenom dem i [föreläsningen om kodkvalitet]({{ '/2026/code-quality/#regular-expressions' | relative_url }}).
Du kan också ange en katalog i stället för en fil (eller utelämna för `.`) och skicka `-r` för rekursiv sökning.

> Överväg att installera och använda [`ripgrep`](https://github.com/BurntSushi/ripgrep) i stället för `grep` för ett snabbare och mer användarvänligt (men mindre portabelt) alternativ.
> `ripgrep` söker som standard också rekursivt i aktuell arbetskatalog.

Det finns även väldigt nyttiga verktyg med aningen mer avancerat gränssnitt.
Ett är `sed`, som är en programmerbar filredigerare.
Den har ett eget språk för automatiserade redigeringar, men den vanligaste användningen är:

```console
saknade:~$ sed -i 's/mönster/ersättning/g' en_fil 
```

Det ersätter alla förekomster av `mönster` med `ersättning` i `en_fil`.
`-i` betyder att ersättningen sker i filen (i stället för att skriva ut det ändrade innehållet till standard ut).
`s/` uttrycker i sed-språket att vi vill göra en substitution.
`/` separerar mönster från ersättning.
Avslutande `/g` betyder att vi ersätter _alla_ förekomster per rad, inte bara den första.
Som i `grep` är `mönster` ett reguljärt uttryck, vilket ger en väldigt god uttryckskraft.
Reguljära uttryck låter också `ersättning` hänvisa bakåt i matchningen; vi ska strax visa ett exempel.

Sedan har vi `find`, som låter dig hitta filer rekursivt som matchar vissa villkor.
Till exempel:

```console
saknade:~$ find ~/Hämtningar -type f -name "*.zip" -mtime +30
```

Hittar ZIP-filer i nedladdningskatalogen som är äldre än 30 dagar.

```console
saknade:~$ find ~ -type f -size +100M -exec ls -lh {} \;
```

Hittar filer större än 100M i hemkatalogen och listar dem.
Observera att `-exec` tar ett _kommando_ som avslutas med ett fristående `;` (där du måste använda undantagstecken), där `{}` ersätts av varje matchande sökväg.

```console
saknade:~$ find . -name "*.py" -exec grep -l "TODO" {} \;
```

Hittar alla `.py`-filer med TODO-poster.

`find`-syntaxen kan kännas avskräckande, men förhoppningsvis ger exemplen en bild av hur användbart det är.

> Överväg att installera och använda [`fd`](https://github.com/sharkdp/fd) i stället för `find` för en mer användarvänlig (men mindre portabel) upplevelse.

Nästa verktyg är `awk`, som likt `sed` har ett eget språk.
Där `sed` är byggt för att redigera filer är `awk` byggt för att tolka dem.
Den vanligaste användningen är datafiler med regelbunden struktur (som CSV-filer) där du vill extrahera vissa delar av varje post (dvs. rad):

```console
saknade:~$ awk '{print $2}' en_fil
```

Skriver ut andra blankteckensseparerade kolumnen i varje rad av `en_fil`.
Om du lägger till `-F,` skrivs i stället andra kommaseparerade kolumnen ut.
`awk` kan mycket mer än så; filtrera rader, beräkna aggregat med mera. Se övningarna för fler exempel.

Genom att kombinera de här verktygen kan vi göra avancerade saker som:

```console
saknade:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

Det här hämtar SSH-loggar från en fjärrserver, söker efter frånkopplingsmeddelanden, extraherar användarnamn, och skriver ut de 10 vanligaste kommaseparerat.
Allt i ett kommando. Vi utelämnar analysen av varje steg som en övning.

## Skalspråket (bash)

Föregående exempel introducerade ett nytt koncept: rör (`|`).
De låter dig kedja utdata från ett program till indata för ett annat.
Det fungerar eftersom de flesta kommandoradsprogram läser från "standard in" om inget `fil`-argument ges.
`|` tar "standard ut" från programmet före `|` och gör det till standard in för programmet efter `|`.
Det låter dig _komponera_ skalprogram, och är en stor anledning till varför skalet är så produktivt.

De flesta skal implementerar faktiskt ett komplett programmeringsspråk (som bash), precis som Python eller Ruby.
Det har variabler, villkor, slingor och funktioner.
När du kör kommandon skriver du i praktiken små kodsnuttar som skalet sedan tolkar.
Vi lär inte ut hela bash i dag, men några delar är särskilt användbara:

Först, omdirigeringar: `>en_fil` tar standard ut från ett program och skriver till `en_fil` i stället för terminalen.
`>>en_fil` lägger till i `en_fil` i stället för att skriva över.
`<en_fil` säger åt skalet att läsa standard in från filen i stället för tangentbordet.

> Detta är ett bra tillfälle att nämna `tee`.
> `tee` skriver standard in till standard ut (som `cat`), men _också_ till filen.
> `verbose cmd | tee verbose.log | grep CRITICAL` bevarar alltså hela den utförliga (verbose) loggen i filen samtidigt som terminalen hålls ren.

Sedan villkor: `if kommando1; then kommando2; kommando3; fi` kör `kommando1`, och om det lyckas körs `kommando2` och `kommando3`.
Du kan också ha en `else`-gren.
Vanligast är att använda `test` som `kommando1`, ofta förkortat `[`, för villkor som "finns filen" (`test -f en_fil` / `[ -f en_fil ]`) eller "är strängen lika med" (`[ "$var" = "string" ]`).
I bash finns också `[[ ]]`, en säkrare inbyggd variant med färre konstigheter kring citering.

Bash har två slingtyper, `while` och `for`.
`while kommando1; do kommando2; kommando3; done` fungerar som motsvarande `if`, men upprepas så länge `kommando1` inte ger fel.
`for variabelnamn in a b c d; do kommando; done` kör `kommando` fyra gånger, med `$variabelnamn` satt till `a`, `b`, `c` respektive `d`.
I stället för att lista element explicit används ofta kommandosubstitution (_command substitution_), som:

```bash
for i in $(seq 1 10); do
```

Det kör `seq 1 10` (som skriver ut 1 till 10) och ersätter hela `$()` med kommandots utdata.
Du får då en for-slinga med 10 iterationer.
I äldre kod ser du ibland bakåtcitattecken i stället (``for i in `seq 1 10`; do``), men föredra `$()` eftersom den kan nästlas.

Även om du _kan_ skriva långa skalskript direkt i prompten, vill du oftast lägga dem i en `.sh`-fil.
Här är ett skript som kör ett program i en slinga tills det misslyckas, skriver ut en logg från den felande körningen, och belastar CPU i bakgrunden (nyttigt för att återskapa instabila tester):

```bash
#!/bin/bash
set -euo pipefail

# Starta CPU-belastning i bakgrunden
stress --cpu 8 &
STRESS_PID=$!

# Sätt upp loggfil
LOGFILE="test_runs_$(date +%s).log"
echo "Loggar till $LOGFILE"

# Kör tester tills ett misslyckas
RUN=1
while cargo test my_test > "$LOGFILE" 2>&1; do
    echo "Körning $RUN lyckades"
    ((RUN++))
done

# Städa upp och rapportera
kill $STRESS_PID
echo "Testet misslyckades under körning $RUN"
echo "Sista 20 raderna i utdata:"
tail -n 20 "$LOGFILE"
echo "Fullständig logg: $LOGFILE"
```

Det här innehåller flera nya saker värda att gräva i, som bakgrundsjobb (`&`) för samtidighet, mer avancerade [skalomdirigeringar](https://www.gnu.org/software/bash/manual/html_node/Redirections.html), och [aritmetisk expansion](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html).

Det är också värt att titta på programmets två första rader.
Första raden är "shebang".
Den syns i många filtyper, inte bara skalskript.
När en fil som börjar med `#!/path` körs startar skalet programmet på `/path` och skickar filens innehåll som indata.
För skalskript innebär det att innehållet skickas till `/bin/bash`.
Du kan på samma sätt skriva Python-skript med shebang `/usr/bin/python`.

Andra raden gör bash "striktare" och minskar vanliga fallgropar.
`set` tar många argument, men kort: `-e` avslutar skriptet när ett kommando misslyckas, `-u` gör att odefinierade variabler blir fel i stället för tom sträng, och `-o pipefail` gör att fel i en `|`-kedja också får hela skriptet att avslutas.

> Skalprogrammering är ett djupt ämne, och en varning är befogad:
> bash har ovanligt många fallgropar,
> till den grad att det finns [flera](https://tldp.org/LDP/abs/html/gotchas.html) webbplatser som [listar dem](https://mywiki.wooledge.org/BashPitfalls).
> Vi rekommenderar starkt [shellcheck](https://www.shellcheck.net/) när du skriver skalskript.
> LLM:er är också bra på att skriva och felsöka skalskript,
> och på att översätta dem till ett "riktigt" språk (som Python) när de blir för stora för bash.

# Nästa steg

Nu kan du tillräckligt mycket om skalet för att utföra grundläggande uppgifter.
Du bör kunna navigera, hitta intressanta filer och använda basfunktionerna i de flesta program.
I nästa föreläsning pratar vi om hur du utför och automatiserar mer komplexa uppgifter med skalet samt om de användbara kommandoradsprogrammen som finns därute.

# Övningar

Alla pass i kursen har tillhörande övningar.
Vissa är tydliga och konkreta, andra är friare, som "testa att använda X och Y".
Vi uppmuntrar dig starkt att prova dem.

Vi har inte skrivit facit till övningarna.
Om du fastnar får du gärna skriva i `#missing-semester-forum` på [Discord](https://ossu.dev/#community) eller e-posta oss och beskriva vad du testat.
De här övningarna fungerar också bra som startprompter i en LLM-konversation där du interaktivt kan utforska ett ämne.
Det verkliga värdet är vägen till svaret, inte själva svaret.
Så, följ gärna sidospår och fråga dig "varför" när du jobbar dig igenom dem.

1. För kursen behöver du använda ett Unix-skal som Bash eller ZSH.
   Om du använder Linux eller macOS behöver du inte göra något särskilt.
   Om du använder Windows måste du se till att du inte kör cmd.exe eller PowerShell.
   Du kan använda [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) eller en Linux-VM för Unix-liknande kommandoradsverktyg.
   För att kontrollera att du kör rätt skal kan du testa `echo $SHELL`.
   Om det står något som `/bin/bash` eller `/usr/bin/zsh` kör du rätt program.

2. Vad gör flaggan `-l` i `ls`?
   Kör `ls -l /` och granska utdata.
   Vad betyder de första 10 tecknen på varje rad?
   (Tips: `man ls`.)

3. I kommandot `find ~/Hämtningar -type f -name "*.zip" -mtime +30` är `*.zip` ett jokerteckenmönster (glob).
   Vad är mönstermatchning?
   Skapa en testkatalog med några filer och experimentera med `ls *.txt`, `ls fil?.txt` och `ls {a,b,c}.txt`.
   Se [Mönstermatchning](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html) i Bash-manualen.

4. Vad är skillnaden mellan `'single quotes'`, `"double quotes"` och `$'ANSI quotes'`?
   Skriv ett kommando som ekar en sträng med en bokstavlig `$`, en `!` och en radbrytning.
   Se [citering](https://www.gnu.org/software/bash/manual/html_node/Quoting.html).

5. Skalet har tre standardströmmar: standard in (stdin) (0), standard ut (stdout) (1) och standard fel (stderr) (2).
   Kör `ls /nonexistent /tmp` och omdirigera stdout till en fil och stderr till en annan.
   Hur omdirigerar du båda till samma fil?
   Se [omdirigeringar](https://www.gnu.org/software/bash/manual/html_node/Redirections.html).

6. `$?` innehåller slutstatus för senaste kommandot (0 = lyckat).
   `&&` kör nästa kommando bara om föregående lyckas och `||` kör nästa bara om föregående misslyckas.
   Skriv ett enradskommando som skapar `/tmp/mydir` bara om den inte redan finns.
   Se [slutstatus](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html).

7. Varför måste `cd` vara inbyggt i skalet i stället för ett fristående program?
   (Tips: fundera på vad en barnprocess kan och inte kan påverka i sin förälder.)

8. Skriv ett skript som tar ett filnamn som argument (`$1`) och kontrollerar om filen finns med `test -f` eller `[ -f ... ]`.
   Skriptet ska skriva olika meddelanden beroende på om filen finns.
   Se [villkorsuttryck i Bash](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html).

9. Spara skriptet från förra övningen till en fil (t.ex. `check.sh`).
   Testa `./check.sh somefile`.
   Vad händer?
   Kör sedan `chmod +x check.sh` och testa igen.
   Varför är steget nödvändigt?
   (Tips: jämför `ls -l check.sh` före och efter `chmod`.)

10. Vad händer om du lägger till `-x` till `set`-flaggorna i ett skript?
   Prova med ett enkelt skript och observera utdata.
   Se [det inbyggda kommandot set](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html).

11. Skriv ett kommando som kopierar en fil till en säkerhetskopia med dagens datum i filnamnet (t.ex. `anteckning.txt` -> `anteckning_2026-01-12.txt`).
   (Tips: `$(date +%Y-%m-%d)`.)
   Se [kommandosubstitution (command substitution)](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html).

12. Modifiera skriptet för instabila tester från föreläsningen så att testkommandot tas som argument i stället för ett hårdkodat `cargo test my_test`.
   (Tips: `$1` eller `$@`.)
   Se [särskilda parametrar](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html).

13. Använd rör (`|`) för att hitta de 5 vanligaste filändelserna i din hemkatalog.
   (Tips: kombinera `find`, `grep` eller `sed` eller `awk`, `sort`, `uniq -c` och `head`.)

14. `xargs` gör om rader från stdin till kommandoradsargument.
   Använd `find` och `xargs` tillsammans (inte `find -exec`) för att hitta alla `.sh`-filer i en katalog och räkna rader i varje med `wc -l`.
   Bonus: få det att fungera med filnamn som innehåller blanksteg.
   (Tips: `-print0` och `-0`.)
   Se `man xargs`.

15. Använd `curl` för att hämta HTML för ursprungliga kurswebbplatsen (`https://missing.csail.mit.edu/`) och skicka resultatet genom ett rör till `grep` för att räkna hur många föreläsningar som listas.
   (Tips: leta efter ett mönster som finns en gång per föreläsning och använd `curl -s` för tyst läge.)

16. [`jq`](https://jqlang.github.io/jq/) är ett kraftfullt verktyg för att bearbeta JSON.
   Hämta exempeldata på `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json` med `curl` och använd `jq` för att extrahera namnen på personer vars version är större än 6. (Tips: skicka först genom ett rör till `jq .` för att se strukturen och testa sedan `jq '.[] | select(...) | .name'`.)

17. `awk` kan filtrera rader baserat på kolumnvärden och manipulera utdata.
   Exempel: `awk '$3 ~ /pattern/ {$4=""; print}'` skriver bara rader där tredje kolumnen matchar `pattern`, men utelämnar fjärde kolumnen.
   Skriv ett `awk`-kommando som bara skriver rader där andra kolumnen är större än 100, och byter plats på första och tredje kolumnen.
   Testa med: `printf 'a 50 x\nb 150 y\nc 200 z\n'`.

18. Analysera SSH-loggkedjan från föreläsningen: vad sker i varje steg?
   Bygg sedan något liknande för att hitta dina mest använda skalkommandon från `~/.bash_history` (eller `~/.zsh_history`).
