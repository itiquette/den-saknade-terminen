---
layout: lecture
title: "Felsökning och profilering"
description: >
  Lär dig hur du felsöker program med loggning och felsökningsverktyg, och hur du profilerar kod för prestanda.
thumbnail: /static/assets/thumbnails/2026/lec4.png
date: 2026-01-15
ready: true
panopto: "https://mit.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=a72c48e3-5eb2-46fa-aa03-b3b700e1ca8d"
video:
  aspect: 56.25
  id: 8VYT9TcUmKs
---

En gyllene regel inom programmering är att kod inte gör det du förväntar dig att den ska göra, utan det du säger åt den att göra.
Att överbrygga den luckan kan ibland vara ganska svårt.
I föreläsningen går vi igenom användbara tekniker för att hantera felaktig och resurshungrig kod: felsökning och profilering.

# Felsökning (debugging)

## Printf-felsökning och loggning

> "Det effektivaste felsökningsverktyget är fortfarande eftertanke, i kombination med välplacerade utskriftssatser" — Brian Kernighan, _Unix for Beginners_.

Ett sätt att felsöka ett program är att lägga till utskriftssatser kring där du upptäckt problemet, och fortsätta iterera tills du har tillräcklig information för att förstå vad som orsakar felet.

Ett andra sätt är att använda loggning i ditt program, i stället för ad hoc-utskriftssatser.
Loggning är i princip "utskrift med mer omsorg", och sker vanligtvis med ett loggningsramverk som har inbyggt stöd för saker som:

- möjligheten att styra loggarna (eller delmängder av loggarna) till andra utmatningsplatser,
- att sätta allvarlighetsnivåer (som INFO, DEBUG, WARN, ERROR, etc.) och filtrera utdata utifrån dessa,
- stöd för strukturerad loggning av data kopplad till loggposter, som sedan kan extraheras enklare i efterhand.

Loggsatser lägger du ofta in proaktivt medan du programmerar så att datan du behöver för att felsöka redan kan finnas där.
Och när du väl hittat och rättat ett problem med utskriftssatser är det ofta värt att konvertera dessa utskrifter till riktiga loggsatser innan du tar bort dem.
På så sätt har du redan den diagnostiska information du behöver om liknande programfel uppstår i framtiden, utan att behöva ändra koden.

> **Loggar från tredjepart**: Många program stödjer flaggan `-v` eller `--verbose` för att skriva ut mer information när de körs.
Det kan vara användbart för att upptäcka varför ett visst kommando misslyckas.
Vissa tillåter till och med att flaggan upprepas för mer detaljer.
När du felsöker problem med tjänster (databaser, webbservrar, etc.), kontrollera deras loggar, ofta i `/var/log/` på Linux.
Använd `journalctl -u <service>` för att visa loggar för systemd-tjänster.
För tredjepartsbibliotek, kontrollera om de stödjer felsökningsloggning via miljövariabler eller konfiguration.

## Felsökare

Printf-felsökning fungerar bra när du vet vad du ska skriva ut och enkelt kan modifiera och köra om koden.
Felsökare blir värdefulla när du inte vet vilken information du behöver, när programfelet bara visar sig under svårreproducerade förhållanden, eller när det är dyrt att modifiera och starta om programmet (långa uppstartstider, komplexa tillstånd att återskapa, etc.).

Felsökare är program som låter dig interagera med programmets körning, och låter dig:

- Stoppa körningen när den når en viss rad.
- Stega en instruktion i taget.
- Inspektera variabelvärden efter en krasch.
- Villkorligt stoppa körningen när ett visst villkor uppfylls.
- Och många fler avancerade funktioner.

