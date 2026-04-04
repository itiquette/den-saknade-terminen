---
layout: lecture
title: "Paketera och distribuera kod"
description: >
  Lär dig om projektpaketering, miljöer, versionshantering och distribution av bibliotek, applikationer och tjänster.
thumbnail: /static/assets/thumbnails/2026/lec6.png
date: 2026-01-20
ready: true
video:
  aspect: 56.25
  id: KBMiB-8P4Ns
---

Att få kod att fungera som tänkt är svårt.
Att få samma kod att köra på en annan maskin än din egen är ofta ännu svårare.

Att distribuera kod innebär att ta koden du skrev och omvandla den till en användbar form som någon annan kan köra utan din dators exakta miljö.
Att distribuera kod kan se ut på många sätt och beror på val av programmeringsspråk, systembibliotek, operativsystem och många andra faktorer.
Det beror också på vad du bygger; ett programbibliotek, ett kommandoradsverktyg och en webbtjänst har olika krav och driftsättningssteg.
Oavsett finns ett gemensamt mönster i alla dessa scenarier: vi måste definiera vad slutprodukten är --- det vill säga en artefakt --- och vilka antaganden den gör om miljön runt omkring.

I föreläsningen går vi igenom:

- [Beroenden och miljöer](#dependencies--environments)
- [Artefakter och paketering](#artifacts--packaging)
- [Utgåvor och versionering](#utgavor-och-versionering)
- [Reproducerbarhet](#reproducibility)
- [VM:ar och containers](#vms--containers)
- [Konfiguration](#configuration)
- [Tjänster och orkestrering](#services--orchestration)
- [Publicering](#publishing)

Vi förklarar dessa koncept med exempel från Python-ekosystemet, eftersom konkreta exempel hjälper förståelsen.
Verktygen är annorlunda i andra språks ekosystem, men koncepten är till stor del desamma.

# Beroenden och miljöer {#dependencies--environments}

I modern programvaruutveckling är abstraktionslager överallt.
Program flyttar naturligt över logik till andra bibliotek eller tjänster.
Det introducerar dock ett beroendeförhållande mellan ditt program och biblioteken det behöver för att fungera.
I Python gör vi till exempel ofta följande för att hämta innehållet på en webbsida:

```python
import requests

response = requests.get("https://missing.csail.mit.edu")
```

Men biblioteket `requests` följer inte med standardinstallationen av Python, så om vi försöker köra koden utan att ha `requests` installerat får vi ett fel från Python:

```console
$ python fetch.py
Traceback (most recent call last):
  File "fetch.py", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

För att göra biblioteket tillgängligt måste vi först köra `pip install requests` för att installera det.
`pip` är kommandoradsverktyget som Python-språket tillhandahåller för att installera paket.
Att köra `pip install requests` ger följande sekvens av steg:

1. Sök efter requests i Python Package Index ([PyPI](https://pypi.org/)).
1. Sök efter rätt artefakt för plattformen vi kör på.
1. Lös beroenden --- biblioteket `requests` beror själv på andra paket, så installeraren måste hitta kompatibla versioner av alla transitiva beroenden och installera dem först.
1. Ladda ner artefakter, packa upp och kopiera filer till rätt platser i filsystemet.

```console
$ pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Collecting charset-normalizer<4,>=2
  Downloading charset_normalizer-3.4.0-cp311-cp311-manylinux_x86_64.whl (142 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.10-py3-none-any.whl (70 kB)
Collecting urllib3<3,>=1.21.1
  Downloading urllib3-2.2.3-py3-none-any.whl (126 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2024.8.30-py3-none-any.whl (167 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2024.8.30 charset-normalizer-3.4.0 idna-3.10 requests-2.32.3 urllib3-2.2.3
```

Här ser vi att `requests` har egna beroenden som `certifi` och `charset-normalizer`, och att de måste installeras innan `requests` kan installeras.
När det är installerat kan Python hitta biblioteket vid import.

```console
$ python -c 'import requests; print(requests.__path__)'
['/usr/local/lib/python3.11/dist-packages/requests']

$ pip list | grep requests
requests        2.32.3
```

Programmeringsspråk har olika verktyg, konventioner och arbetssätt för att installera och publicera bibliotek.
I vissa språk som Rust är verktygskedjan enhetlig --- `cargo` hanterar bygg, test, beroendehantering och publicering.
I andra som Python sker enhetligheten på specifikationsnivå --- i stället för ett enda verktyg finns standardiserade specifikationer som definierar hur paketering fungerar, vilket möjliggör flera konkurrerande verktyg för varje uppgift (`pip` vs [`uv`](https://docs.astral.sh/uv/), `setuptools` vs [`hatch`](https://hatch.pypa.io/) vs [`poetry`](https://python-poetry.org/)).
Och i vissa ekosystem som LaTeX levereras distributioner som TeX Live eller MacTeX med tusentals förinstallerade paket.

Att lägga till beroenden medför också beroendekonflikter.
Konflikter uppstår när program kräver inkompatibla versioner av samma beroende.
Om till exempel `tensorflow==2.3.0` kräver `numpy>=1.16.0,<1.19.0` och `pandas==1.2.0` kräver `numpy>=1.16.5`, så är alla versioner som uppfyller `numpy>=1.16.5,<1.19.0` giltiga.
Men om ett annat paket i projektet kräver `numpy>=1.19` har du en konflikt utan någon giltig version som uppfyller alla krav.

Denna situation --- där flera paket kräver ömsesidigt inkompatibla versioner av delade beroenden --- kallas ofta beroendekaos.
Ett sätt att hantera konflikter är att isolera varje programs beroenden i en egen miljö.
I Python skapar vi en virtuell miljö genom att köra:

```console
$ which python
/usr/bin/python
$ pwd
/home/missingsemester
$ python -m venv venv
$ source venv/bin/activate
$ which python
/home/missingsemester/venv/bin/python
$ which pip
/home/missingsemester/venv/bin/pip
$ python -c 'import requests; print(requests.__path__)'
['/home/missingsemester/venv/lib/python3.11/site-packages/requests']

$ pip list
Package Version
------- -------
pip     24.0
```

Du kan tänka på en miljö som en helt fristående version av språkets körmiljö med egna installerade paket.
Denna virtuella miljö, eller venv, isolerar installerade beroenden från den globala Python-installationen.
Det är god praxis att ha en virtuell miljö per projekt, som innehåller de beroenden projektet kräver.

> Även om många moderna operativsystem levereras med installerade körmiljöer för programmeringsspråk som Python, är det klokt att inte modifiera dessa installationer eftersom OS:et kan förlita sig på dem för egen funktionalitet.
Använd i stället separata miljöer.

I vissa språk definieras installationsprotokollet inte av ett verktyg utan som en specifikation.
I Python definierar [PEP 517](https://peps.python.org/pep-0517/) gränssnittet för byggsystem och [PEP 621](https://peps.python.org/pep-0621/) specificerar hur projektmetadata lagras i `pyproject.toml`.
Det har gjort det möjligt att förbättra `pip` och ta fram mer optimerade verktyg som `uv`.
För att installera `uv` räcker det att köra `pip install uv`.

Att använda `uv` i stället för `pip` följer samma gränssnitt men är betydligt snabbare:

```console
$ uv pip install requests
Resolved 5 packages in 12ms
Prepared 5 packages in 0.45ms
Installed 5 packages in 8ms
 + certifi==2024.8.30
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

> Vi rekommenderar starkt att använda `uv pip` i stället för `pip` när det är möjligt eftersom installationstiden minskar dramatiskt.

Utöver beroendeisolering låter miljöer dig också ha olika versioner av språkets körmiljö.

```console
$ uv venv --python 3.12 venv312
Using CPython 3.12.7
Creating virtual environment at: venv312

$ source venv312/bin/activate && python --version
Python 3.12.7

$ uv venv --python 3.11 venv311
Using CPython 3.11.10
Creating virtual environment at: venv311

$ source venv311/bin/activate && python --version
Python 3.11.10
```

Det hjälper när du behöver testa kod mot flera Python-versioner eller när ett projekt kräver en specifik version.

> I vissa språk får varje projekt automatiskt sin egen miljö för beroenden i stället för att du skapar den manuellt, men principen är densamma.
De flesta språk har i dag också en mekanism för att hantera flera språkversioner på samma system och sedan välja version per projekt.

# Artefakter och paketering {#artifacts--packaging}

I programvaruutveckling skiljer vi mellan källkod och artefakter.
Utvecklare skriver och läser källkod, medan artefakter är paketerade, distribuerbara utdata som produceras från källkoden --- redo att installeras eller driftsättas.
En artefakt kan vara så enkel som en kodfil som vi kör, och så komplex som en hel virtuell maskin som innehåller alla nödvändiga delar av en applikation.
Tänk på detta exempel där vi har en Python-fil `greet.py` i nuvarande katalog:

```console
$ cat greet.py
def greet(name):
    return f"Hej, {name}!"

$ python -c "from greet import greet; print(greet('World'))"
Hej, World!

$ cd /tmp
$ python -c "from greet import greet; print(greet('World'))"
ModuleNotFoundError: No module named 'greet'
```

Importen misslyckas när vi byter katalog eftersom Python bara söker moduler på specifika platser (nuvarande katalog, installerade paket och sökvägar i `PYTHONPATH`).
Paketering löser detta genom att installera koden på en känd plats.

I Python innebär paketering av ett bibliotek att producera en artefakt som paketinstallerare som `pip` eller `uv` kan använda för att installera relevanta filer.
Python-artefakter kallas _wheels_ och innehåller all nödvändig information för att installera ett paket: kodfiler, metadata om paketet (namn, version, beroenden) och instruktioner för var filer ska placeras i miljön.
Att bygga en artefakt kräver att vi skriver en projektfil (ofta kallad manifest) som specificerar projektets detaljer, nödvändiga beroenden, paketversion och annan information.
I Python använder vi `pyproject.toml` för detta.

> `pyproject.toml` är det moderna och rekommenderade sättet.
Även om äldre paketeringsmetoder som `requirements.txt` eller `setup.py` fortfarande stöds bör du föredra `pyproject.toml` när det går.

Här är en minimal `pyproject.toml` för ett bibliotek som också tillhandahåller ett kommandoradsverktyg:

```toml
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:cli"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

Biblioteket `typer` är ett populärt Python-paket för att skapa kommandoradsgränssnitt med minimalt upprepningsarbete.

Och motsvarande `greeting.py`:

```python
import typer


def greet(name: str) -> str:
    return f"Hej, {name}!"


def cli():
    typer.run(greet)


if __name__ == "__main__":
    cli()
```

Med denna fil kan vi nu bygga wheel-filen:

```console
$ uv build
Building source distribution...
Building wheel from source distribution...
Successfully built dist/greeting-0.1.0.tar.gz
Successfully built dist/greeting-0.1.0-py3-none-any.whl

$ ls dist/
greeting-0.1.0-py3-none-any.whl
greeting-0.1.0.tar.gz
```

Filen `.whl` är wheel-filen (ett zip-arkiv med en specifik struktur), och `.tar.gz` är en källdistribution för system som behöver bygga från källkod.

Du kan inspektera innehållet i en wheel för att se vad som paketeras:

```console
$ unzip -l dist/greeting-0.1.0-py3-none-any.whl
Archive:  dist/greeting-0.1.0-py3-none-any.whl
  Length      Date    Time    Name
---------  ---------- -----   ----
      150  2024-01-15 10:30   greeting.py
      312  2024-01-15 10:30   greeting-0.1.0.dist-info/METADATA
       92  2024-01-15 10:30   greeting-0.1.0.dist-info/WHEEL
        9  2024-01-15 10:30   greeting-0.1.0.dist-info/top_level.txt
      435  2024-01-15 10:30   greeting-0.1.0.dist-info/RECORD
---------                     -------
      998                     5 files
```

Om vi sedan ger denna wheel till någon annan kan de installera den genom att köra:

```console
$ uv pip install ./greeting-0.1.0-py3-none-any.whl
$ greet Alice
Hej, Alice!
```

Det installerar biblioteket vi byggde tidigare i deras miljö, inklusive kommandoradsverktyget `greet`.

Det finns begränsningar med detta tillvägagångssätt.
Om biblioteket beror på plattformsspecifika bibliotek, till exempel CUDA för GPU-acceleration, fungerar artefakten bara på system med dessa bibliotek installerade, och vi kan behöva bygga separata wheels för olika plattformar (Linux, macOS, Windows) och arkitekturer (x86, ARM).

Vid installation av programvara finns en viktig skillnad mellan installation från källkod och installation av en förbyggd binär.
Installation från källkod innebär att ladda ner originalkoden och kompilera den på din maskin --- detta kräver kompilator och byggverktyg, och kan ta lång tid för stora projekt.

Installation av en förbyggd binär innebär att ladda ner en artefakt som redan kompilerats av någon annan --- snabbare och enklare, men binären måste matcha din plattform och arkitektur.
Till exempel visar [ripgreps utgåvosida](https://github.com/BurntSushi/ripgrep/releases) förbyggda binärer för Linux (x86_64, ARM), macOS (Intel, Apple Silicon) och Windows.

# Utgåvor och versionering {#utgavor-och-versionering}

Kod byggs kontinuerligt men släpps i diskreta steg.
I programvaruutveckling finns en tydlig skillnad mellan utvecklings- och produktionsmiljöer.
Kod måste bevisas fungera i en utvecklingsmiljö innan den driftsätts i produktion.
Utgivningsprocessen omfattar många steg, inklusive testning, beroendehantering, versionering, konfiguration, driftsättning och publicering.

Mjukvarubibliotek är inte statiska utan utvecklas över tid med fixar och nya funktioner.
Vi spårar denna utveckling med diskreta versionsidentifierare som motsvarar bibliotekets tillstånd vid en viss tidpunkt.
Förändringar i ett biblioteks beteende kan vara allt från patchar som fixar icke-kritisk funktionalitet och nya funktioner som utökar funktionaliteten till ändringar som bryter bakåtkompatibilitet.
Ändringsloggar dokumenterar vilka ändringar en version introducerar --- dokument som utvecklare använder för att kommunicera ändringar i en ny utgåva.

Att hålla koll på pågående ändringar i varje beroende är dock opraktiskt, särskilt när vi tar hänsyn till transitiva beroenden --- alltså beroendenas beroenden.

> Du kan visualisera hela beroendeträdet för projektet med `uv tree`, som visar alla paket och deras transitiva beroenden i trädformat.

För att förenkla detta finns konventioner för versionssättning av programvara, och en av de vanligaste är [semantisk versionshantering](https://semver.org/) (SemVer).
Under semantisk versionshantering har en version formatet MAJOR.MINOR.PATCH där varje värde är ett heltal.
Kortversionen är att en uppgradering av:

- PATCH (t.ex. 1.2.3 → 1.2.4) bör bara innehålla buggfixar och vara helt bakåtkompatibel.
- MINOR (t.ex. 1.2.3 → 1.3.0) lägger till ny funktionalitet på ett bakåtkompatibelt sätt.
- MAJOR (t.ex. 1.2.3 → 2.0.0) signalerar brytande ändringar som kan kräva kodmodifieringar.

> Detta är en förenkling och vi uppmuntrar dig att läsa hela SemVer-specifikationen för att förstå till exempel varför en övergång från 0.1.3 till 0.2.0 kan ge brytande ändringar, eller vad 1.0.0-rc.1 betyder.
Python-paketering stödjer semantisk versionshantering inbyggt, så när vi specificerar versionskrav för beroenden kan vi använda olika uttryck.

I `pyproject.toml` har vi olika sätt att begränsa intervall av kompatibla versionsnummer för våra beroenden:

```toml
[project]
dependencies = [
    "requests==2.32.3",  # Exakt version - bara just den här versionen
    "click>=8.0",        # Minimiversion - 8.0 eller nyare
    "numpy>=1.24,<2.0",  # Intervall - minst 1.24 men lägre än 2.0
    "pandas~=2.1.0",     # Kompatibel utgåva - >=2.1.0 och <2.2.0
]
```

Versionsspecifikationer finns i många pakethanterare (npm, cargo, osv.) med varierande exakta betydelser.
Operatorn `~=` är Pythons operator för kompatibel utgåva --- `~=2.1.0` betyder "vilken version som helst kompatibel med 2.1.0", vilket motsvarar `>=2.1.0` och `<2.2.0`.
Det är ungefär ekvivalent med caret-operatorn (`^`) i npm och cargo, som följer SemVers kompatibilitetsbegrepp.

All programvara använder inte semantisk versionshantering.
Ett vanligt alternativ är Calendar Versioning (CalVer), där versioner baseras på utgivningsdatum i stället för semantisk betydelse.
Ubuntu använder till exempel versioner som `24.04` (april 2024) och `24.10` (oktober 2024).
CalVer gör det lätt att se hur gammal en utgåva är, men kommunicerar inget om kompatibilitet.
Slutligen är semantisk versionshantering inte ofelbar, och de ansvariga kan oavsiktligt introducera brytande ändringar i minor- eller patch-versioner.

# Reproducerbarhet {#reproducibility}

I modern programvaruutveckling vilar koden du skriver ovanpå många abstraktionslager.
Det omfattar språkets körmiljö, tredjepartsbibliotek, operativsystemet eller till och med hårdvaran.
Skillnader i något av dessa lager kan ändra kodens beteende eller till och med hindra den från att fungera som avsett.
Dessutom påverkar även skillnader i underliggande hårdvara din förmåga att leverera programvara.

Att låsa ett bibliotek till en specifik version innebär att använda en exakt version i stället för ett intervall, t.ex. `requests==2.32.3` i stället för `requests>=2.0`.

En del av jobbet för en pakethanterare är att ta hänsyn till alla begränsningar från beroenden --- inklusive transitiva beroenden --- och sedan producera en giltig lista av versioner som uppfyller alla begränsningar.
Den specifika listan av versioner kan sedan sparas i en fil för reproducerbarhet.
Dessa filer kallas låsfiler (_lock files_).

```console
$ uv lock
Resolved 12 packages in 45ms

$ cat uv.lock | head -20
version = 1
requires-python = ">=3.11"

[[package]]
name = "certifi"
version = "2024.8.30"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/...", hash = "sha256:..." }
wheels = [
    { url = "https://files.pythonhosted.org/...", hash = "sha256:..." },
]
...
```

En kritisk skillnad i beroendeversionering och reproducerbarhet är skillnaden mellan bibliotek och applikationer/tjänster.
Ett bibliotek är avsett att importeras och användas av annan kod som kan ha egna beroenden, så alltför strikta versionskrav kan orsaka konflikter med användarens andra beroenden.
Applikationer eller tjänster är däremot slutkonsumenter av programvaran och exponerar vanligtvis funktionalitet via användargränssnitt eller API, inte via programmeringsgränssnitt.
För bibliotek är det god praxis att ange versionsintervall för maximal kompatibilitet med det bredare paketekosystemet.
För applikationer säkerställer låsning till exakta versioner reproducerbarhet --- alla som kör applikationen använder exakt samma beroenden.

För projekt som kräver maximal reproducerbarhet kan verktyg som [Nix](https://nixos.org/) och [Bazel](https://bazel.build/) användas för hermetiska byggen.
Det betyder att all indata --- även kompilatorer, systembibliotek och själva byggmiljön --- är låst och innehållsadresserad.
Det garanterar bit-för-bit-identiska utdata oavsett när eller var bygget körs.

> Du kan till och med använda NixOS för att hantera hela datorinstallationen så att du enkelt kan sätta upp nya kopior av din miljö och hantera komplett konfiguration genom versionskontrollerade konfigurationsfiler.

En ständig spänning i programvaruutveckling är att nya programvaruversioner introducerar brytande ändringar, avsiktligt eller oavsiktligt, medan gamla versioner med tiden blir sårbara för säkerhetsproblem.
Vi kan hantera detta med CI-pipelines (vi ser mer i föreläsningen [Kodkvalitet och kontinuerlig integration]({{ '/2026/code-quality/' | relative_url }})) som testar applikationen mot nya programvaruversioner, och med automatisering för att upptäcka när nya beroendeversioner släpps, som [Dependabot](https://github.com/dependabot).

Även med CI-testning uppstår problem vid versionsuppgraderingar, ofta på grund av den oundvikliga skillnaden mellan utvecklings- och produktionsmiljöer.
I de fallen är bästa åtgärd att ha en återställningsplan, där versionsuppgraderingen återställs och en känd fungerande version driftsätts igen.

# VM:ar och containrar {#vms--containers}

När du börjar förlita dig på mer komplexa beroenden är det sannolikt att beroendena för din kod sträcker sig bortom vad pakethanteraren kan hantera.
En vanlig orsak är behovet av gränssnitt mot specifika systembibliotek eller hårdvarudrivrutiner.
I vetenskaplig beräkning och AI behöver program till exempel ofta specialiserade bibliotek och drivrutiner för att använda GPU-hårdvara.
Många systemnivåberoenden (GPU-drivrutiner, specifika kompilatorversioner, delade bibliotek som OpenSSL) kräver fortfarande systemomfattande installation.

Traditionellt löstes detta bredare beroendeproblem med virtuella maskiner (VM:ar).
VM:ar abstraherar hela datorn och ger en helt isolerad miljö med eget dedikerat operativsystem.
Ett modernare angreppssätt är containrar, som paketerar en applikation tillsammans med beroenden, bibliotek och filsystem, men delar värdens OS-kärna i stället för att virtualisera en hel dator.
Containrar är lättviktigare än VM:ar eftersom de delar kärna, vilket gör dem snabbare att starta och mer effektiva att köra.

Den mest populära containerplattformen är [Docker](https://www.docker.com/).
Docker introducerade ett standardiserat sätt att bygga, distribuera och köra containrar.
Under huven använder Docker containerd som körmiljö för containrar --- en industristandard som även verktyg som Kubernetes använder.

Att köra en container är enkelt.
För att till exempel köra en Python-interpreter i en container använder vi `docker run`.
Flaggorna `-it` gör containern interaktiv med terminalen.
När du avslutar stoppas containern.

```console
$ docker run -it python:3.12 python
Python 3.12.7 (main, Nov  5 2024, 02:53:25) [GCC 12.2.0] on linux
>>> print("Hej från insidan av en container!")
Hej från insidan av en container!
```

I praktiken kan ditt program bero på hela filsystemet.
För att hantera detta kan vi använda containeravbilder som skickar med applikationens hela filsystem som artefakt.
Containeravbilder skapas programmatiskt.
Med Docker specificerar vi exakta beroenden, systembibliotek och avbildningskonfiguration med Dockerfile-syntax:

```dockerfile
FROM python:3.12
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get install -y libpq-dev
RUN pip install numpy
RUN pip install pandas
COPY . /app
WORKDIR /app
RUN pip install .
```

En viktig skillnad: en Docker-**avbild** är den paketerade artefakten (som en mall), medan en **container** är en körande instans av avbilden.
Du kan köra flera containrar från samma avbild.
Avbilder byggs i lager, där varje instruktion (`FROM`, `RUN`, `COPY`, etc.) i en Dockerfile skapar ett nytt lager.
Docker cachelagrar dessa lager, så om du ändrar en rad i Dockerfile behöver bara det lagret och efterföljande lager byggas om.

Föregående Dockerfile har flera problem: den använder full Python-avbild i stället för slim-variant, kör separata `RUN`-kommandon som skapar onödiga lager, versioner är inte låsta, och den rensar inte pakethanterarens cache vilket skickar med onödiga filer.
Andra vanliga misstag omfattar att osäkert köra containrar som superanvändare och att av misstag baka in hemligheter i lager.

Här är en förbättrad version.

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*
COPY pyproject.toml uv.lock ./
RUN uv pip install --system -r uv.lock
COPY . /app
```

I föregående exempel ser vi att vi i stället för att installera `uv` från källkod kopierar den förbyggda binären från avbilden `ghcr.io/astral-sh/uv:latest`.
Det kallas byggarmönstret (eng. _builder pattern_).
Med detta mönster behöver vi inte skicka med alla verktyg som krävs för att kompilera koden, bara den slutliga binären som behövs för att köra applikationen (`uv` i detta fall).

Docker har viktiga begränsningar att känna till.
För det första är containeravbilder ofta plattformsspecifika --- en avbild byggd för `linux/amd64` körs inte nativt på `linux/arm64` (Apple Silicon Macs) utan emulering, vilket är långsamt.
För det andra kräver Docker-containrar en Linux-kärna, så på macOS och Windows kör Docker i praktiken en lättviktig Linux-VM under huven, vilket ger viss prestandaförlust. För det tredje är Dockers isolering svagare än VM:ars --- containrar delar värdens kärna, vilket är en säkerhetsrisk i miljöer med flera hyresgäster.

> Numera använder fler projekt också nix för att hantera även "systemomfattande" bibliotek och applikationer per projekt via [nix flakes](https://serokell.io/blog/practical-nix-flakes).

# Konfiguration {#configuration}

Mjukvara är i grunden konfigurerbar.
I föreläsningen om [kommandoradsmiljön]({{ '/2026/command-line-environment/' | relative_url }}) såg vi program som tar emot alternativ via flaggor, miljövariabler eller konfigurationsfiler (så kallade dotfiles).
Det gäller även mer komplexa applikationer, och det finns etablerade mönster för att hantera konfiguration i skala.
Programkonfiguration bör inte vara inbakad i koden utan tillhandahållas vid körning.
Två vanliga sätt är miljövariabler och konfigurationsfiler.

Här är ett exempel på en applikation som konfigureras via miljövariabler:

```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]  # Required - will raise if not set
```

En applikation kan också konfigureras via en konfigurationsfil (t.ex. ett Python-program som laddar konfiguration via `yaml.load`), `config.yaml`:

```yaml
database:
  url: "postgresql://localhost/myapp"
  pool_size: 5
server:
  host: "0.0.0.0"
  port: 8080
  debug: false
```

En bra tumregel för konfiguration är att samma kodbas ska kunna driftsättas till olika miljöer (utveckling, test och produktion) med endast konfigurationsändringar, aldrig kodändringar.

Bland konfigurationsalternativ finns ofta känslig data som API-nycklar.
Hemligheter måste hanteras varsamt för att undvika oavsiktlig exponering och får inte ingå i versionshantering.

# Tjänster och orkestrering {#services--orchestration}

Moderna applikationer existerar sällan isolerat.
En typisk webbapplikation kan behöva en databas för beständig lagring, en cache för prestanda, en meddelandekö för bakgrundsjobb och olika andra stödtjänster.
I stället för att paketera allt i en monolitisk applikation bryter moderna arkitekturer ofta ner funktionalitet i separata tjänster som kan utvecklas, driftsättas och skalas oberoende.

Som exempel, om vi avgör att applikationen kan tjäna på att använda cache, kan vi i stället för att bygga en egen lösning utnyttja etablerade lösningar som [Redis](https://redis.io/) eller [Memcached](https://memcached.org/).
Vi skulle kunna bädda in Redis i applikationens beroenden genom att bygga den i containern, men det innebär att harmonisera alla beroenden mellan Redis och vår applikation, vilket kan vara utmanande eller omöjligt.
I stället kan vi driftsätta varje applikation separat i sin egen container.
Det kallas ofta en mikrotjänstarkitektur där varje komponent körs som en oberoende tjänst som kommunicerar över nätverket, vanligtvis via HTTP-API:er.

[Docker Compose](https://docs.docker.com/compose/) är ett verktyg för att definiera och köra applikationer med flera containrar.
I stället för att hantera containrar individuellt deklarerar du alla tjänster i en enda YAML-fil och orkestrerar dem tillsammans.
Nu omfattar hela applikationen mer än en container:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - REDIS_URL=redis://cache:6379
    depends_on:
      - cache

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

Med `docker compose up` startar båda tjänsterna tillsammans, och webapplikationen kan ansluta till Redis med värdnamnet `cache` (Dockers interna DNS slår upp tjänstnamn automatiskt).
Docker Compose låter oss deklarera hur vi vill driftsätta en eller flera tjänster, och hanterar orkestreringen av att starta dem tillsammans, sätta upp nätverk mellan dem och hantera delade volymer för datapersistens.

För driftsättning i produktion vill du ofta att docker compose-tjänster startar automatiskt vid uppstart och startar om vid fel.
Ett vanligt tillvägagångssätt är att använda systemd för att hantera docker compose-driftsättning:

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
```

Den här systemd-enhetsfilen säkerställer att applikationen startar när systemet startar (efter att Docker är redo), och ger standardkommandon som `systemctl start myapp`, `systemctl stop myapp` och `systemctl status myapp`.

När driftsättningskraven blir mer komplexa --- med behov av skalning över flera maskiner, feltolerans när tjänster kraschar och hög tillgänglighet --- går organisationer över till mer avancerade containerorkestreringsplattformar som Kubernetes (k8s), som kan hantera tusentals containrar över kluster av maskiner.
Kubernetes har dock en brant inlärningskurva och betydande driftsmässig belastning, så det är ofta överdrivet för mindre projekt.

Denna uppsättning med flera containrar är delvis möjlig eftersom moderna tjänster kommunicerar via standardiserade API:er, särskilt REST-API:er över HTTP.
Till exempel, när ett program interagerar med en LLM-leverantör som OpenAI eller Anthropic skickar det under huven en HTTP-begäran till deras servrar och tolkar svaret:

```console
$ curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2023-06-01" \
    -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 256,
         "messages": [{"role": "user", "content": "Explain containers vs VMs in one sentence."}]}'
```

# Publicering {#publishing}

När du har visat att koden fungerar kan du vilja distribuera den så att andra kan ladda ner och installera den.
Distribution finns i många former och är starkt kopplad till programmeringsspråket och de miljöer du arbetar med.

Den enklaste distributionsformen är att ladda upp artefakter som människor kan ladda ner och installera lokalt.
Det är fortfarande vanligt och kan ses på platser som [Ubuntus paketarkiv](http://archive.ubuntu.com/ubuntu/pool/main/), som i princip är en HTTP-kataloglistning med `.deb`-filer.

I dag har GitHub blivit den faktiska standardplattformen för att publicera källkod och artefakter.
Även om källkoden ofta är offentligt tillgänglig låter GitHub Releases de ansvariga bifoga förbyggda binärer och andra artefakter till taggade versioner.

Pakethanterare stödjer ibland installation direkt från GitHub, antingen från källkod eller från en förbyggd wheel:

```console
# Installera från källkod (klonar och bygger)
$ pip install git+https://github.com/psf/requests.git

# Installera från en specifik tagg/gren
$ pip install git+https://github.com/psf/requests.git@v2.32.3

# Installera en wheel direkt från en GitHub-utgåva
$ pip install https://github.com/user/repo/releases/download/v1.0/package-1.0-py3-none-any.whl
```

Vissa språk som Go använder faktiskt en decentraliserad distributionsmodell --- i stället för ett centralt paketregister distribueras Go-moduler direkt från sina källkodsförråd.
Modulvägar som `github.com/gorilla/mux` anger var koden finns, och `go get` hämtar direkt därifrån.
De flesta pakethanterare som `pip`, `cargo` eller `brew` har dock centrala index med förpaketerade projekt för enkel distribution och installation.
Om vi kör

```console
$ uv pip install requests --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: requests==2.32.5 [compatible] (requests-2.32.5-py3-none-any.whl)
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl.metadata
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl
```

ser vi varifrån vi hämtar `requests`-wheel-filen.
Notera `py3-none-any` i filnamnet --- det betyder att wheel-filen fungerar med valfri Python 3-version, på valfritt OS, på valfri arkitektur.
För paket med kompilerad kod är wheel-filen plattformsspecifik:

```console
$ uv pip install numpy --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: numpy==2.2.1 [compatible] (numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl)
```

Här anger `cp312-cp312-macosx_14_0_arm64` att denna wheel är specifik för CPython 3.12 på macOS 14+ för ARM64 (Apple Silicon).
Om du är på en annan plattform laddar `pip` ner en annan wheel eller bygger från källkod.

Omvänt behöver vi, för att andra ska kunna hitta paketet vi skapat, publicera det till något av dessa register.
I Python är huvudregistret [Python Package Index (PyPI)](https://pypi.org).
Precis som vid installation finns flera sätt att publicera paket.
Kommandot `uv publish` ger ett modernt gränssnitt för att ladda upp paket till PyPI:

```console
$ uv publish --publish-url https://test.pypi.org/legacy/
Publishing greeting-0.1.0.tar.gz
Publishing greeting-0.1.0-py3-none-any.whl
```

Här använder vi [TestPyPI](https://test.pypi.org) --- ett separat paketregister avsett för att testa publiceringsflödet utan att förorena riktiga PyPI.
När paketet laddats upp kan du installera från TestPyPI:

```console
$ uv pip install --index-url https://test.pypi.org/simple/ greeting
```

En nyckelfråga vid publicering av programvara är tillit.
Hur verifierar användare att paketet de laddar ner faktiskt kommer från dig och inte har manipulerats?
Paketregister använder checksummor för att verifiera integritet, och vissa ekosystem stödjer paketsignering för att ge kryptografiskt bevis på upphovsperson.

Olika språk har egna paketregister: [crates.io](https://crates.io) för Rust, [npm](https://www.npmjs.com) för JavaScript, [RubyGems](https://rubygems.org) för Ruby, och [Docker Hub](https://hub.docker.com) för container images.
För privata eller interna paket sätter organisationer ofta upp egna paketförråd (som en privat PyPI-server eller ett privat Docker-register) eller använder hanterade lösningar från molnleverantörer.

Att driftsätta en webbtjänst till internet kräver ytterligare infrastruktur: domänregistrering, DNS-konfiguration som pekar domänen till servern, och ofta en omvänd proxy som nginx för att hantera HTTPS och dirigera trafik.
För enklare användningsfall som dokumentation eller statiska sajter erbjuder [GitHub Pages](https://pages.github.com/) gratis webbhotell direkt från ett kodförråd.

<!--
## Documentation

So far we have emphasized the deliverable _artifact_ as the main output of packaging and shipping code.
In addition to the artifact, we need to document for users the code's functionality, installation instructions, and usage examples.

Tools like [Sphinx](https://www.sphinx-doc.org/) (Python) and [MkDocs](https://www.mkdocs.org/) can automatically generate browsable documentation from docstrings and markdown files, often hosted on services like [Read the Docs](https://readthedocs.org/).
For HTTP-based APIs, the [OpenAPI specification](https://www.openapis.org/) (formerly Swagger) provides a standard format for describing API endpoints, which tools can use to generate interactive documentation and client libraries automatically. -->

# Övningar

1. Spara din miljö med `printenv` i en fil, skapa en venv, aktivera den, kör `printenv` till en annan fil och `diff before.txt after.txt`.
   Vad ändrades i miljön?
   Varför föredrar shell venv?
   (Tips: titta på `$PATH` före och efter aktivering.)
   Kör `which deactivate` och resonera kring vad bash-funktionen deactivate gör.
1. Skapa ett Python-paket med `pyproject.toml` och installera det i en virtuell miljö.
   Skapa en lockfile och inspektera den.
1. Installera Docker och använd det för att bygga Den saknade terminen-kursens webbplats lokalt med docker compose.
1. Skriv en Dockerfile för en enkel Python-applikation.
   Skriv sedan en `docker-compose.yml` som kör applikationen tillsammans med en Redis-cache.
1. Publicera ett Python-paket till TestPyPI (publicera inte till riktiga PyPI om det inte är värt att dela!).
   Bygg sedan en Docker-avbild med paketet och skicka den till `ghcr.io`.
1. Bygg en webbplats med [GitHub Pages](https://docs.github.com/en/pages/quickstart).
   Extra (icke-)poäng: konfigurera den med en egen domän.
