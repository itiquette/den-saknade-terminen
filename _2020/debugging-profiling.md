---
layout: lecture
title: "Felsökning och profilering"
description: >
  Lär dig hur du felsöker program med loggning, felsökningsverktyg och statisk analys, och hur du profilerar kod för prestanda.
thumbnail: /static/assets/thumbnails/2020/lec7.png
date: 2020-01-23
ready: true
video:
  aspect: 56.25
  id: l812pUnKxME
---

En gyllene regel inom programmering är att kod inte gör det du förväntar dig att den ska göra, utan det du säger åt den att göra.
Att överbrygga den luckan kan ibland vara ganska svårt.
I föreläsningen går vi igenom användbara tekniker för att hantera felaktig och resurshungrig kod: felsökning och profilering.

# Felsökning (debugging)

## Printf-felsökning och loggning

"Det effektivaste felsökningsverktyget är fortfarande eftertanke, i kombination med välplacerade utskriftssatser" — Brian Kernighan, _Unix for Beginners_.

Ett första sätt att felsöka ett program är att lägga till utskriftssatser kring där du upptäckt problemet, och fortsätta iterera tills du extraherat tillräcklig information för att förstå vad som orsakar felet.

Ett andra sätt är att använda loggning i ditt program i stället för ad hoc-utskriftssatser.
Loggning är bättre än vanliga utskriftssatser av flera skäl:

- Du kan logga till filer, sockets eller till och med fjärrservrar i stället för standardutmatning.
- Loggning stödjer allvarlighetsnivåer (som INFO, DEBUG, WARN, ERROR, etc.) som gör att du kan filtrera utdata därefter.
- För nya problem finns en rimlig chans att loggarna redan innehåller tillräcklig information för att upptäcka vad som går fel.

[Här]({{ '/static/files/logger.py' | relative_url }}) finns ett exempelprogram som loggar meddelanden:

```bash
$ python logger.py
# Rå utdata som med vanliga utskriftssatser
$ python logger.py log
# Loggformaterad utdata
$ python logger.py log ERROR
# Skriv bara ut ERROR-nivå och högre
$ python logger.py color
# Färgformaterad utdata
```

