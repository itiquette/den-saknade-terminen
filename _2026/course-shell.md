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

Den här kursen undervisas gemensamt av [Anish](https://anish.io/),
[Jon](https://thesquareplanet.com/) och [Jose](http://josejg.com/).
Vi är alla tidigare MIT-studenter som startade den här MIT IAP-kursen när vi själva studerade.
Du kan nå oss gemensamt på [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

Vi får inte betalt för att undervisa kursen och vi tjänar inte pengar på den på något sätt.
Vi gör allt [kursmaterial](https://missing.csail.mit.edu/) och [inspelningar av föreläsningarna](https://www.youtube.com/@MissingSemester) fritt tillgängliga på nätet.
Om du vill stötta vårt arbete är bästa sättet att sprida ordet om kursen.
Om du representerar ett företag, universitet eller annan organisation som kör innehållet för större grupper får du gärna e-posta erfarenheter eller omdömen så vi får höra om det.

# Motivation

Som datavetare vet vi att datorer är fantastiska på att hjälpa till med repetitiva uppgifter.
Alltför ofta glömmer vi dock att detta gäller lika mycket för _hur vi använder_ datorn som för de beräkningar vi vill att våra program ska utföra.
Vi har en stor uppsättning verktyg nära till hands som gör oss mer produktiva och låter oss lösa mer komplexa problem i allt datorrelaterat arbete.
Trots det använder många av oss bara en liten del av verktygen.
Vi kan precis tillräckligt många magiska rader utantill för att klara oss och kopierar blint kommandon från internet när vi kör fast.

Den här kursen är ett försök att [åtgärda det]({{ '/about/' | relative_url }}).

Vi vill lära dig att få ut mer av verktygen du redan känner till,
visa nya verktyg att lägga i verktygslådan,
och förhoppningsvis väcka lust att utforska (och kanske bygga) fler verktyg själv.
Det är detta vi menar är den saknade terminen i många datavetenskapliga utbildningar.

# Kursupplägg

Den poängfria kursen består av nio föreläsningar på en timme vardera,
och varje föreläsning fokuserar på ett [särskilt ämne]({{ '/2026/' | relative_url }}).
Föreläsningarna är till stor del fristående,
men ju längre terminen går desto mer antar vi att du kan innehållet från tidigare pass.
Vi har föreläsningsanteckningar på nätet,
men visst innehåll som tas upp i klass (t.ex. demos) kanske inte finns i anteckningarna.
Som tidigare år spelar vi in föreläsningarna och publicerar dem [på nätet](https://www.youtube.com/@MissingSemester).

Vi försöker täcka mycket på bara några få en-timmespass,
så föreläsningarna är ganska täta.
För att du ska kunna ta till dig innehållet i din egen takt innehåller varje föreläsning en uppsättning övningar som leder dig genom nyckelpunkterna.
Vi kommer inte att ha särskilda mottagningstider,
men vi uppmuntrar frågor på [OSSU Discord](https://ossu.dev/#community), i `#missing-semester-forum`, eller via e-post till [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

På grund av begränsad tid kan vi inte täcka alla verktyg med samma detaljnivå som en fullskalig kurs.
Där det går försöker vi hänvisa dig till resurser för att fördjupa dig i ett verktyg eller ämne,
men om något verkligen fångar ditt intresse får du gärna höra av dig och be om tips.

Om du har synpunkter på kursen får du gärna e-posta dem till [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

# Ämne 1: Skalet

{% comment %}
lecturer: Jon
{% endcomment %}

## Vad är skalet?

Datorer har i dag många gränssnitt för att ge dem kommandon:
avancerade grafiska gränssnitt, röstgränssnitt, AR/VR,
och på senare tid: LLM:er.
De är utmärkta för 80 % av användningsfallen,
men de är ofta i grunden begränsade i vad de låter dig göra.
Du kan inte trycka på en knapp som inte finns,
eller ge ett röstkommando som ingen har programmerat.
För att fullt ut utnyttja verktygen i din dator behöver vi gå tillbaka till grunderna och använda ett textbaserat gränssnitt: skalet.

Nästan alla plattformar du kan få tag i har ett skal i någon form,
och många har flera skal att välja mellan.
Detaljerna skiljer sig,
men i grunden är de ungefär lika:
de låter dig köra program, ge dem indata och läsa deras utdata på ett semistrukturerat sätt.

För att öppna en skalprompt (där du kan skriva kommandon) behöver du först en terminal,
som är det visuella gränssnittet till ett skal.
Din enhet har troligen en installerad,
annars är den enkel att installera:

- **Linux:**
  Tryck `Ctrl + Alt + T` (fungerar i de flesta distributioner).
  Eller sök efter "Terminal" i programmenyn.
- **Windows:**
  Tryck `Win + R`, skriv `cmd` eller `powershell` och tryck Enter.
  Du kan också söka efter "Terminal" eller "Command Prompt" i Start-menyn.
- **macOS:**
  Tryck `Cmd + Space` för att öppna Spotlight,
  skriv "Terminal" och tryck Enter.
  Eller hitta den i Applications -> Utilities -> Terminal.

På Linux och macOS öppnas oftast Bourne Again SHell,
eller "bash".
Det är ett av de mest använda skalen,
och dess syntax liknar det du ser i många andra skal.
På Windows möts du av "batch" eller "powershell" beroende på vilket kommando du körde.
De är Windows-specifika och inte det vi fokuserar på i kursen,
även om de har motsvarigheter till det mesta vi lär ut.
Du vill i stället använda [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) eller en Linux-virtuell maskin.

Andra skal finns också,
ofta med många ergonomiska förbättringar jämfört med bash (fish och zsh är vanliga exempel).
Även om de är populära (alla lärare använder något av dem),
är de inte alls lika allmänt förekommande som bash,
och bygger på samma grundkoncept,
så vi lägger inte fokus på dem i den här föreläsningen.

## Varför ska du bry dig?

Skalet är inte bara (oftast) mycket snabbare än att "klicka runt".
Det ger också en uttryckskraft som du sällan hittar i ett enskilt grafiskt program.
Som vi ska se ger skalet dig möjlighet att _kombinera_ program på kreativa sätt för att automatisera nästan vilken uppgift som helst.

Att kunna skalet är också väldigt nyttigt när du navigerar i världen av fri programvara och öppen källkod,
där installationsinstruktioner ofta kräver skalet,
när du bygger kontinuerlig integration för projekt (som i [föreläsningen om kodkvalitet]({{ '/2026/code-quality/' | relative_url }})),
och när du felsöker fel när andra program kraschar.

## Navigera i skalet

När du startar terminalen ser du en _prompt_ som ofta liknar detta:

```console
missing:~$
```

Det här är skalets huvudsakliga textgränssnitt.
Det berättar att du är på maskinen `missing` och att din "nuvarande arbetskatalog" är `~` (kort för "home").
`$` visar att du inte är root-användare (mer om det senare).
Vid prompten kan du skriva ett _kommando_ som skalet tolkar.
Det mest grundläggande kommandot är att köra ett program:

```console
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$
```

Här körde vi programmet `date`,
som (inte oväntat) skriver ut aktuellt datum och tid.
Sedan ber skalet oss om nästa kommando.
Vi kan också köra kommandon med _argument_:

```console
missing:~$ echo hello
hello
```

Här bad vi skalet köra programmet `echo` med argumentet `hello`.
`echo` skriver helt enkelt ut sina argument.
Skalet parsar kommandot genom att dela på blanktecken,
kör programmet i första ordet,
och skickar efterföljande ord som argument programmet kan läsa.
Om du vill ge ett argument som innehåller blanksteg eller specialtecken (t.ex. en katalog med namnet "My Photos") kan du citera med `'` eller `"` (`"My Photos"`),
eller escapa enskilda tecken med `\` (`My\ Photos`).

Det viktigaste kommandot i början är kanske `man`, kort för "manual".
`man` låter dig slå upp information om kommandon på ditt system.
Om du till exempel kör `man date` får du en beskrivning av `date` och alla argument som ändrar beteendet.
Du kan ofta också få en kortare hjälpsida genom att ge `--help` till kommandot.

> Överväg att installera och använda [`tldr`](https://tldr.sh/) utöver `man`, eftersom det visar vanliga exempel direkt i terminalen.
> LLM:er är också ofta mycket bra på att förklara hur kommandon fungerar och hur de kan anropas.

Efter `man` är `cd` ("change directory") det viktigaste kommandot.
Det är inbyggt i skalet och inte ett separat program (dvs. `which cd` säger "no cd found").
Du skickar in en sökväg,
och den blir din nuvarande arbetskatalog.
Du ser det också i prompten:

```console
missing:~$ cd /bin
missing:/bin$ cd /
missing:/$ cd ~
missing:~$
```

> Skalet har autokomplettering,
> så du kan ofta komplettera sökvägar snabbare med `<TAB>`.

Många kommandon arbetar på nuvarande arbetskatalog om inget annat anges.
Om du är osäker på var du är kan du köra `pwd` eller skriva ut miljövariabeln `$PWD` med `echo $PWD`.
Båda visar nuvarande arbetskatalog.

Nuvarande arbetskatalog är också viktig eftersom den låter oss använda _relativa_ sökvägar.
Alla sökvägar vi sett hittills har varit _absoluta_ --- de börjar med `/` och anger hela vägen från filsystemets rot (`/`).
I praktiken jobbar du oftare med relativa sökvägar,
som är relativa till nuvarande arbetskatalog.
I en relativ sökväg (allt som _inte_ börjar med `/`) slås första komponenten upp i nuvarande katalog och resten följer därifrån.
Till exempel:

```console
missing:~$ cd /
missing:/$ cd bin
missing:/bin$
```

Det finns också två "specialkomponenter" i varje katalog:
`.` och `..`.
`.` är "den här katalogen" och `..` är "föräldrakatalogen".
Alltså:

```console
missing:~$ cd /
missing:/$ cd bin/../bin/../bin/././../bin/..
missing:/$
```

Du kan i regel använda absoluta och relativa sökvägar omväxlande i kommandon.
Kom bara ihåg var din nuvarande arbetskatalog är när du använder relativa.

> Överväg att installera och använda [`zoxide`](https://github.com/ajeetdsouza/zoxide) för snabbare `cd` --- `z` minns sökvägar du ofta besöker.

## Vad finns tillgängligt i skalet?

Hur vet skalet var program som `date` och `echo` finns?
Om skalet ombeds köra ett kommando tittar det på en _miljövariabel_ kallad `$PATH`.
Den listar kataloger som skalet ska söka i efter program:

```console
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

När vi kör `echo` ser skalet att programmet `echo` ska köras,
och söker i den `:`-separerade listan i `$PATH` efter en fil med det namnet.
När det hittas körs den (om filen är _körbar_, mer om det senare).
Vi kan se exakt vilken fil som körs för ett namn med `which`.
Vi kan också kringgå `$PATH` helt genom att ange full _sökväg_ till filen.

Detta visar också hur vi kan ta reda på _alla_ program vi kan köra:
lista innehållet i alla kataloger i `$PATH`.
Det gör vi genom att ge en katalog till `ls`, som listar filer:

```console
missing:~$ ls /bin
```

> Överväg att installera och använda [`eza`](https://eza.rocks/) för en mer användarvänlig `ls`.

Detta skriver på de flesta datorer ut _väldigt_ många program,
men här fokuserar vi på några viktiga:

- `cat file`, som skriver ut innehållet i `file`.
- `sort file`, som skriver ut raderna i `file` sorterade.
- `uniq file`, som tar bort intilliggande dubblettrader i `file`.
- `head file` och `tail file`, som visar de första respektive sista raderna.

> Överväg att installera och använda [`bat`](https://github.com/sharkdp/bat) i stället för `cat` för syntaxfärgning och scrollning.

Det finns också `grep pattern file`, som hittar rader som matchar `pattern` i `file`.
Detta kommando förtjänar lite extra uppmärksamhet eftersom det är både _mycket_ användbart och mer kraftfullt än det ser ut.
`pattern` är ett _reguljärt uttryck_ som kan beskriva komplexa mönster.
Vi går igenom dem i [föreläsningen om kodkvalitet]({{ '/2026/code-quality/#regular-expressions' | relative_url }}).
Du kan också ange en katalog i stället för en fil (eller utelämna för `.`) och skicka `-r` för rekursiv sökning.

> Överväg att installera och använda [`ripgrep`](https://github.com/BurntSushi/ripgrep) i stället för `grep` för ett snabbare och mer användarvänligt (men mindre portabelt) alternativ.
> `ripgrep` söker också rekursivt i aktuell arbetskatalog som standard.

Det finns även mycket nyttiga verktyg med lite mer avancerat gränssnitt.
Ett är `sed`, som är en programmerbar filredigerare.
Den har ett eget språk för automatiserade redigeringar,
men vanligaste användningen är:

```console
missing:~$ sed -i 's/pattern/replacement/g' file
```

Detta ersätter alla förekomster av `pattern` med `replacement` i `file`.
`-i` betyder att ersättningen görs i filen (i stället för att skriva ut modifierat innehåll till stdout).
`s/` uttrycker i sed-språket att vi vill göra en substitution.
`/` separerar mönster från ersättning.
Avslutande `/g` betyder att vi ersätter _alla_ förekomster per rad,
inte bara den första.
Som i `grep` är `pattern` ett reguljärt uttryck,
vilket ger hög uttryckskraft.
Reguljära ersättningar låter också `replacement` referera tillbaka till delar av matchningen.

Sedan har vi `find`, som låter dig hitta filer rekursivt som matchar vissa villkor.
Till exempel:

```console
missing:~$ find ~/Downloads -type f -name "*.zip" -mtime +30
```

Hittar ZIP-filer i nedladdningskatalogen som är äldre än 30 dagar.

```console
missing:~$ find ~ -type f -size +100M -exec ls -lh {} \;
```

Hittar filer större än 100M i hemkatalogen och listar dem.
Observera att `-exec` tar ett _kommando_ som avslutas med ett fristående `;` (som måste escapas), där `{}` ersätts av varje matchande sökväg.

```console
missing:~$ find . -name "*.py" -exec grep -l "TODO" {} \;
```

Hittar alla `.py`-filer med TODO-poster.

`find`-syntaxen kan kännas avskräckande,
men förhoppningsvis ger exemplen en bild av hur användbart det är.

> Överväg att installera och använda [`fd`](https://github.com/sharkdp/fd) i stället för `find` för en mer användarvänlig (men mindre portabel) upplevelse.

Nästa verktyg är `awk`, som likt `sed` har ett eget språk.
Där `sed` är byggt för att redigera filer är `awk` byggt för att parsa dem.
Den vanligaste användningen är datafiler med regelbunden struktur (som CSV) där du vill extrahera vissa delar av varje post (dvs. rad):

```console
missing:~$ awk '{print $2}' file
```

Skriver ut andra blankteckensseparerade kolumnen i varje rad av `file`.
Om du lägger till `-F,` skrivs i stället andra kommaseparerade kolumnen ut.
`awk` kan mycket mer --- filtrera rader, beräkna aggregat med mera.

Genom att kombinera verktygen kan vi göra avancerade saker som:

```console
missing:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

Detta hämtar SSH-loggar från en fjärrserver,
söker efter frånkopplingsmeddelanden,
extraherar användarnamn,
och skriver ut de 10 vanligaste kommaseparerat.
Allt i ett kommando.

## Skalspråket (bash)

Föregående exempel introducerade ett nytt koncept: rör (`|`).
De låter dig kedja utdata från ett program till indata för ett annat.
Det fungerar eftersom de flesta kommandoradsprogram läser från "standard input" om inget `file`-argument ges.
`|` tar "standard output" från programmet före `|` och gör det till standard input för programmet efter `|`.
Det låter dig _komponera_ skalprogram,
och är en stor del av varför skalet är så produktivt.

De flesta skal implementerar faktiskt ett komplett programmeringsspråk (som bash),
precis som Python eller Ruby.
Det har variabler, villkor, loopar och funktioner.
När du kör kommandon skriver du i praktiken små kodsnuttar som skalet tolkar.
Vi lär inte ut hela bash i dag,
men några delar är särskilt nyttiga:

Först, omdirigeringar:
`>file` tar standard output från ett program och skriver till `file` i stället för terminalen.
`>>file` appenderar till `file` i stället för att skriva över.
`<file` säger åt skalet att läsa standard input från fil i stället för tangentbordet.

> Detta är ett bra tillfälle att nämna `tee`.
> `tee` skriver standard input till standard output (som `cat`), men _också_ till fil.
> `verbose cmd | tee verbose.log | grep CRITICAL` bevarar alltså full logg i fil samtidigt som terminalen hålls ren.

Sedan villkor:
`if command1; then command2; command3; fi` kör `command1`,
och om det lyckas körs `command2` och `command3`.
Du kan också ha en `else`-gren.
Vanligast är att använda `test` som `command1`, ofta förkortat `[`,
för villkor som "finns filen" (`test -f file` / `[ -f file ]`) eller "är strängen lika med" (`[ "$var" = "string" ]`).
I bash finns också `[[ ]]`,
en säkrare inbyggd variant med färre underligheter kring quoting.

Bash har två looptyper, `while` och `for`.
`while command1; do command2; command3; done` fungerar som motsvarande `if`,
men upprepas så länge `command1` inte ger fel.
`for varname in a b c d; do command; done` kör `command` fyra gånger,
med `$varname` satt till `a`, `b`, `c` respektive `d`.
I stället för att lista element explicit används ofta kommandosubstitution (_command substitution_),
som:

```bash
for i in $(seq 1 10); do
```

Det kör `seq 1 10` (som skriver ut 1 till 10) och ersätter hela `$()` med kommandots utdata.
Du får då en for-loop med 10 iterationer.
I äldre kod ser du ibland backticks i stället (``for i in `seq 1 10`; do``),
men föredra starkt `$()` eftersom den kan nästlas.

Även om du _kan_ skriva långa skalskript direkt i prompten,
vill du oftast lägga dem i en `.sh`-fil.
Här är ett skript som kör ett program i loop tills det fallerar,
skriver ut logg från den felande körningen,
och belastar CPU i bakgrunden (nyttigt för att reproducera flakiga tester):

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

Detta innehåller flera nya saker värda att gräva i,
som bakgrundsjobb (`&`) för samtidighet,
mer avancerade [skalomdirigeringar](https://www.gnu.org/software/bash/manual/html_node/Redirections.html),
och [aritmetisk expansion](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html).

Det är också värt att titta på programmets två första rader.
Första raden är "shebang".
Den syns i många filtyper, inte bara skalskript.
När en fil som börjar med `#!/path` körs startar skalet programmet på `/path` och skickar filens innehåll som indata.
För skalskript innebär det att innehållet skickas till `/bin/bash`.
Du kan på samma sätt skriva Python-skript med shebang `/usr/bin/python`.

Andra raden gör bash "striktare" och minskar vanliga fallgropar.
`set` tar många argument, men kort:
`-e` avslutar skriptet när ett kommando misslyckas,
`-u` gör att odefinierade variabler blir fel i stället för tom sträng,
och `-o pipefail` gör att fel i en `|`-kedja också får hela skriptet att avslutas.

> Skalprogrammering är ett djupt ämne, och en varning är befogad:
> bash har ovanligt många fallgropar,
> till den grad att det finns [flera](https://tldp.org/LDP/abs/html/gotchas.html) webbplatser som [listar dem](https://mywiki.wooledge.org/BashPitfalls).
> Vi rekommenderar starkt [shellcheck](https://www.shellcheck.net/) när du skriver skalskript.
> LLM:er är också bra på att skriva och felsöka skalskript,
> och på att översätta dem till ett "riktigt" språk (som Python) när de blir för stora för bash.

# Nästa steg

Nu kan du tillräckligt mycket om skalet för att utföra grundläggande uppgifter.
Du bör kunna navigera, hitta filer av intresse och använda basfunktioner i de flesta program.
I nästa föreläsning pratar vi om hur du utför och automatiserar mer komplexa uppgifter med skalet och många användbara kommandoradsprogram.

# Övningar

Alla pass i kursen har tillhörande övningar.
Vissa är tydliga och konkreta,
andra är öppnare,
som "testa att använda X och Y".
Vi uppmuntrar dig starkt att prova.

Vi har inte skrivit facit till övningarna.
Om du fastnar får du gärna skriva i `#missing-semester-forum` på [Discord](https://ossu.dev/#community) eller e-posta oss och beskriva vad du testat.
De här övningarna fungerar också bra som startprompter i en LLM-konversation.
Det verkliga värdet är vägen till svaret, inte själva svaret.
Följ gärna sidospår och fråga "varför" när du jobbar igenom dem.

1. För den här kursen behöver du använda ett Unix-skal som Bash eller ZSH.
   Om du använder Linux eller macOS behöver du inte göra något särskilt.
   Om du använder Windows måste du se till att du inte kör cmd.exe eller PowerShell.
   Du kan använda [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) eller en Linux-VM för Unix-liknande kommandoradsverktyg.
   För att kontrollera att du kör rätt skal kan du testa `echo $SHELL`.
   Om det står något som `/bin/bash` eller `/usr/bin/zsh` kör du rätt program.

1. Vad gör flaggan `-l` i `ls`?
   Kör `ls -l /` och granska utdata.
   Vad betyder de första 10 tecknen på varje rad?
   (Tips: `man ls`.)

1. I kommandot `find ~/Downloads -type f -name "*.zip" -mtime +30` är `*.zip` en "glob".
   Vad är en glob?
   Skapa en testkatalog med några filer och experimentera med `ls *.txt`, `ls file?.txt` och `ls {a,b,c}.txt`.
   Se [Mönstermatchning](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html) i Bash-manualen.

1. Vad är skillnaden mellan `'single quotes'`, `"double quotes"` och `$'ANSI quotes'`?
   Skriv ett kommando som ekar en sträng med en bokstavlig `$`, en `!` och en radbrytning.
   Se [citering](https://www.gnu.org/software/bash/manual/html_node/Quoting.html).

1. Skalet har tre standardströmmar: stdin (0), stdout (1) och stderr (2).
   Kör `ls /nonexistent /tmp` och omdirigera stdout till en fil och stderr till en annan.
   Hur omdirigerar du båda till samma fil?
   Se [omdirigeringar](https://www.gnu.org/software/bash/manual/html_node/Redirections.html).

1. `$?` innehåller exit-status för senaste kommandot (0 = lyckat).
   `&&` kör nästa kommando bara om föregående lyckas och `||` kör nästa bara om föregående misslyckas.
   Skriv ett enradskommando som skapar `/tmp/mydir` bara om den inte redan finns.
   Se [avslutningsstatus](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html).

1. Varför måste `cd` vara inbyggt i skalet i stället för ett fristående program?
   (Tips: fundera på vad en barnprocess kan och inte kan påverka i sin förälder.)

1. Skriv ett skript som tar ett filnamn som argument (`$1`) och kontrollerar om filen finns med `test -f` eller `[ -f ... ]`.
   Skriptet ska skriva olika meddelanden beroende på om filen finns.
   Se [villkorsuttryck i Bash](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html).

1. Spara skriptet från förra övningen till en fil (t.ex. `check.sh`).
   Testa `./check.sh somefile`.
   Vad händer?
   Kör sedan `chmod +x check.sh` och testa igen.
   Varför är steget nödvändigt?
   (Tips: jämför `ls -l check.sh` före och efter `chmod`.)

1. Vad händer om du lägger till `-x` till `set`-flaggorna i ett skript?
   Prova med ett enkelt skript och observera utdata.
   Se [det inbyggda kommandot set](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html).

1. Skriv ett kommando som kopierar en fil till en backup med dagens datum i filnamnet (t.ex. `notes.txt` -> `notes_2026-01-12.txt`).
   (Tips: `$(date +%Y-%m-%d)`.)
   Se [kommandosubstitution (command substitution)](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html).

1. Modifiera skriptet för flakiga tester från föreläsningen så att testkommandot tas som argument i stället för hårdkodat `cargo test my_test`.
   (Tips: `$1` eller `$@`.)
   Se [särskilda parametrar](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html).

1. Använd rör (`|`) för att hitta de 5 vanligaste filändelserna i din hemkatalog.
   (Tips: kombinera `find`, `grep` eller `sed` eller `awk`, `sort`, `uniq -c` och `head`.)

1. `xargs` gör om rader från stdin till kommandoradsargument.
   Använd `find` och `xargs` tillsammans (inte `find -exec`) för att hitta alla `.sh`-filer i en katalog och räkna rader i varje med `wc -l`.
   Bonus: få det att fungera med filnamn som innehåller blanksteg.
   (Tips: `-print0` och `-0`.)
   Se `man xargs`.

1. Använd `curl` för att hämta HTML för kurswebbplatsen (`https://missing.csail.mit.edu/`) och skicka resultatet genom ett rör till `grep` för att räkna hur många föreläsningar som listas.
   (Tips: leta efter ett mönster som finns en gång per föreläsning och använd `curl -s` för tyst läge.)

1. [`jq`](https://jqlang.github.io/jq/) är ett kraftfullt verktyg för att bearbeta JSON.
   Hämta exempeldata på `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json` med `curl` och använd `jq` för att extrahera namnen på personer vars version är större än 6.
   (Tips: skicka först genom ett rör till `jq .` för att se strukturen och testa sedan `jq '.[] | select(...) | .name'`.)

1. `awk` kan filtrera rader baserat på kolumnvärden och manipulera utdata.
   Exempel: `awk '$3 ~ /pattern/ {$4=""; print}'` skriver bara rader där tredje kolumnen matchar `pattern`, men utelämnar fjärde kolumnen.
   Skriv ett `awk`-kommando som bara skriver rader där andra kolumnen är större än 100, och byter plats på första och tredje kolumnen.
   Testa med: `printf 'a 50 x\nb 150 y\nc 200 z\n'`.

1. Dela upp SSH-loggkedjan från föreläsningen: vad gör varje steg?
   Bygg sedan något liknande för att hitta dina mest använda skalkommandon från `~/.bash_history` (eller `~/.zsh_history`).