De flesta programmeringsspråk stödjer (eller kommer med) någon form av felsökare.
De mest mångsidiga är **allmänna felsökare** som [`gdb`](https://www.gnu.org/software/gdb/) (GNU Debugger) och [`lldb`](https://lldb.llvm.org/) (LLVM Debugger), som kan felsöka vilken nativ binär som helst. Många språk har också **språkspecifika felsökare** som integrerar tätare med körmiljön (som Pythons pdb eller Javas jdb).

`gdb` är den faktiska standardfelsökaren för C, C++, Rust och andra kompilerade språk.
Den låter dig undersöka i princip vilken process som helst och se dess aktuella maskintillstånd: register, stack, programräknare och mer.

Några användbara GDB-kommandon:

- `run` - Starta programmet
- `b {function}` eller `b {file}:{line}` - Sätt en brytpunkt
- `c` - Fortsätt körning
- `step` / `next` / `finish` - Stega in / stega över / stega ut
- `p {variable}` - Skriv ut värdet på variabel
- `bt` - Visa backtrace (anropsstack)
- `watch {expression}` - Bryt när värdet ändras

> Överväg att använda GDB:s TUI-läge (`gdb -tui` eller tryck `Ctrl-x a` i GDB) för delad skärm med källkod bredvid kommandoprompten.

### Inspelnings-/uppspelningsfelsökning

Några av de mest frustrerande programfelen är så kallade _Heisenfel_ (_Heisenbugs_): programfel som verkar försvinna eller ändra beteende när du försöker observera dem.
Kapplöpningsproblem (race conditions), tidsberoende programfel och problem som bara dyker upp under vissa systemförhållanden tillhör den kategorin.
Traditionell felsökning är ofta värdelös här eftersom nästa körning ger annat beteende (t.ex. kan utskriftssatser sakta ner koden så mycket att kapplöpningen inte längre händer).

**Inspelnings-/uppspelningsfelsökning** (record-replay) löser detta genom att spela in ett programs körning och låta dig spela upp den deterministiskt så många gånger du behöver.
Ännu bättre är att du kan gå _baklänges_ i körningen för att hitta exakt var något gick fel.

[rr](https://rr-project.org/) är ett kraftfullt verktyg för Linux som spelar in programkörning och tillåter deterministisk uppspelning med fulla felsökningsmöjligheter.
Det fungerar med GDB, så du kan redan gränssnittet.

Grundläggande användning:

```bash
# Spela in en programkörning
rr record ./my_program

# Spela upp inspelningen (öppnar GDB)
rr replay
```

Magin sker under uppspelning.
Eftersom körningen är deterministisk kan du använda kommandon för **omvänd felsökning** (reverse debugging):

- `reverse-continue` (`rc`) - Kör baklänges tills en brytpunkt nås
- `reverse-step` (`rs`) - Stega baklänges en rad
- `reverse-next` (`rn`) - Stega baklänges och hoppa över funktionsanrop
- `reverse-finish` - Kör baklänges tills du går in i aktuell funktion

Det är otroligt kraftfullt för felsökning.
Säg att du har en krasch, i stället för att gissa var programfelet är och sätta brytpunkter kan du:

1. Köra till kraschen.
2. Inspektera det korrupta tillståndet.
3. Sätta en watchpoint på den korrupta variabeln.
4. Köra `reverse-continue` för att hitta exakt var den blev korrupt.

**När du ska använda rr:**
- Instabila tester som misslyckas sporadiskt.
- Kapplöpningsproblem och trådningsfel.
- Krascher som är svåra att reproducera.
- Alla programfel där du önskar att du kunde "gå tillbaka i tiden".

> Obs: rr fungerar bara på Linux och kräver hårdvaruprestandaräknare.
Det fungerar inte i VM:ar som inte exponerar dessa räknare, till exempel på de flesta AWS EC2-instanser, och stödjer inte GPU-åtkomst. För macOS, kolla in [Warpspeed](https://warpspeed.dev/).

> **rr och samtidighet**: Eftersom rr spelar in körningen deterministiskt serialiserar det trådschemaläggning.
Det innebär att vissa kapplöpningsproblem kanske inte visar sig under rr om de beror på specifika tidförhållanden. rr är fortfarande användbart för att felsöka kapplöpningsproblem, när du väl fångat en felande körning kan du spela upp den tillförlitligt, men du kan behöva flera inspelningsförsök för att fånga ett sporadiskt programfel.
För programfel utan samtidighet glänser rr mest: du kan alltid reproducera exakt körning och använda omvänd felsökning för att spåra korruption.

## Spårning av systemanrop

Ibland behöver du förstå hur programmet interagerar med operativsystemet.
Program gör [systemanrop](https://en.wikipedia.org/wiki/System_call) för att begära tjänster från kärnan, öppna filer, allokera minne, skapa processer och mer.
Att spåra dessa anrop kan avslöja varför ett program hänger sig, vilka filer det försöker komma åt eller var det spenderar tid på att vänta.

### strace (Linux) och dtruss (macOS)

[`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) låter dig observera varje systemanrop ett program gör:

```bash
# Spåra alla systemanrop
strace ./my_program

# Spåra bara filrelaterade anrop
strace -e trace=file ./my_program

# Följ barnprocesser (viktigt för program som startar andra program)
strace -f ./my_program

# Spåra en redan körande process
strace -p <PID>

# Visa tidsinformation
strace -T ./my_program
```

> På macOS och BSD, använd [`dtruss`](https://www.manpagez.com/man/1/dtruss/) (som kapslar `dtrace`) för liknande funktionalitet.

> För djupdykningar i `strace`, kolla Julia Evans utmärkta [strace-zine](https://jvns.ca/strace-zine-unfolded.pdf).

### bpftrace och eBPF

[eBPF](https://ebpf.io/) (extended Berkeley Packet Filter) är en kraftfull Linux-teknik som låter isolerade program köras i en sandlåda i kärnan.
[`bpftrace`](https://github.com/iovisor/bpftrace) ger en högnivåsyntax för att skriva eBPF-program.
Det är godtyckliga program som körs i kärnan och har därför stor uttryckskraft (men också en något klumpig awk-liknande syntax).
Det vanligaste användningsfallet är att undersöka vilka systemanrop som anropas, inklusive aggregeringar (som antal eller latensstatistik) eller introspektion (eller till och med filtrering på) systemanropsargument.

```bash
# Spåra filöppningar i hela systemet (skrivs ut direkt)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# Räkna systemanrop per namn (skriver ut sammanfattning med Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

Du kan också skriva eBPF-program direkt i C med en verktygskedja som [`bcc`](https://github.com/iovisor/bcc), som också levereras med [många praktiska verktyg](https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html) som `biosnoop` för att skriva ut latensfördelningar för diskoperationer eller `opensnoop` för att skriva ut alla öppnade filer.

Där `strace` är användbart eftersom det är lätt att "bara komma igång", är `bpftrace` verktyget du ska ta till när du behöver lägre belastning, vill spåra genom kärnfunktioner eller behöver någon form av aggregering.
Notera att `bpftrace` måste köras som `root`, och att det i allmänhet övervakar hela kärnan, inte bara en viss process.
För att rikta in dig på ett specifikt program kan du filtrera på kommandonamn eller PID:

```bash
# Filtrera på kommandonamn (skriver ut sammanfattning med Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# Spåra ett specifikt kommando från start med -c (cpid = barnets PID)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la'
```

Flaggan `-c` kör det angivna kommandot och sätter `cpid` till dess PID, vilket är användbart för att spåra ett program från det ögonblick det startar.
När det spårade kommandot avslutas skriver bpftrace ut de aggregerade resultaten.

### Nätverksfelsökning

För nätverksproblem låter [`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) och [Wireshark](https://www.wireshark.org/) dig fånga och analysera nätverkspaket:

```bash
# Fånga paket på port 80
sudo tcpdump -i any port 80

# Fånga och spara till fil för analys i Wireshark
sudo tcpdump -i any -w capture.pcap
```

Vid HTTPS-trafik är datan krypterad, vilket gör att tcpdump inte kan visa innehållet.
Verktyg som [mitmproxy](https://mitmproxy.org/) kan agera avlyssnande proxy för att inspektera krypterad trafik.
Webbläsarens utvecklarverktyg (Network-fliken) är ofta enklaste sättet att felsöka HTTPS-förfrågningar från webbapplikationer, de visar dekrypterad data för begäran och svar, headers och tidsmätning.

## Minnesfelsökning

Minnesfel, buffertöverskridningar, use-after-free, minnesläckor, är bland de farligaste och svåraste att felsöka.
De kraschar ofta inte direkt utan korruptar minne på sätt som orsakar problem långt senare.

### Sanitizers (saneringsverktyg)

Ett sätt att hitta minnesfel är att använda **sanitizers**, vilket är kompilatorfunktioner som instrumenterar koden för att upptäcka fel vid körning.
Den mycket använda **AddressSanitizer (ASan)** upptäcker till exempel:
- Buffertöverskridningar (stack, heap och globalt).
- Use-after-free.
- Use-after-return.
- Minnesläckor.

```bash
# Kompilera med AddressSanitizer
gcc -fsanitize=address -g program.c -o program
./program
```

Det finns flera användbara sanitizers:

- **ThreadSanitizer (TSan)**: Upptäcker datakapplöpning i multitrådad kod (`-fsanitize=thread`)
- **MemorySanitizer (MSan)**: Upptäcker läsningar av oinitierat minne (`-fsanitize=memory`)
- **UndefinedBehaviorSanitizer (UBSan)**: Upptäcker odefinierat beteende som heltalsöverspill (`-fsanitize=undefined`)

Sanitizers kräver omkompilering men är tillräckligt snabba för CI-pipelines och vanlig utveckling.

### Valgrind: när du inte kan omkompilera

[Valgrind](https://valgrind.org/) kör i stället programmet i något som liknar en virtuell maskin för att upptäcka minnesfel.
Det är långsammare än sanitizers men kräver ingen omkompilering:

```bash
valgrind --leak-check=full ./my_program
```

Använd Valgrind när:
- Du inte har källkoden.
- Du inte kan omkompilera (tredjepartsbibliotek).
- Du behöver specifika verktyg som inte finns som sanitizers.

Valgrind är faktiskt en väldigt kraftfull kontrollerad körmiljö, och vi kommer se mer av den senare när vi kommer till profilering.

## AI för felsökning

Stora språkmodeller har blivit förvånansvärt användbara felsökningsassistenter.
De är särskilt bra på vissa felsökningsuppgifter som kompletterar traditionella verktyg.

**Där LLM:er är starka:**

- **Förklara kryptiska felmeddelanden**: Kompilatorfel, särskilt från C++-templates eller Rusts lånekontroll (borrow checker), kan vara ökänt kryptiska.
  LLM:er kan översätta dem till vanlig svenska/engelska och föreslå fixar.

- **Navigera språk- och abstraktionsgränser**: Om du felsöker ett problem som spänner över flera språk (säg ett programfel i ett C-bibliotek som visar sig via en Python-binding), kan LLM:er hjälpa dig navigera lagren.
  De är särskilt bra på att förstå FFI-gränser, problem i byggsystem och felsökning över språkgränser (t.ex. mitt program ger fel, men jag tror det beror på ett programfel i ett av mina beroenden).

- **Koppla symptom till grundorsak**: "Mitt program fungerar men använder 10 gånger mer minne än väntat" är typen av diffust symptom som LLM:er kan hjälpa att undersöka genom att föreslå troliga orsaker och vad du ska titta efter.

- **Analysera kraschdumpar och stackspår**: Klistra in ett stackspår och fråga vad som kan ha orsakat det.

> **Obs om felsökningssymboler**: För meningsfulla stackspår och felsökning, se till att dina binärer (och länkade bibliotek) kompileras med felsökningssymboler (flaggan `-g`).
Felsökningsinformation lagras typiskt i DWARF-format.
Dessutom gör kompilering med frame pointers (`-fno-omit-frame-pointer`) stackspår mer tillförlitliga, särskilt för profileringsverktyg.
Utan detta kan stackspår bara visa minnesadresser eller vara ofullständiga.
Det spelar större roll för nativt kompilerade program (C++, Rust) än för Python eller Java.

**Begränsningar att ha i åtanke:**
- LLM:er kan hitta på förklaringar som låter trovärdiga men är felaktiga.
- De kan föreslå fixar som maskerar programfelet i stället för att lösa det.
- Verifiera alltid förslag med riktiga felsökningsverktyg.
- De fungerar bäst som ett komplement till, inte en ersättning för, förståelse av din kod.

> Det skiljer sig från de [generella AI-kodningsförmågorna]({{ '/2026/development-environment/#ai-powered-development' | relative_url }}) som tas upp i föreläsningen om utvecklingsmiljö.
Här pratar vi specifikt om att använda LLM:er som ett hjälpmedel vid felsökning.

# Profilering

Även om koden funktionellt beter sig som förväntat kanske det inte räcker om den samtidigt slukar all CPU eller allt minne.
Algoritmkurser lär ofta ut big _O_-notation men inte hur man hittar flaskhalsar i program.
Eftersom [för tidig optimering är roten till allt ont](https://wiki.c2.com/?PrematureOptimization) bör du lära dig om profileringsverktyg och övervakningsverktyg.
De hjälper dig förstå vilka delar av programmet som tar mest tid och/eller resurser så att du kan fokusera optimering där den spelar roll.

## Tidmätning

Det enklaste sättet att mäta prestanda är att mäta tid.
I många scenarier räcker det att bara skriva ut tiden som koden tog mellan två punkter.

Men faktisk förfluten tid (wall clock) kan vara missvisande eftersom datorn kan köra andra processer samtidigt eller vänta på händelser.
Kommandot `time` skiljer mellan _Real_, _User_ och _Sys_ tid:

- **Real** - Förfluten tid från start till slut, inklusive väntetid.
- **User** - Tid som CPU:n spenderar på användarkod.
- **Sys** - Tid som CPU:n spenderar på kärnkod.

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s
```

Här tog förfrågan nästan 300 millisekunder (real tid) men bara 107 ms CPU-tid (user + sys).
Resten var väntan på nätverket.

## Resursövervakning

Ibland är första steget i att analysera programmets prestanda att förstå faktisk resursförbrukning.
Program kör ofta långsamt när de är resursbegränsade.

- **Allmän övervakning**: [`htop`](https://htop.dev/) är en förbättrad version av `top` som visar olika statistik för processer som körs.
  Användbara kortkommandon: `<F6>` för att sortera processer, `t` för att visa trädhierarki, `h` för att växla trådar.
  Det finns också [`btop`](https://github.com/aristocratos/btop) som övervakar _mycket_ mer.

- **I/O-operationer**: [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) visar live-information om I/O-användning.

- **Minnesanvändning**: [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) visar totalt ledigt och använt minne.

- **Öppna filer**: [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) listar filinformation om filer öppnade av processer.
  Användbart för att se vilken process som öppnat en specifik fil.

- **Nätverksanslutningar**: [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) låter dig övervaka nätverksanslutningar.
  Ett vanligt användningsfall är att ta reda på vilken process som använder en viss port: `ss -tlnp | grep :8080`.

- **Nätverksanvändning**: [`nethogs`](https://github.com/raboof/nethogs) och [`iftop`](https://pdw.ex-parrot.com/iftop/) är bra interaktiva CLI-verktyg för att övervaka nätverksanvändning per process.

## Visualisering av prestandadata

Människor ser mönster i grafer betydligt snabbare än i tabeller med siffror.
När du analyserar prestanda avslöjar diagram ofta trender, toppar och avvikelser som är osynliga i rådata.

**Gör data visualiserbar**: När du lägger till utskrifts- eller loggsatser för felsökning, överväg att formatera utdata så att den enkelt kan visas i diagram senare.
En enkel tidsstämpel och ett värde i CSV-format (`1705012345,42.5`) är betydligt enklare att rita diagram av än en fullständig mening.
JSON-strukturerade loggar kan också tolkas och visualiseras med minimal ansträngning.
Med andra ord, logga din data [på ett välstrukturerat sätt](https://vita.had.co.nz/papers/tidy-data.pdf).

**Snabb diagramritning med gnuplot**: För enkel diagramritning från kommandoraden kan [`gnuplot`](http://www.gnuplot.info/) skapa grafer direkt från datafiler:

```bash
# Rita diagram av en enkel CSV med tidsstämpel,värde
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines"
```

**Iterativ utforskning med matplotlib och ggplot2**: För djupare analys möjliggör Pythons [`matplotlib`](https://matplotlib.org/) och R:s [`ggplot2`](https://ggplot2.tidyverse.org/) iterativ utforskning.
Till skillnad från engångsdiagram låter dessa verktyg dig snabbt skära och transformera data för att undersöka hypoteser. ggplot2:s uppdelade diagram är särskilt kraftfulla, du kan dela ett dataset över flera deldiagram per kategori (t.ex. latens per ändpunkt eller tid på dygnet) för att få fram mönster som annars skulle döljas.

**Exempel på användningsfall:**
- Att rita upp svarsfördröjning över tid avslöjar återkommande fördröjningar (skräpsamling, cron-jobb, trafikmönster) som råa percentiler döljer.
- Att visualisera insättningstider för en växande datastruktur kan avslöja algoritmisk komplexitet, en graf över vektorinsättningar visar typiska toppar när den underliggande arrayen dubblas.
- Att dela upp mätvärden över olika dimensioner (förfrågningstyp, användarkohort, server) avslöjar ofta att ett "systemomfattande" problem i själva verket är isolerat till en kategori.

## CPU-profilerare

Oftast när folk säger _profilerare_ menar de _CPU-profilerare_.
Det finns två huvudtyper:

- **Spårningsprofilerare** behåller en logg över varje funktionsanrop programmet gör.
- **Samplingsprofilerare** provar programmet med jämna mellanrum (vanligen varje millisekund) och spelar in programmets stack.

Samplingprofilering har lägre belastning och är generellt att föredra i produktion.

### perf: samplingsprofileraren

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) är standardprofileraren på Linux.
Den kan profilera vilket program som helst utan omkompilering:

`perf stat` ger en snabb överblick över var tiden spenderas:

```bash
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches
```

Profileringsutdata för verkliga program innehåller ofta stora mängder information.
Människor är visuella och ganska dåliga på att läsa stora mängder siffror.
[Flamdiagram (flame graph)](https://www.brendangregg.com/flamegraphs.html) är en visualisering som gör profileringsdata betydligt enklare att förstå.

Ett flamdiagram visar en hierarki av funktionsanrop längs Y-axeln och tidsåtgång proportionellt mot X-axeln.
De är interaktiva, du kan klicka för att zooma in i specifika delar av programmet.

[![Flamdiagram](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

För att generera ett flamdiagram från `perf`-data:

```bash
# Spela in profilering
perf record -g ./my_program

# Generera flamdiagram (kräver flamegraph-skript)
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

> Överväg att använda [Speedscope](https://www.speedscope.app/) för en interaktiv webbaserad flamdiagramvisare, eller [Perfetto](https://perfetto.dev/) för omfattande systemnivåanalys.

### Valgrinds Callgrind: spårningsprofilerare

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html) är ett profileringsverktyg som spelar in anropshistorik och instruktionsantal för programmet.
Till skillnad från samplingsprofilerare ger den exakta anropsantal och kan visa relationen mellan anropande och anropade funktioner:

```bash
# Kör med callgrind
valgrind --tool=callgrind ./my_program

# Analysera med callgrind_annotate (text) eller kcachegrind (GUI)
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>
```

Callgrind är långsammare än samplingsprofilerare men ger exakta anropsantal och kan valfritt simulera cache-beteende (med `--cache-sim=yes`) om du behöver den informationen.

> Om du använder ett särskilt språk kan det finnas mer specialiserade profilerare.
Till exempel har Python [`cProfile`](https://docs.python.org/3/library/profile.html) och [`py-spy`](https://github.com/benfred/py-spy), Go har [`go tool pprof`](https://pkg.go.dev/cmd/pprof), och Rust har [`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph) (som faktiskt fungerar för alla kompilerade program!).

## Minnesprofilerare

Minnesprofilerare hjälper dig förstå hur programmet använder minne över tid och hitta minnesläckor.

### Valgrinds Massif

[`massif`](https://valgrind.org/docs/manual/ms-manual.html) profilerar heap-minnesanvändning:

```bash
valgrind --tool=massif ./my_program
ms_print massif.out.<pid>
```

Det visar heap-användning över tid och hjälper dig identifiera minnesläckor och överdriven allokering.

> För Python ger [`memory-profiler`](https://pypi.org/project/memory-profiler/) rad-för-rad-information om minnesanvändning.

## Prestandajämförelse

När du behöver jämföra prestanda mellan olika implementationer eller verktyg är [`hyperfine`](https://github.com/sharkdp/hyperfine) utmärkt för att prestandatesta kommandoradsprogram:

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

> För webbutveckling har webbläsarens utvecklarverktyg utmärkta profilerare.
Se dokumentationen för [Firefox Profiler](https://profiler.firefox.com/docs/) och [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools).

# Övningar

## Felsökning

1. **Felsök en sorteringsalgoritm**: Följande pseudokod implementerar merge sort men innehåller ett programfel.
   Implementera den i ett språk du väljer, och använd sedan en felsökare (`gdb`, `lldb`, `pdb` eller din IDE:s felsökare) för att hitta och åtgärda programfelet.

   ```
   function merge_sort(arr):
       if length(arr) <= 1:
           return arr
       mid = length(arr) / 2
       left = merge_sort(arr[0..mid])
       right = merge_sort(arr[mid..end])
       return merge(left, right)

   function merge(left, right):
       result = []
       i = 0, j = 0
       while i < length(left) AND j < length(right):
           if left[i] <= right[j]:
               append result, left[i]
               i = i + 1
           else:
               append result, right[i]
               j = j + 1
       append remaining elements from left and right
       return result
   ```

   Testvektor: `merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` ska returnera `[1, 1, 2, 3, 4, 5, 6, 9]`.
   Använd brytpunkter och stega genom merge-funktionen för att hitta var felaktigt element väljs.

1. Installera [`rr`](https://rr-project.org/) och använd omvänd felsökning (reverse debugging) för att hitta ett korruptionsfel.
   Spara detta program som `corruption.c`:

   ```c
   #include <stdio.h>

   typedef struct {
       int id;
       int scores[3];
   } Student;

   Student students[2];

   void init() {
       students[0].id = 1001;
       students[0].scores[0] = 85;
       students[0].scores[1] = 92;
       students[0].scores[2] = 78;

       students[1].id = 1002;
       students[1].scores[0] = 90;
       students[1].scores[1] = 88;
       students[1].scores[2] = 95;
   }

   void curve_scores(int student_idx, int curve) {
       for (int i = 0; i < 4; i++) {
           students[student_idx].scores[i] += curve;
       }
   }

   int main() {
       init();
       printf("=== Starttillstånd ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       curve_scores(0, 5);

       printf("\n=== Efter kurvanpassning ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       if (students[1].id != 1002) {
           printf("\nFEL: Student 1:s ID blev korrupt! Förväntade 1002, fick %d\n",
                  students[1].id);
           return 1;
       }
       return 0;
   }
   ```

   Kompilera med `gcc -g corruption.c -o corruption` och kör programmet.
   Student 1:s ID blir korrupt, men korruptionen händer i en funktion som bara rör student 0. Använd `rr record ./corruption` och `rr replay` för att hitta boven.
   Sätt en watchpoint på `students[1].id` och använd `reverse-continue` efter korruptionen för att hitta exakt vilken kodrad som skrev över värdet.

1. Felsök ett minnesfel med AddressSanitizer.
   Spara detta som `uaf.c`:

   ```c
   #include <stdlib.h>
   #include <string.h>
   #include <stdio.h>

   int main() {
       char *greeting = malloc(32);
       strcpy(greeting, "Hej, världen!");
       printf("%s\n", greeting);

       free(greeting);

       greeting[0] = 'J';
       printf("%s\n", greeting);

       return 0;
   }
   ```

   Kompilera och kör först utan sanitizers: `gcc uaf.c -o uaf && ./uaf`.
   Det kan verka fungera.
   Kompilera nu med AddressSanitizer: `gcc -fsanitize=address -g uaf.c -o uaf && ./uaf`.
   Läs felrapporten.
   Vilket programfel hittar ASan?
   Fixa problemet den identifierar.

1. Använd `strace` (Linux) eller `dtruss` (macOS) för att spåra systemanropen som utförs av ett kommando som `ls -l`.
   Vilka systemanrop utförs?
   Prova att spåra ett mer komplext program och se vilka filer det öppnar.

1. Använd en LLM för att hjälpa till att felsöka ett kryptiskt felmeddelande.
   Prova att kopiera ett kompilatorfel (särskilt från C++-templates eller Rust) och be om förklaring och fix.
   Prova att klistra in delar av utdata från `strace` eller AddressSanitizer.

## Profilering

1. Använd `perf stat` för att få grundläggande prestandastatistik för ett valfritt program.
   Vad betyder de olika räknarna?

1. Profilera med `perf record`.
   Spara detta som `slow.c`:

   ```c
   #include <math.h>
   #include <stdio.h>

   double slow_computation(int n) {
       double result = 0;
       for (int i = 0; i < n; i++) {
           for (int j = 0; j < 1000; j++) {
               result += sin(i * j) * cos(i + j);
           }
       }
       return result;
   }

   int main() {
       double r = 0;
       for (int i = 0; i < 100; i++) {
           r += slow_computation(1000);
       }
       printf("Resultat: %f\n", r);
       return 0;
   }
   ```

   Kompilera med felsökningssymboler: `gcc -g -O2 slow.c -o slow -lm`.
   Kör `perf record -g ./slow`, sedan `perf report` för att se var tid spenderas.
   Prova att generera ett flamdiagram med flamegraph-skripten.

1. Använd `hyperfine` för att prestandatesta två olika implementationer av samma uppgift (t.ex. `find` vs `fd`, `grep` vs `ripgrep`, eller två versioner av din egen kod).

1. Använd `htop` för att övervaka systemet medan du kör ett resursintensivt program.
   Prova att använda `taskset` för att begränsa vilka CPU:er en process kan använda: `taskset --cpu-list 0,2 stress -c 3`.
   Varför använder inte `stress` tre CPU:er?

1. Ett vanligt problem är att en port du vill lyssna på redan används av en annan process.
   Lär dig hitta den processen: kör först `python -m http.server 4444` för att starta en minimal webbserver på port 4444. Kör i en separat terminal `ss -tlnp | grep 4444` för att hitta processen.
   Avsluta den med `kill <PID>`.