Ett av mina favorittips för mer läsbara loggar är att färgkoda dem.
Nu har du troligen märkt att terminalen använder färger för att göra saker mer läsbara.
Men hur gör den det?
Program som `ls` eller `grep` använder [ANSI-kontrollkoder](https://en.wikipedia.org/wiki/ANSI_escape_code), som är särskilda teckensekvenser för att tala om för skalet att ändra färg på utdata.
Till exempel skriver `echo -e "\e[38;2;255;0;0mDet här är rött\e[0m"` ut meddelandet `Det här är rött` i rött i terminalen, så länge terminalen stödjer [äkta färg (true color)](https://github.com/termstandard/colors#truecolor-support-in-output-devices).
Om din terminal inte stödjer det (t.ex. macOS Terminal.app) kan du använda mer allmänt stödda kontrollkoder för 16 färger, till exempel `echo -e "\e[31;1mDet här är rött\e[0m"`.

Skriptet nedan visar hur man skriver ut många RGB-färger i terminalen (igen, så länge äkta färg stöds).

```bash
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done
```

## Tredjepartsloggar

När du börjar bygga större programvarusystem stöter du sannolikt på beroenden som kör som separata program.
Webbservrar, databaser och meddelandebrokers är vanliga exempel på sådana beroenden.
När du interagerar med dessa system behöver du ofta läsa deras loggar, eftersom felmeddelanden på klientsidan inte alltid räcker.

Som tur är skriver de flesta program sina egna loggar någonstans i systemet.
I UNIX-system är det vanligt att program skriver loggar under `/var/log`.
Till exempel placerar webbservern [NGINX](https://www.nginx.com/) sina loggar under `/var/log/nginx`.
På senare tid har system också börjat använda en **systemlogg**, som i allt högre grad är platsen där alla loggmeddelanden hamnar.
De flesta (men inte alla) Linux-system använder `systemd`, en systemdemon som styr många delar av systemet, som vilka tjänster som är aktiverade och körs.
`systemd` lägger loggar under `/var/log/journal` i ett specialformat, och du kan använda kommandot [`journalctl`](https://www.man7.org/linux/man-pages/man1/journalctl.1.html) för att visa meddelandena.
På macOS finns fortfarande `/var/log/system.log`, men fler och fler verktyg använder systemloggen som kan visas med [`log show`](https://www.manpagez.com/man/1/log/).
På de flesta UNIX-system kan du också använda [`dmesg`](https://www.man7.org/linux/man-pages/man1/dmesg.1.html) för att komma åt kärnloggen.

För att skriva till systemloggar kan du använda skalprogrammet [`logger`](https://www.man7.org/linux/man-pages/man1/logger.1.html).
Här är ett exempel på att använda `logger` och kontrollera att posten hamnat i systemloggen.
Dessutom har de flesta programspråk bindningar för loggning till systemloggen.

```bash
logger "Hej loggar"
# På macOS
log show --last 1m | grep Hej
# På Linux
journalctl --since "1m ago" | grep Hej
```

Som vi såg i föreläsningen om datahantering kan loggar vara väldigt utförliga och kräva viss bearbetning och filtrering för att få fram rätt information.
Om du ofta filtrerar i `journalctl` och `log show` kan du använda deras flaggor som gör en första filtrering av utdata.
Det finns också verktyg som [`lnav`](https://lnav.org/) som ger bättre presentation och navigering i loggfiler.

## Felsökare

När printf-felsökning inte räcker bör du använda en felsökare.
Felsökare är program som låter dig interagera med ett programs körning, och de gör det möjligt att:

- Stoppa programkörningen när en viss rad nås.
- Stega genom programmet en instruktion i taget.
- Inspektera variabelvärden efter att programmet kraschat.
- Stoppa körningen villkorligt när ett givet villkor uppfylls.
- Och mycket mer.

Många programmeringsspråk levereras med någon form av felsökare.
I Python är detta Python Debugger [`pdb`](https://docs.python.org/3/library/pdb.html).

Här är en kort beskrivning av några kommandon som `pdb` stödjer:

- **l**(ist) - Visar 11 rader runt aktuell rad eller fortsätter föregående listning.
- **s**(tep) - Kör aktuell rad och stoppar vid första möjliga tillfälle.
- **n**(ext) - Fortsätter tills nästa rad i aktuell funktion nås eller funktionen returnerar.
- **b**(reak) - Sätter en brytpunkt (beroende på argument).
- **p**(rint) - Utvärderar uttrycket i aktuell kontext och skriver ut värdet.
  Det finns även **pp** för utskrift med [`pprint`](https://docs.python.org/3/library/pprint.html).
- **r**(eturn) - Fortsätter tills aktuell funktion returnerar.
- **q**(uit) - Avslutar felsökaren.

Låt oss gå igenom ett exempel där vi använder `pdb` för att fixa följande felaktiga Python-kod (se föreläsningsvideon).

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n):
            if arr[j] > arr[j+1]:
                arr[j] = arr[j+1]
                arr[j+1] = arr[j]
    return arr

print(bubble_sort([4, 2, 1, 8, 7, 6]))
```

Notera att eftersom Python är ett tolkat språk kan vi använda `pdb`-skalet både för att köra kommandon och instruktioner.
[`ipdb`](https://pypi.org/project/ipdb/) är en förbättrad `pdb` som använder [`IPython`](https://ipython.org)-REPL och ger tab completion, syntax highlighting, bättre tracebacks och bättre introspektion samtidigt som samma gränssnitt som `pdb`-modulen behålls.

För mer låg-nivåprogrammering vill du sannolikt titta på [`gdb`](https://www.gnu.org/software/gdb/) (och dess livskvalitetsförbättring [`pwndbg`](https://github.com/pwndbg/pwndbg)) samt [`lldb`](https://lldb.llvm.org/).
De är optimerade för felsökning av C-liknande språk men låter dig undersöka i princip vilken process som helst och se dess aktuella maskintillstånd: register, stack, programräknare osv.

## Specialiserade verktyg

Även om det du försöker felsöka är en svart-låda-binär finns verktyg som kan hjälpa.
När program behöver utföra åtgärder som bara kärnan kan göra använder de [systemanrop](https://en.wikipedia.org/wiki/System_call).
Det finns kommandon som låter dig spåra vilka systemanrop programmet gör.
I Linux finns [`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html), och i macOS/BSD finns [`dtrace`](https://dtrace.org/about/).
`dtrace` kan vara svårt att använda eftersom det använder sitt eget språk, `D`, men det finns ett omslutarprogram som heter [`dtruss`](https://www.manpagez.com/man/1/dtruss/) och ger ett gränssnitt mer likt `strace` (mer detaljer [här](https://8thlight.com/blog/colin-jones/2015/11/06/dtrace-even-better-than-strace-for-osx.html)).

Nedan följer exempel på hur `strace` eller `dtruss` används för att visa spårning av systemanropet [`stat`](https://www.man7.org/linux/man-pages/man2/stat.2.html) vid körning av `ls`.
För en djupare genomgång av `strace` är [den här artikeln](https://blogs.oracle.com/linux/strace-the-sysadmins-microscope-v2) och [det här zinet](https://jvns.ca/strace-zine-unfolded.pdf) bra läsning.

```bash
# På Linux
sudo strace -e lstat ls -l > /dev/null
# På macOS
sudo dtruss -t lstat64_extended ls -l > /dev/null
```

I vissa situationer kan du behöva titta på nätverkspaket för att förstå felet i programmet.
Verktyg som [`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) och [Wireshark](https://www.wireshark.org/) är paketanalysverktyg som låter dig läsa innehållet i nätverkspaket och filtrera dem med olika kriterier.

För webbutveckling är utvecklarverktygen i Chrome/Firefox praktiska.
De har ett stort antal verktyg, inklusive:
- Källkod - inspektera HTML/CSS/JS-källkod för vilken webbplats som helst.
- Live-redigering av HTML, CSS och JS - ändra innehåll, stil och beteende för att testa (du kan själv se att skärmdumpar av webbplatser inte är giltiga bevis).
- Javascript-skal - kör kommandon i JS-REPL.
- Network - analysera tidslinjen för förfrågningar.
- Storage - undersök cookies och lokalt applikationslager.

## Statisk analys

För vissa problem behöver du inte köra någon kod alls.
Till exempel kan du, bara genom att noggrant läsa kod, upptäcka att slingvariabeln skuggar ett redan existerande variabel- eller funktionsnamn, eller att ett program läser en variabel innan den definierats.
Det är här verktyg för [statisk analys](https://en.wikipedia.org/wiki/Static_program_analysis) kommer in.
Program för statisk analys tar källkod som indata och analyserar den med kodregler för att resonera om korrekthet.

I följande Python-exempel finns flera misstag.
Först skuggar slingvariabeln `foo` den tidigare definitionen av funktionen `foo`.
Vi skrev också `baz` i stället för `bar` på sista raden, så programmet kraschar efter `sleep`-anropet (som tar en minut).

```python
import time

def foo():
    return 42

for foo in range(5):
    print(foo)
bar = 1
bar *= 0.2
time.sleep(60)
print(baz)
```

Verktyg för statisk analys kan identifiera den typen av problem.
När vi kör [`pyflakes`](https://pypi.org/project/pyflakes) på koden får vi fel kopplade till båda programfelen.
[`mypy`](https://mypy-lang.org/) är ett annat verktyg som kan upptäcka typkontrollproblem.
Här varnar `mypy` för att `bar` först är en `int` och sedan omvandlas till `float`.
Notera igen att alla dessa problem upptäcktes utan att köra koden.

```bash
$ pyflakes foobar.py
foobar.py:6: redefinition of unused 'foo' from line 3
foobar.py:11: undefined name 'baz'

$ mypy foobar.py
foobar.py:6: error: Incompatible types in assignment (expression has type "int", variable has type "Callable[[], Any]")
foobar.py:9: error: Incompatible types in assignment (expression has type "float", variable has type "int")
foobar.py:11: error: Name 'baz' is not defined
Found 3 errors in 1 file (checked 1 source file)
```

I föreläsningen om skalverktyg tog vi upp [`shellcheck`](https://www.shellcheck.net/), som är ett liknande verktyg för skalskript.

De flesta redigerare och IDE:er kan visa utdata från dessa verktyg direkt i redigeraren och markera varningar och fel.
Det kallas ofta **lintning** och kan också användas för andra typer av problem, som stilavvikelser eller osäkra konstruktioner.

I Vim kan insticksmodulerna [`ale`](https://vimawesome.com/plugin/ale) eller [`syntastic`](https://vimawesome.com/plugin/syntastic) ge den funktionen.
För Python är [`pylint`](https://github.com/PyCQA/pylint) och [`pep8`](https://pypi.org/project/pep8/) exempel på stil-linters, och [`bandit`](https://pypi.org/project/bandit/) är ett verktyg för att hitta vanliga säkerhetsproblem.
För andra språk har människor sammanställt omfattande listor över användbara verktyg för statisk analys, till exempel [Awesome Static Analysis](https://github.com/mre/awesome-static-analysis) (du kan titta på avsnittet _Writing_), och för linters finns [Awesome Linters](https://github.com/caramelomartins/awesome-linters).

Ett komplement till stil-lintning är kodformaterare, som [`black`](https://github.com/psf/black) för Python, `gofmt` för Go, `rustfmt` för Rust eller [`prettier`](https://prettier.io/) för JavaScript, HTML och CSS.
Dessa verktyg autoformaterar koden så att den följer vedertagna stilkonventioner för det aktuella språket.
Även om du kanske inte vill ge upp stilkontroll över koden hjälper standardiserat format andra att läsa din kod och gör dig bättre på att läsa andras (stilmässigt standardiserade) kod.

# Profilering

Även om koden fungerar som väntat kanske det inte räcker om den samtidigt slukar all CPU eller allt minne.
Algoritmkurser lär ofta ut big _O_-notation men inte hur man hittar flaskhalsar i program.
Eftersom [för tidig optimering är roten till allt ont](https://wiki.c2.com/?PrematureOptimization) bör du lära dig både profileringsverktyg och övervakningsverktyg.
De hjälper dig att förstå vilka delar av programmet som tar mest tid och resurser, så att du kan fokusera optimering där den spelar roll.

## Tidmätning

Precis som i felsökningsfallet räcker det i många scenarier att bara skriva ut tiden som koden tog mellan två punkter.
Här är ett exempel i Python med modulen [`time`](https://docs.python.org/3/library/time.html).

```python
import time, random
n = random.randint(1, 10) * 100

# Hämta aktuell tid
start = time.time()

# Gör lite arbete
print("Vilar i {} ms".format(n))
time.sleep(n/1000)

# Beräkna tid mellan start och nu
print(time.time() - start)

# Exempelutdata
# Vilar i 500 ms
# 0.5713930130004883
```

Förfluten tid kan dock vara missvisande eftersom datorn kan köra andra processer samtidigt eller vänta på händelser.
Det är vanligt att verktyg skiljer på _Real_, _User_ och _Sys_ tid.
Generellt visar _User_ + _Sys_ hur mycket tid processen faktiskt spenderade på CPU:n (mer förklaring [här](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)).

- _Real_ - Förfluten tid från start till slut, inklusive tid som tas av andra processer och blockeringstid (t.ex. väntan på I/O eller nätverk).
- _User_ - Tid i CPU:n för användarkod.
- _Sys_ - Tid i CPU:n för kärnkod.

Prova till exempel att köra ett kommando som gör en HTTP-förfrågan och prefixa det med [`time`](https://www.man7.org/linux/man-pages/man1/time.1.html).
Med långsam anslutning kan du få utdata som nedan.
Här tog förfrågan över 2 sekunder, men processen tog bara 15 ms användartid och 12 ms kärntid på CPU.

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real    0m2.561s
user    0m0.015s
sys     0m0.012s
```

## Profilerare

### CPU

Oftast när folk säger _profilerare_ menar de egentligen _CPU-profilerare_, som är vanligast. Det finns två huvudtyper av CPU-profilerare: _spårning_ och _sampling_.
Spårningsprofilerare behåller en logg över varje funktionsanrop programmet gör, medan samplingsprofilerare provar programmet med jämna mellanrum (vanligen varje millisekund) och spelar in programmets stack.
De använder dessa data för att presentera aggregerad statistik över vad programmet lagt mest tid på.
[Här](https://jvns.ca/blog/2017/12/17/how-do-ruby---python-profilers-work-) finns en bra introduktion om du vill ha mer detaljer.

De flesta programmeringsspråk har någon kommandoradsprofilerare du kan använda för att analysera kod.
De integreras ofta med fullfjädrade IDE:er, men i föreläsningen fokuserar vi på kommandoradsverktygen.

I Python kan vi använda modulen `cProfile` för att profilera tid per funktionsanrop.
Här är ett enkelt exempel som implementerar en rudimentär grep i Python:

```python
#!/usr/bin/env python

import sys, re

def grep(pattern, file):
    with open(file, 'r') as f:
        print(file)
        for i, line in enumerate(f.readlines()):
            pattern = re.compile(pattern)
            match = pattern.search(line)
            if match is not None:
                print("{}: {}".format(i, line), end="")

if __name__ == '__main__':
    times = int(sys.argv[1])
    pattern = sys.argv[2]
    for i in range(times):
        for file in sys.argv[3:]:
            grep(pattern, file)
```

Vi kan profilera den här koden med kommandot nedan.
Om vi analyserar utdata ser vi att I/O tar mest tid och att kompilering av regex också tar en del tid.
Eftersom regex bara behöver kompileras en gång kan vi flytta ut det ur slingan.

```
$ python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py

[utelämnad programutdata]

 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     8000    0.266    0.000    0.292    0.000 {built-in method io.open}
     8000    0.153    0.000    0.894    0.000 grep.py:5(grep)
    17000    0.101    0.000    0.101    0.000 {built-in method builtins.print}
     8000    0.100    0.000    0.129    0.000 {method 'readlines' of '_io._IOBase' objects}
    93000    0.097    0.000    0.111    0.000 re.py:286(_compile)
    93000    0.069    0.000    0.069    0.000 {method 'search' of '_sre.SRE_Pattern' objects}
    93000    0.030    0.000    0.141    0.000 re.py:231(compile)
    17000    0.019    0.000    0.029    0.000 codecs.py:318(decode)
        1    0.017    0.017    0.911    0.911 grep.py:3(<module>)

[omitted lines]
```

En invändning mot Pythons `cProfile` (och många profilerare) är att de visar tid per funktionsanrop.
Det kan snabbt bli ointuitivt, särskilt när du använder tredjepartsbibliotek eftersom interna anrop också räknas.
Ett mer intuitivt sätt att visa profileringsdata är tid per kodrad, vilket är vad radprofilerare (_line profilers_) gör.

Till exempel gör följande Python-kod en förfrågan till kurswebbplatsen och tolkar svaret för att hämta alla URL:er på sidan:

```python
#!/usr/bin/env python
import requests
from bs4 import BeautifulSoup

# Detta är en dekorator som talar om för line_profiler
# att vi vill analysera den här funktionen
@profile
def get_urls():
    response = requests.get('https://missing.csail.mit.edu')
    s = BeautifulSoup(response.content, 'lxml')
    urls = []
    for url in s.find_all('a'):
        urls.append(url['href'])

if __name__ == '__main__':
    get_urls()
```

Om vi använder Pythons `cProfile` får vi över 2500 rader utdata, och även med sortering är det svårt att se var tiden går.
En snabb körning med [`line_profiler`](https://github.com/pyutils/line_profiler) visar i stället tidsåtgång per rad:

```bash
$ kernprof -l -v a.py
Wrote profile results to urls.py.lprof
Timer unit: 1e-06 s

Total time: 0.636188 s
File: a.py
Function: get_urls at line 5

Line #  Hits         Time  Per Hit   % Time  Line Contents
==============================================================
 5                                           @profile
 6                                           def get_urls():
 7         1     613909.0 613909.0     96.5      response = requests.get('https://missing.csail.mit.edu')
 8         1      21559.0  21559.0      3.4      s = BeautifulSoup(response.content, 'lxml')
 9         1          2.0      2.0      0.0      urls = []
10        25        685.0     27.4      0.1      for url in s.find_all('a'):
11        24         33.0      1.4      0.0          urls.append(url['href'])
```

### Minne

I språk som C eller C++ kan minnesläckor göra att programmet aldrig frigör minne som det inte längre behöver.
För att hjälpa vid minnesfelsökning kan du använda verktyg som [Valgrind](https://valgrind.org/) som hjälper dig att hitta minnesläckor.

I språk med skräpsamling, som Python, är minnesprofilerare fortfarande användbara, eftersom objekt inte samlas upp så länge du har referenser till dem.
Här är ett exempelprogram och dess utdata när det körs med [memory-profiler](https://pypi.org/project/memory-profiler/) (notera dekoratorn, liksom i `line-profiler`).

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

```bash
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

### Händelseprofilering

Precis som med `strace` för felsökningsfallet kan du vilja ignorera kodens detaljer och behandla det du kör som en svart låda när du profilerar.
Kommandot [`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) abstraherar bort CPU-skillnader och rapporterar inte tid eller minne direkt, utan systemhändelser relaterade till programmet.
Till exempel kan `perf` enkelt rapportera dålig cachelokalitet, höga nivåer av sidfel eller låsningsloopar.
Här är en översikt:

- `perf list` - Lista händelser som kan spåras med perf.
- `perf stat COMMAND ARG1 ARG2` - Hämtar antal för olika händelser relaterade till en process eller ett kommando.
- `perf record COMMAND ARG1 ARG2` - Spelar in körningen av ett kommando och sparar statistikdata i en fil som heter `perf.data`.
- `perf report` - Formaterar och skriver ut data i `perf.data`.

### Visualisering

Profileringsutdata för verkliga program innehåller ofta stora mängder information på grund av programvaruprojekts inneboende komplexitet.
Människor är visuella och ganska dåliga på att läsa stora mängder siffror och förstå dem.
Därför finns många verktyg för att visa profileringsutdata på ett lättare sätt.

Ett vanligt sätt att visa CPU-profileringsdata för samplingsprofilerare är [flamdiagram (flame graph)](https://www.brendangregg.com/flamegraphs.html), som visar en hierarki av funktionsanrop längs Y-axeln och tidsåtgång proportionellt längs X-axeln.
De är också interaktiva, så du kan zooma in i specifika delar av programmet och se stackspår (prova att klicka i bilden nedan).

[![Flamdiagram](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

Anropsgrafer (call graphs) eller kontrollflödesgrafer (control flow graphs) visar relationer mellan underrutiner i ett program genom att representera funktioner som noder och funktionsanrop mellan dem som riktade kanter.
När detta kombineras med profileringsdata som antal anrop och tidsåtgång kan anropsgrafer vara väldigt användbara för att tolka programmets flöde.
I Python kan du använda biblioteket [`pycallgraph`](https://pycallgraph.readthedocs.io/) för att generera dem.

![Anropsgraf](https://upload.wikimedia.org/wikipedia/commons/2/2f/A_Call_Graph_generated_by_pycallgraph.png)

## Resursövervakning

Ibland är första steget i prestandaanalys att förstå faktiskt resursutnyttjande.
Program kör ofta långsamt när de är resursbegränsade, till exempel om minnet är för litet eller nätverket långsamt.
Det finns mängder av kommandoradsverktyg för att undersöka och visa olika systemresurser som CPU-användning, minnesanvändning, nätverk, diskanvändning och så vidare.

- **Allmän övervakning** - Det kanske mest populära är [`htop`](https://htop.dev/), en förbättrad version av [`top`](https://www.man7.org/linux/man-pages/man1/top.1.html).
`htop` visar olika statistik för processer som körs i systemet.
`htop` har många alternativ och tangentbindningar.
Några användbara är `<F6>` för att sortera processer, `t` för trädhierarki och `h` för att växla trådar.
Se också [`glances`](https://nicolargo.github.io/glances/) för en liknande implementation med bra UI.
För aggregerade mått över alla processer är [`dool`](https://github.com/scottchiefbaker/dool) ett annat smidigt verktyg som beräknar realtidsmått för många delsystem, som I/O, nätverk, CPU-utnyttjande, context switches osv.
- **I/O-operationer** - [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) visar liveinformation om I/O-användning och är praktiskt för att se om en process gör tung disk-I/O.
- **Diskanvändning** - [`df`](https://www.man7.org/linux/man-pages/man1/df.1.html) visar mått per partition och [`du`](https://man7.org/linux/man-pages/man1/du.1.html) visar **d**isk **u**sage per fil i aktuell katalog.
I dessa verktyg betyder flaggan `-h` **h**uman readable format.
En mer interaktiv variant av `du` är [`ncdu`](https://dev.yorhel.nl/ncdu), där du kan navigera kataloger och radera filer/kataloger under navigeringen.
- **Minnesanvändning** - [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) visar total mängd ledigt och använt minne i systemet.
Minne visas också i verktyg som `htop`.
- **Öppna filer** - [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) listar filinformation om filer öppnade av processer.
Det kan vara användbart för att se vilken process som öppnat en viss fil.
- **Nätverksanslutningar och konfiguration** - [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) låter dig övervaka statistik för inkommande och utgående nätverkspaket samt gränssnittsstatistik.
Ett vanligt användningsfall är att ta reda på vilken process som använder en viss port på en maskin.
För routing, nätverksenheter och gränssnitt kan du använda [`ip`](https://man7.org/linux/man-pages/man8/ip.8.html).
Notera att `netstat` och `ifconfig` har ersatts av dessa verktyg.
- **Nätverksanvändning** - [`nethogs`](https://github.com/raboof/nethogs) och [`iftop`](https://pdw.ex-parrot.com/iftop/) är bra interaktiva CLI-verktyg för övervakning av nätverksanvändning.

Om du vill testa dessa verktyg kan du också skapa artificiell belastning med kommandot [`stress`](https://linux.die.net/man/1/stress).

### Specialverktyg

Ibland är svart-låda-prestandatestning allt du behöver för att avgöra vilken programvara du ska använda.
Verktyg som [`hyperfine`](https://github.com/sharkdp/hyperfine) låter dig snabbt prestandatesta kommandoradsprogram.
I föreläsningen om skalverktyg och skriptning rekommenderade vi till exempel `fd` framför `find`.
Vi kan använda `hyperfine` för att jämföra dem i vanliga uppgifter.
I exemplet nedan var `fd` 20x snabbare än `find` på min maskin.

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

Precis som i felsökningsfallet kommer webbläsare också med fantastiska verktyg för profilering av sidladdning, så att du kan se var tiden går (laddning, återgivning, skriptning osv).
Mer information för [Firefox](https://profiler.firefox.com/docs/) och [Chrome](https://developers.google.com/web/tools/chrome-devtools/rendering-tools).

# Övningar

## Felsökning
1. Använd `journalctl` i Linux eller `log show` i macOS för att hämta superuser-åtkomster och kommandon under senaste dygnet.
Om det inte finns några kan du köra några ofarliga kommandon som `sudo ls` och kontrollera igen.

1. Gör den här praktiska [`pdb`](https://github.com/spiside/pdb-tutorial)-handledningen för att bekanta dig med kommandona.
För en djupare guide, läs [den här](https://realpython.com/python-debugging-pdb).

1. Installera [`shellcheck`](https://www.shellcheck.net/) och prova att kontrollera skriptet nedan.
Vad är fel i koden?
Fixa det.
Installera ett linter-tillägg i din redigerare så att varningarna visas automatiskt.

   ```bash
   #!/bin/sh
   ## Exempel: ett typiskt skript med flera problem
   for f in $(ls *.m3u)
   do
      grep -qi hq.*mp3 $f \
        && echo -e 'Spellistan $f innehåller en mp3-fil i hög kvalitet'
   done
   ```

1. (Avancerad) Läs om omvänd felsökning ([reverse debugging](https://undo.io/resources/reverse-debugging-whitepaper/)) och få ett enkelt exempel att fungera med [`rr`](https://rr-project.org/) eller [`RevPDB`](https://morepypy.blogspot.com/2016/07/reverse-debugging-for-python.html).

## Profilering

1. [Här]({{ '/static/files/sorts.py' | relative_url }}) finns implementationer av några sorteringsalgoritmer.
Använd [`cProfile`](https://docs.python.org/3/library/profile.html) och [`line_profiler`](https://github.com/pyutils/line_profiler) för att jämföra exekveringstid för insertion sort och quicksort.
Vad är flaskhalsen i respektive algoritm?
Använd sedan `memory_profiler` för att kontrollera minnesförbrukningen.
Varför är insertion sort bättre?
Kolla nu in-place-versionen av quicksort.
Utmaning: använd `perf` för att titta på cykelantal och cacheträffar/missar för varje algoritm.

1. Här är lite (möjligen omständlig) Python-kod för att beräkna Fibonaccital med en funktion per tal.

   ```python
   #!/usr/bin/env python
   def fib0(): return 0

   def fib1(): return 1

   s = """def fib{}(): return fib{}() + fib{}()"""

   if __name__ == '__main__':

       for n in range(2, 10):
           exec(s.format(n, n-1, n-2))
       # from functools import lru_cache
       # for n in range(10):
       #     exec("fib{} = lru_cache(1)(fib{})".format(n, n))
       print(eval("fib9()"))
   ```

   Lägg koden i en fil och gör den körbar.
   Installera förkrav: [`pycallgraph`](https://lewiscowles1986.github.io/py-call-graph/) och [`graphviz`](https://graphviz.org/).
   (Om du kan köra `dot` har du redan GraphViz.)
   Kör koden som den är med `pycallgraph graphviz -- ./fib.py` och kontrollera filen `pycallgraph.png`.
   Hur många gånger anropas `fib0`?
   Vi kan göra bättre genom att memoizera funktionerna.
   Avkommentera de kommenterade raderna och generera bilderna igen.
   Hur många gånger anropar vi varje `fibN`-funktion nu?

1. Ett vanligt problem är att en port du vill lyssna på redan används av en annan process.
Låt oss lära oss hitta processens PID.
Kör först `python -m http.server 4444` för att starta en minimal webbserver som lyssnar på port `4444`.
Kör i en annan terminal `lsof | grep LISTEN` för att skriva ut alla lyssnande processer och portar.
Hitta processens PID och avsluta den med `kill <PID>`.

1. Att begränsa en process resurser kan vara ett annat användbart verktyg i verktygslådan.
Prova `stress -c 3` och visualisera CPU-förbrukningen med `htop`.
Kör sedan `taskset --cpu-list 0,2 stress -c 3` och visualisera igen.
Använder `stress` tre CPU:er?
Varför inte?
Läs [`man taskset`](https://www.man7.org/linux/man-pages/man1/taskset.1.html).
Utmaning: uppnå samma sak med [`cgroups`](https://www.man7.org/linux/man-pages/man7/cgroups.7.html).
Prova att begränsa minnesförbrukningen för `stress -m`.

1. (Avancerad) Kommandot `curl ipinfo.io` gör en HTTP-förfrågan och hämtar information om din publika IP.
Öppna [Wireshark](https://www.wireshark.org/) och försök fånga upp förfrågnings- och svarspaketen som `curl` skickar och tar emot.
(Tips: använd filtret `http` för att bara se HTTP-paket.)
