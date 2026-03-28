---
layout: lecture
title: "Frågor och svar"
description: >
  Svar på studentfrågor om ämnen som operativsystem, skalskriptning, verktygsrekommendationer och mer.
thumbnail: /static/assets/thumbnails/2020/lec11.png
date: 2020-01-30
ready: true
video:
  aspect: 56.25
  id: Wz50FvGG6xU
special: true
---

I den sista föreläsningen svarade vi på frågor som studenterna skickade in:

- [Några rekommendationer för att lära sig operativsystemsrelaterade ämnen som processer, virtuellt minne, avbrott, minneshantering osv?](#any-recommendations-on-learning-operating-systems-related-topics-like-processes-virtual-memory-interrupts-memory-management-etc)
- [Vilka verktyg skulle ni prioritera att lära er först?](#what-are-some-of-the-tools-youd-prioritize-learning-first)
- [När ska jag använda Python jämfört med Bash-skript jämfört med något annat språk?](#when-do-i-use-python-versus-a-bash-scripts-versus-some-other-language)
- [Vad är skillnaden mellan `source script.sh` och `./script.sh`?](#what-is-the-difference-between-source-scriptsh-and-scriptsh)
- [Var lagras olika paket och verktyg, och hur fungerar referenser till dem? Vad är egentligen `/bin` eller `/lib`?](#what-are-the-places-where-various-packages-and-tools-are-stored-and-how-does-referencing-them-work-what-even-is-bin-or-lib)
- [Ska jag köra `apt-get install` för ett python-paket, eller `pip install` för paketet?](#should-i-apt-get-install-a-python-whatever-or-pip-install-whatever-package)
- [Vilka är de enklaste och bästa profileringsverktygen för att förbättra prestanda i min kod?](#whats-the-easiest-and-best-profiling-tools-to-use-to-improve-performance-of-my-code)
- [Vilka webbläsartillägg använder ni?](#what-browser-plugins-do-you-use)
- [Vilka andra verktyg för datahantering är användbara?](#what-are-other-useful-data-wrangling-tools)
- [Vad är skillnaden mellan Docker och en virtuell maskin?](#what-is-the-difference-between-docker-and-a-virtual-machine)
- [Vilka är för- och nackdelarna med varje operativsystem, och hur väljer man mellan dem (t.ex. bästa Linux-distribution för sitt syfte)?](#what-are-the-advantages-and-disadvantages-of-each-os-and-how-can-we-choose-between-them-eg-choosing-the-best-linux-distribution-for-our-purposes)
- [Vim eller Emacs?](#vim-vs-emacs)
- [Några tips eller tricks för maskininlärningsapplikationer?](#any-tips-or-tricks-for-machine-learning-applications)
- [Fler Vim-tips?](#any-more-vim-tips)
- [Vad är 2FA och varför ska jag använda det?](#what-is-2fa-and-why-should-i-use-it)
- [Några kommentarer om skillnader mellan webbläsare?](#any-comments-on-differences-between-web-browsers)

## Några rekommendationer för att lära sig operativsystemsrelaterade ämnen som processer, virtuellt minne, avbrott, minneshantering osv? {#any-recommendations-on-learning-operating-systems-related-topics-like-processes-virtual-memory-interrupts-memory-management-etc}

Först och främst är det inte självklart att du faktiskt behöver vara väldigt insatt i alla de här områdena eftersom det är låg-nivåämnen.
De blir viktiga när du börjar skriva mer låg-nivåkod, som att implementera eller modifiera en kärna.
Annars är de flesta ämnen inte så relevanta, med undantag för processer och signaler som vi kort berörde i andra föreläsningar.

Några bra resurser för att lära sig mer:

- [MIT:s kurs 6.828](https://pdos.csail.mit.edu/6.828/) - forskarnivåkurs i operativsystemteknik.
  Kursmaterialet är publikt.
- Modern Operating Systems (4:e upplagan) av Andrew S.
  Tanenbaum är en bra översikt över många av de nämnda begreppen.
- The Design and Implementation of the FreeBSD Operating System - en bra resurs om FreeBSD (obs att det inte är Linux).
- Andra guider som [Writing an OS in Rust](https://os.phil-opp.com/) där man implementerar en kärna steg för steg i olika språk, främst i undervisningssyfte.

## Vilka verktyg skulle ni prioritera att lära er först? {#what-are-some-of-the-tools-youd-prioritize-learning-first}

Några områden som är värda att prioritera:

- Lär dig använda tangentbordet mer och musen mindre.
  Det kan handla om kortkommandon, gränssnittsanpassning osv.
- Lär dig ditt redigeringsverktyg ordentligt.
  Som programmerare går större delen av tiden åt till att redigera filer, så det lönar sig mycket att bli bra på det.
- Lär dig automatisera och/eller förenkla repetitiva delar av ditt arbetsflöde, eftersom tidsvinsten blir enorm.
- Lär dig versionshanteringsverktyg som Git och hur de används tillsammans med GitHub för samarbete i moderna programvaruprojekt.

## När ska jag använda Python jämfört med Bash-skript jämfört med något annat språk? {#when-do-i-use-python-versus-a-bash-scripts-versus-some-other-language}

Generellt är Bash-skript bra för korta och enkla engångsskript där du bara vill köra en viss sekvens av kommandon.
Bash har dock flera egenheter som gör det svårt att arbeta med i större program eller skript:

- Bash är lätt att få rätt i enkla fall men kan vara väldigt svårt att få rätt för alla möjliga indata.
  Till exempel har mellanslag i skriptargument orsakat otaliga buggar i Bash-skript.
- Bash lämpar sig dåligt för kodåteranvändning, så det kan vara svårt att återanvända delar av tidigare program.
  Mer generellt finns inget tydligt bibliotekstänk i Bash.
- Bash förlitar sig på många magiska strängar som `$?` eller `$@` för specifika värden, medan andra språk ofta använder explicita namn, som `exitCode` eller `sys.args`.

Därför rekommenderar vi mer mogna skriptspråk som Python eller Ruby för större och/eller mer komplexa skript.
Du hittar mängder av bibliotek på nätet där andra redan löst vanliga problem i de språken.
Om du hittar ett bibliotek som implementerar den funktionalitet du behöver i ett visst språk är det oftast bäst att använda just det språket.

## Vad är skillnaden mellan `source script.sh` och `./script.sh`? {#what-is-the-difference-between-source-scriptsh-and-scriptsh}

I båda fallen läses och körs `script.sh` i en Bash-session, men skillnaden är vilken session som kör kommandona.
Med `source` körs kommandona i din nuvarande Bash-session, och därför ligger förändringar i miljön kvar efteråt, som katalogbyte eller funktionsdefinitioner.
När du kör skriptet fristående med `./script.sh` startar din nuvarande Bash-session en ny Bash-instans som kör kommandona i `script.sh`.
Därför kan den nya instansen byta katalog, men när den avslutas och kontrollen går tillbaka till föräldersessionen ligger föräldersessionen kvar i samma katalog som tidigare.
På samma sätt gäller att om `script.sh` definierar en funktion som du vill använda i terminalen behöver du `source`-köra den för att funktionen ska definieras i din aktuella session.
Annars definieras funktionen i den nya Bash-processen i stället för i ditt aktuella skal.

## Var lagras olika paket och verktyg, och hur fungerar referenser till dem? Vad är egentligen `/bin` eller `/lib`? {#what-are-the-places-where-various-packages-and-tools-are-stored-and-how-does-referencing-them-work-what-even-is-bin-or-lib}

När det gäller program du kör i terminalen hittas de i katalogerna som listas i miljövariabeln `PATH`.
Du kan använda kommandot `which` (eller `type`) för att se var skalet hittar ett visst program.
I allmänhet finns konventioner för var olika filtyper ligger.
Här är några av dem vi nämnde, och se [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) för en mer komplett lista.

- `/bin` - viktiga kommandobinärer.
- `/sbin` - viktiga systembinärer, vanligtvis avsedda att köras av root.
- `/dev` - enhetsfiler, specialfiler som ofta är gränssnitt mot hårdvara.
- `/etc` - värdspecifika systemomfattande konfigurationsfiler.
- `/home` - användarnas hemkataloger.
- `/lib` - gemensamma bibliotek för systemprogram.
- `/opt` - valfri applikationsprogramvara.
- `/sys` - information och konfiguration för systemet (täcks i [första föreläsningen]({{ '/2020/course-shell/' | relative_url }})).
- `/tmp` - temporära filer (även `/var/tmp`).
  Rensas ofta mellan omstarter.
- `/usr/` - skrivskyddad användardata.
  + `/usr/bin` - icke-essentiella kommandobinärer.
  + `/usr/sbin` - icke-essentiella systembinärer, vanligtvis avsedda att köras av root.
  + `/usr/local/bin` - binärer för användarkompilerade program.
- `/var` - föränderliga filer som loggar eller cache.

## Ska jag köra `apt-get install` för ett python-paket, eller `pip install` för paketet? {#should-i-apt-get-install-a-python-whatever-or-pip-install-whatever-package}

Det finns inget allmängiltigt svar på den frågan.
Det hänger ihop med den mer generella frågan om du ska använda systemets pakethanterare eller en språkspecifik pakethanterare för att installera programvara.
Några saker att väga in:

- Vanliga paket finns ofta i båda, men mindre populära eller nyare paket kanske inte finns i systemets pakethanterare.
  I det fallet är det bättre med den språkspecifika lösningen.
- Språkspecifika pakethanterare har ofta mer uppdaterade paketversioner än systemets pakethanterare.
- När du använder systemets pakethanterare installeras bibliotek systemomfattande.
  Om du behöver olika versioner av ett bibliotek för utveckling räcker det därför ofta inte.
  För det scenariot erbjuder de flesta språk någon form av isolerad eller virtuell miljö så att du kan installera olika biblioteksversioner utan konflikter.
  För Python finns virtualenv, och för Ruby finns RVM.
- Beroende på operativsystem och hårdvaruarkitektur kan paket komma som binärer eller behöva kompileras.
  På ARM-datorer som Raspberry Pi kan systemets pakethanterare vara bättre än den språkspecifika om den första levererar binärer och den andra kräver kompilering.
  Det beror mycket på din miljö.

Du bör försöka använda antingen den ena eller den andra vägen, inte båda, eftersom blandning kan ge svårfelsökta konflikter.
Vår rekommendation är att använda språkspecifik pakethanterare när det går, och isolerade miljöer (som Pythons virtualenv) för att undvika att smutsa ner den globala miljön.

## Vilka är de enklaste och bästa profileringsverktygen för att förbättra prestanda i min kod? {#whats-the-easiest-and-best-profiling-tools-to-use-to-improve-performance-of-my-code}

Det enklaste och samtidigt ganska användbara verktyget för profilering är [tidmätning med utskrifter]({{ '/2020/debugging-profiling/#tidmätning' | relative_url }}).
Du beräknar manuellt tiden mellan olika delar av koden.
Genom att upprepa detta kan du i praktiken göra en binärsökning genom koden och hitta segmentet som tar längst tid.

För mer avancerade verktyg låter Valgrinds [Callgrind](https://valgrind.org/docs/manual/cl-manual.html) dig köra programmet och mäta hur lång tid allt tar samt hela anropsstackar, alltså vilken funktion som anropat vilken.
Det producerar sedan en annoterad version av programmets källkod med tidsåtgång per rad.
Det saktar dock ner programmet ungefär en storleksordning och stödjer inte trådar.
För andra fall kan [`perf`](https://www.brendangregg.com/perf.html) och andra språkspecifika sampling-profilerare snabbt ge användbar data.
[Flamdiagram](https://www.brendangregg.com/flamegraphs.html) är en bra visualisering av utdata från sådana sampling-profilerare.
Du bör också försöka använda verktyg som är specifika för språket eller uppgiften du jobbar med.
För webbutveckling har till exempel utvecklarverktygen i Chrome och Firefox utmärkta profilerare.

Ibland är den långsamma delen i koden att systemet väntar på en händelse, som en diskläsning eller ett nätverkspaket.
I de fallen är det värt att kontrollera att överslagsräkningar av teoretisk hastighet utifrån hårdvarans kapacitet stämmer med faktisk mätdata.
Det finns också specialverktyg för att analysera väntetider i systemanrop.
Dit hör verktyg som [eBPF](https://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html) som utför kärnspårning av användarprogram.
Särskilt [`bpftrace`](https://github.com/iovisor/bpftrace) är värt att titta på om du behöver den typen av låg-nivåprofilering.

## Vilka webbläsartillägg använder ni? {#what-browser-plugins-do-you-use}

Några av våra favoriter, främst för säkerhet och användbarhet:

- [uBlock Origin](https://github.com/gorhill/uBlock) - en [bred blockerare](https://github.com/gorhill/uBlock/wiki/Blocking-mode) som inte bara stoppar annonser utan även många former av tredjepartskommunikation som en sida försöker göra.
  Det inkluderar även inline-skript och andra typer av resursladdning.
  Om du är villig att lägga tid på konfiguration för att få saker att fungera, testa [medium mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-medium-mode) eller till och med [hard mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-hard-mode).
  Det kommer att göra att vissa sajter inte fungerar förrän du justerat inställningarna tillräckligt, men det förbättrar också din säkerhet på nätet markant.
  Annars är [easy mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-easy-mode) redan ett bra standardläge som blockerar de flesta annonser och spårare.
  Du kan också definiera egna regler för vilka webbobjekt som ska blockeras.
- [Stylus](https://github.com/openstyles/stylus/) - en förgrening av Stylish (använd inte Stylish, det visade sig [stjäla användares webbhistorik](https://www.theregister.co.uk/2018/07/05/browsers_pull_stylish_but_invasive_browser_extension/)) som låter dig sidladda egna CSS-stilmallar till webbplatser.
  Med Stylus kan du enkelt anpassa och ändra utseendet på webbplatser.
  Det kan vara att ta bort en sidopanel, ändra bakgrundsfärg eller textstorlek och typsnitt.
  Det är fantastiskt för att göra webbplatser du ofta besöker mer lättlästa.
  Stylus kan dessutom hitta stilar som andra användare skrivit och publicerat på [userstyles.org](https://userstyles.org/).
  De flesta vanliga webbplatser har till exempel en eller flera mörka teman där.
- Full Page Screen Capture - [inbyggt i Firefox](https://screenshots.firefox.com/) och finns även som [Chrome-tillägg](https://chrome.google.com/webstore/detail/full-page-screen-capture/fdpohaocaechififmbbbbbknoalclacl?hl=en).
  Låter dig ta skärmdump av en hel webbplats, ofta betydligt bättre än utskrift för referensändamål.
- [Multi Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/) - låter dig separera cookies i "containrar" så att du kan surfa med olika identiteter och/eller säkerställa att webbplatser inte kan dela information mellan varandra.
- Integrering med lösenordshanterare - de flesta lösenordshanterare har webbläsartillägg som gör inmatning av inloggningsuppgifter både smidigare och säkrare.
  Jämfört med att bara kopiera och klistra in användarnamn och lösenord kontrollerar dessa verktyg först att domänen matchar den som finns i posten, vilket skyddar mot nätfiskeattacker.
- [Vimium](https://github.com/philc/vimium) - ett webbläsartillägg som ger tangentbordsbaserad navigering och styrning av webben i Vim-anda.

## Vilka andra verktyg för datahantering är användbara? {#what-are-other-useful-data-wrangling-tools}

Några verktyg för datahantering vi inte hann ta upp i föreläsningen är `jq` och `pup`, som är specialiserade tolkningsverktyg för respektive JSON- och HTML-data.
Programsproket Perl är också ett bra verktyg för mer avancerade datahanteringspipelines.
Ett annat knep är kommandot `column -t` som kan användas för att omvandla blankstegstext (inte nödvändigtvis justerad) till korrekt kolumnjusterad text.

Mer generellt är vim och Python två något mer okonventionella men kraftfulla datahanteringsverktyg.
För vissa komplexa transformationer över flera rader kan vim-makron vara ovärderliga.
Du kan spela in en serie åtgärder och upprepa dem så många gånger du vill.
I [anteckningarna om redigerare]({{ '/2020/editors/#macros' | relative_url }}) (och förra årets [video]({{ '/2019/editors/' | relative_url }})) finns till exempel ett exempel där en XML-fil omvandlas till JSON med enbart vim-makron.

För tabellformad data, ofta i CSV-format, är Python-biblioteket [pandas](https://pandas.pydata.org/) ett utmärkt verktyg.
Inte bara för att det gör det enkelt att definiera komplexa operationer som group by, joins eller filter, utan också för att det gör det enkelt att rita diagram över olika egenskaper i datan.
Det stödjer även export till många tabellformat, inklusive XLS, HTML och LaTeX. Alternativt har programspråket R (ett möjligen [dåligt](https://arrgh.tim-smith.us/) språk) mycket funktionalitet för statistik över data och kan vara användbart som sista steg i din pipeline.
[ggplot2](https://ggplot2.tidyverse.org/) är ett utmärkt diagrambibliotek i R.

## Vad är skillnaden mellan Docker och en virtuell maskin? {#what-is-the-difference-between-docker-and-a-virtual-machine}

Docker bygger på ett mer generellt koncept som kallas containrar.
Den viktigaste skillnaden mellan containrar och virtuella maskiner är att virtuella maskiner kör en hel OS-stack, inklusive kärnan, även om kärnan är samma som på värdmaskinen.
Till skillnad från VM:ar undviker containrar att köra en extra kärninstans och delar i stället kärna med värden.
I Linux sker detta via en mekanism som kallas LXC, och den använder en serie isoleringsmekanismer för att starta ett program som tror att det kör på egen hårdvara, fast det i verkligheten delar hårdvara och kärna med värden.
Containrar har därför lägre belastning än en full VM.
Å andra sidan har containrar svagare isolering och fungerar bara om värden kör samma kärna.
Om du till exempel kör Docker på macOS behöver Docker starta en Linux-VM för att få en Linux-kärna, och därför blir belastningen fortfarande betydande.
Till sist är Docker en specifik containerimplementation anpassad för programvarudistribution.
Därför har den vissa egenheter, till exempel att Docker-containrar som standard inte bevarar någon lagring mellan omstarter.

## Vilka är för- och nackdelarna med varje operativsystem, och hur väljer man mellan dem (t.ex. bästa Linux-distribution för sitt syfte)? {#what-are-the-advantages-and-disadvantages-of-each-os-and-how-can-we-choose-between-them-eg-choosing-the-best-linux-distribution-for-our-purposes}

När det gäller Linuxdistributioner gäller att även om det finns väldigt många så beter sig de flesta ganska likt för de flesta användningsfall.
Det mesta av Linux- och UNIX-funktioner och intern mekanik kan läras i vilken distribution som helst. En grundläggande skillnad mellan distributioner är hur de hanterar paketuppdateringar.
Vissa, som Arch Linux, använder en rullande uppdateringsmodell där du får det senaste men där saker ibland går sönder.
Andra, som Debian, CentOS eller Ubuntu LTS, är betydligt mer konservativa med uppdateringar i sina kodförråd, vilket oftast ger mer stabilitet men färre nya funktioner.
Vår rekommendation för en enkel och stabil upplevelse på både desktop och server är Debian eller Ubuntu.

macOS är en bra mellanpunkt mellan Windows och Linux med ett välpolerat gränssnitt. macOS bygger dock på BSD i stället för Linux, så vissa delar av systemet och vissa kommandon skiljer sig.
Ett alternativ värt att titta på är FreeBSD.
Även om vissa program inte körs på FreeBSD är BSD-ekosystemet mindre fragmenterat och bättre dokumenterat än Linux.
Vi avråder från Windows för allt utom utveckling av Windows-applikationer eller om du behöver en avgörande funktion, som bra drivrutinsstöd för spel.

För dual-boot-system tycker vi att den mest fungerande implementationen är macOS Boot Camp, och att andra kombinationer kan bli problematiska över tid, särskilt i kombination med till exempel diskkryptering.

## Vim eller Emacs? {#vim-vs-emacs}

Vi tre använder vim som primär redigerare, men Emacs är också ett bra alternativ, och det är värt att prova båda för att se vad som passar dig bäst. Emacs följer inte Vims modala redigering som standard, men det kan aktiveras via Emacs-tillägg som [Evil](https://github.com/emacs-evil/evil) eller [Doom Emacs](https://github.com/hlissner/doom-emacs).
En fördel med Emacs är att tillägg kan implementeras i Lisp, ett bättre skriptspråk än vimscript, som är Vims standardskriptspråk.

## Några tips eller tricks för maskininlärningsapplikationer? {#any-tips-or-tricks-for-machine-learning-applications}

Flera av lärdomarna från kursen går att tillämpa direkt på ML-applikationer.
Som i många vetenskapliga discipliner gör man inom ML ofta en serie experiment och vill se vad som fungerade och inte.
Du kan använda skalverktyg för att snabbt söka igenom experiment och aggregera resultaten på ett vettigt sätt.
Det kan handla om att välja ut alla experiment inom ett visst tidsintervall eller de som använder en viss datamängd.
Om du loggar relevanta experimentparametrar i en enkel JSON-fil kan detta bli väldigt enkelt med verktygen vi gått igenom i kursen.
Om du dessutom inte jobbar i ett kluster där du skickar in GPU-jobb bör du se över hur den processen kan automatiseras, eftersom den annars kan bli både tidskrävande och mentalt dränerande.

## Fler Vim-tips? {#any-more-vim-tips}

Några ytterligare tips:

- Tillägg - ta dig tid att utforska tilläggsekosystemet.
  Det finns många bra tillägg som åtgärdar begränsningar i vim eller lägger till ny funktionalitet som passar bra i befintliga vim-arbetsflöden.
  Bra resurser är [VimAwesome](https://vimawesome.com/) och andra programmerares dotfiles.
- Markeringar - i vim kan du sätta en markering med `m<X>` för en bokstav `X`.
  Du hoppar tillbaka till markeringen med `'<X>`.
  Det gör det enkelt att snabbt navigera till specifika positioner inom en fil eller mellan filer.
- Navigering - `Ctrl+O` och `Ctrl+I` flyttar dig bakåt respektive framåt mellan nyligen besökta positioner.
- Undo tree - Vim har en avancerad mekanism för att hålla reda på ändringar.
  Till skillnad från andra redigerare lagrar vim ett träd av ändringar, så även om du ångrar och sedan gör en annan ändring kan du fortfarande gå tillbaka till ursprungsläget genom att navigera i trädet.
  Tillägg som [gundo.vim](https://github.com/sjl/gundo.vim) och [undotree](https://github.com/mbbill/undotree) visar trädet grafiskt.
- Tidsbaserad ångra - kommandona `:earlier` och `:later` låter dig navigera filer via tidsreferenser i stället för en ändring i taget.
- [Persistent undo](https://vim.fandom.com/wiki/Using_undo_branches#Persistent_undo) är en fantastisk inbyggd vim-funktion som är avstängd som standard.
  Den bevarar ångrahistorik mellan vim-sessioner.
  Genom att sätta `undofile` och `undodir` i `.vimrc` sparar vim filspecifik ändringshistorik.
- Leader key - leader-tangenten är en specialtangent som ofta lämnas till användaren för egna kommandon.
  Mönstret är vanligtvis att trycka och släppa den tangenten (ofta mellanslag) och sedan en annan tangent för ett visst kommando.
  Ofta använder tillägg den här tangenten för egen funktionalitet, till exempel använder UndoTree `<Leader> U` för att öppna undo tree.
- Avancerade textobjekt - textobjekt som sökningar kan också kombineras med vim-kommandon.
  Till exempel tar `d/<pattern>` bort text fram till nästa träff av mönstret, och `cgn` ändrar nästa förekomst av den senaste söksträngen.

## Vad är 2FA och varför ska jag använda det? {#what-is-2fa-and-why-should-i-use-it}

Tvåfaktorsautentisering (2FA) lägger till ett extra skyddslager för dina konton ovanpå lösenord.
För att logga in behöver du inte bara kunna ett lösenord, utan också på något sätt "bevisa" att du har tillgång till en fysisk enhet.
I enklaste fallet kan det vara ett SMS till mobilen, även om det finns [kända problem](https://www.kaspersky.com/blog/2fa-practical-guide/24219/) med SMS-baserad 2FA.
Ett bättre alternativ som vi rekommenderar är en [U2F](https://en.wikipedia.org/wiki/Universal_2nd_Factor)-lösning som [YubiKey](https://www.yubico.com/).

## Några kommentarer om skillnader mellan webbläsare? {#any-comments-on-differences-between-web-browsers}

Webbläsarlandskapet runt 2020 var att de flesta i praktiken liknade Chrome eftersom de använder samma motor (Blink).
Det betyder att Microsoft Edge, som också bygger på Blink, och Safari, som bygger på WebKit (en liknande motor), i praktiken var sämre varianter av Chrome.
Chrome är en rimligt bra webbläsare både vad gäller prestanda och användbarhet.
Om du vill ha ett alternativ är Firefox vår rekommendation.
Den är jämförbar med Chrome på nästan alla punkter och utmärker sig av integritetsskäl.
En annan webbläsare, [Flow](https://www.ekioh.com/flow-browser/), var inte redo för användare ännu men implementerade en ny återgivningsmotor som lovade högre prestanda än de dåvarande.
