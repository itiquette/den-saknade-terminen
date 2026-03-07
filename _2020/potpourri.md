---
layout: lecture
title: "Blandat"
description: >
  Lär dig en rad användbara ämnen, bland annat tangentbordsomappning, demoner (daemons), säkerhetskopiering, API:er och mer.
thumbnail: /static/assets/thumbnails/2020/lec10.png
date: 2020-01-29
ready: true
video:
  aspect: 56.25
  id: JZDt-PRq0uo
special: true
---

## Innehåll

- [Tangentbordsomappning](#tangentbordsomappning)
- [Demoner (daemons)](#daemoner)
- [FUSE](#fuse)
- [Säkerhetskopior](#säkerhetskopior)
- [API:er](#apier)
- [Vanliga kommandoradsflaggor/mönster](#vanliga-kommandoradsflaggormönster)
- [Fönsterhanterare](#fönsterhanterare)
- [VPN](#vpn)
- [Markdown](#markdown)
- [Hammerspoon (skrivbordsautomation på macOS)](#hammerspoon-skrivbordsautomation-på-macos)
- [Uppstart + live-USB](#uppstart--live-usb)
- [Docker, Vagrant, VM:ar, molnet, OpenStack](#docker-vagrant-vmar-molnet-openstack)
- [Notebook-programmering](#notebook-programmering)
- [GitHub](#github)

## Tangentbordsomappning

Som programmerare är tangentbordet din huvudsakliga inmatningsmetod.
Precis som nästan allt annat i datorn går det att konfigurera (och det är värt att konfigurera).

Den mest grundläggande ändringen är att mappa om tangenter.
Det innebär vanligtvis någon programvara som lyssnar och, när en viss tangent trycks, fångar händelsen och ersätter den med en annan händelse som motsvarar en annan tangent.
Några exempel:
- Mappa om Caps Lock till Ctrl eller Escape.
  Vi (föreläsarna) rekommenderar starkt detta eftersom Caps Lock ligger väldigt bra till men sällan används.
- Mappa om PrtSc till Play/Pause för musik.
  De flesta operativsystem har en play/pause-tangent.
- Byta plats på Ctrl och Meta-tangenten (Windows/Command).

Du kan också mappa tangenter till valfria kommandon.
Det är användbart för vanliga uppgifter du utför ofta.
Här lyssnar programvara på en viss tangentkombination och kör ett skript när händelsen upptäcks.
- Öppna ett nytt terminal- eller webbläsarfönster.
- Sätta in specifik text, t.ex. din långa e-postadress eller ditt MIT-id.
- Försätta datorn eller skärmarna i vila.

Du kan även konfigurera mer avancerade ändringar:
- Mappa om tangentsekvenser, t.ex. att trycka Shift fem gånger växlar Caps Lock.
- Mappning vid tryck vs håll, t.ex. att Caps Lock mappas till Esc vid snabbt tryck men till Ctrl vid hållning som modifierare.
- Göra ommappningar tangentbords- eller programspecifika.

Några resurser för att komma igång:
- macOS - [karabiner-elements](https://karabiner-elements.pqrs.org/), [skhd](https://github.com/koekeishiya/skhd) eller [BetterTouchTool](https://folivora.ai/)
- Linux - [xmodmap](https://wiki.archlinux.org/index.php/Xmodmap) eller [Autokey](https://github.com/autokey/autokey)
- Windows - inbyggt i Kontrollpanelen, [AutoHotkey](https://www.autohotkey.com/) eller [SharpKeys](https://www.randyrants.com/category/sharpkeys/)
- QMK - om tangentbordet stödjer anpassad inbyggd programvara (firmware) kan du använda [QMK](https://docs.qmk.fm/) för att konfigurera hårdvaran direkt så att ommappningen fungerar på alla datorer där tangentbordet används.

## Demoner (daemons) {#daemoner}

Du är antagligen redan bekant med begreppet demoner (daemons), även om ordet kan kännas nytt.
De flesta datorer har en serie processer som alltid körs i bakgrunden i stället för att vänta på att en användare startar och interagerar med dem.
Dessa processer kallas demoner, och program som körs som demoner slutar ofta på `d` för att markera det.
Till exempel `sshd`, SSH-daemonen, som lyssnar på inkommande SSH-förfrågningar och kontrollerar att fjärranvändaren har nödvändiga uppgifter för att logga in.

I Linux är `systemd` (systemdaemonen) den vanligaste lösningen för att köra och konfigurera daemonprocesser.
Du kan köra `systemctl status` för att lista demoner som körs just nu.
Många kan låta obekanta, men de ansvarar för kärndelar av systemet som nätverkshantering, DNS-uppslagning och grafiskt gränssnitt.
Du interagerar med systemd via `systemctl` för att `enable`, `disable`, `start`, `stop`, `restart` eller kontrollera `status` för tjänster.

Ännu mer intressant är att `systemd` har ett ganska lättillgängligt gränssnitt för att konfigurera och aktivera nya demoner (tjänster).
Nedan är ett exempel på en daemon som kör en enkel Python-app.
Vi går inte in på detaljer, men de flesta fälten är ganska självförklarande.

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom App
After=network.target

[Service]
User=foo
Group=foo
WorkingDirectory=/home/foo/projects/mydaemon
ExecStart=/usr/bin/local/python3.7 app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Om du bara vill köra ett program med viss frekvens behöver du inte bygga en egen daemon.
Du kan använda [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html), en daemon som systemet redan kör för schemalagda uppgifter.

## FUSE

Moderna programvarusystem består ofta av mindre byggblock som kopplas ihop.
Operativsystemet stödjer olika filsystemsbackends eftersom det finns ett gemensamt språk för vilka operationer ett filsystem stödjer.
När du till exempel kör `touch` för att skapa en fil gör `touch` ett systemanrop till kärnan för att skapa filen, och kärnan gör i sin tur rätt filsystemsanrop.
En begränsning är att UNIX-filsystem traditionellt implementeras som kärnmoduler och att bara kärnan får utföra filsystemsanrop.

[FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) (Filesystem in User Space) låter filsystem implementeras av användarprogram.
FUSE låter användare köra userspace-kod för filsystemsoperationer och bryggar sedan nödvändiga anrop till kärnans gränssnitt.
I praktiken innebär det att användare kan implementera godtycklig funktionalitet för filsystemsoperationer.

Till exempel kan FUSE användas så att varje operation i ett virtuellt filsystem skickas vidare via SSH till en fjärrmaskin, utförs där och att resultatet skickas tillbaka.
På så vis kan lokala program se filen som om den låg på din dator, fast den i verkligheten ligger på en fjärrserver.
Detta är i praktiken vad `sshfs` gör.

Några intressanta exempel på FUSE-filsystem:
- [sshfs](https://github.com/libfuse/sshfs) - öppna fjärrfiler/-mappar lokalt via en SSH-anslutning.
- [rclone](https://rclone.org/commands/rclone_mount/) - montera molnlagringstjänster som Dropbox, GDrive, Amazon S3 eller Google Cloud Storage och öppna data lokalt.
- [gocryptfs](https://nuetzlich.net/gocryptfs/) - krypterat overlaysystem.
  Filer lagras krypterat men när filsystemet monteras visas de som klartext i monteringspunkten.
- [kbfs](https://keybase.io/docs/kbfs) - distribuerat filsystem med end-to-end-kryptering.
  Du kan ha privata, delade och publika mappar.
- [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html) - montera deduplicerade, komprimerade och krypterade säkerhetskopior för enklare bläddring.

## Säkerhetskopior

All data som du inte har säkerhetskopierat är data som kan försvinna när som helst, för alltid.
Det är lätt att kopiera data, men svårt att säkerhetskopiera data pålitligt.
Här är några bra grunder och fallgropar.

En kopia av data på samma disk är inte en säkerhetskopia, eftersom disken är en enskild felpunkt för all datan.
På samma sätt är en extern disk hemma en svag backup-lösning eftersom den kan försvinna i brand, inbrott osv.
I stället rekommenderas säkerhetskopior på annan plats.

Synkroniseringslösningar är inte säkerhetskopior.
Dropbox/GDrive är till exempel bekväma lösningar, men när data raderas eller korruptas sprider de ändringen.
Av samma skäl är disk-spegling som RAID inte säkerhetskopior.
Det hjälper inte om data raderas, korruptas eller krypteras av ransomware.

Några kärnegenskaper hos bra backup-lösningar är versionshantering, deduplicering och säkerhet.
Versionshanterade säkerhetskopior säkerställer att du kan komma åt historiken och återställa filer effektivt.
Effektiva lösningar använder deduplicering för att bara lagra inkrementella ändringar och minska lagringskostnaden.
När det gäller säkerhet bör du fråga dig vad någon behöver veta/ha för att kunna läsa din data och, ännu viktigare, radera all din data och alla tillhörande säkerhetskopior.
Till sist är det en dålig idé att blint lita på säkerhetskopior.
Du bör regelbundet verifiera att de faktiskt går att använda för återställning.

Säkerhetskopiering handlar inte bara om lokala filer på din dator.
Med den kraftiga tillväxten av webbapplikationer lagras stora mängder av din data endast i molnet.
Till exempel försvinner webmail, sociala medie-foton, spellistor i strömningstjänster eller dokument på nätet om du tappar tillgång till respektive konto.
Att ha en offline-kopia av sådan information är rätt väg, och det finns verktyg på nätet som hämtar och sparar datan.

För en mer detaljerad genomgång, se 2019 års anteckningar om [säkerhetskopior]({{ '/2019/backups' | relative_url }}).

## API:er

Vi har pratat mycket i kursen om att använda datorn mer effektivt för _lokala_ uppgifter, men du kommer märka att många av de här lärdomarna även gäller det bredare internet.
De flesta tjänster på nätet har "API:er" som låter dig komma åt deras data programmatiskt.
Till exempel har USA:s regering ett API för väderprognoser, vilket du kan använda för att enkelt hämta väderprognos i ditt skal.

De flesta API:er har liknande format.
De är strukturerade URL:er, ofta under `api.service.com`, där path och query-parametrar anger vilken data du vill läsa eller vilken åtgärd du vill utföra.
För väderdatan i USA gör du till exempel en GET-förfrågan (med `curl`, till exempel) till https://api.weather.gov/points/42.3604,-71.094 för att få prognosen för en viss plats.
Svaret innehåller i sin tur flera andra URL:er för mer specifika prognoser i regionen.
Vanligtvis är svaren i JSON-format, som du sedan kan skicka vidare till ett verktyg som [`jq`](https://stedolan.github.io/jq/) för att forma datan till det du behöver.

Vissa API:er kräver autentisering, och det sker oftast i form av en hemlig _token_ som du måste inkludera i förfrågan.
Läs API-dokumentationen för den specifika tjänsten du använder, men "[OAuth](https://www.oauth.com/)" är ett protokoll du ofta kommer se.
I grunden är OAuth ett sätt att ge dig tokenar som kan "agera som du" i en viss tjänst och som bara får användas för specifika syften.
Kom ihåg att dessa tokenar är _hemliga_, och att den som får tillgång till din token kan göra allt tokenen tillåter under _ditt_ konto.

[IFTTT](https://ifttt.com/) är en webbplats och tjänst centrerad kring API-idén.
Den erbjuder integrationer med massor av tjänster och låter dig kedja händelser mellan dem på nästan godtyckliga sätt.
Ta en titt.

## Vanliga kommandoradsflaggor/mönster

Kommandoradsverktyg varierar mycket, och du vill ofta läsa deras `man`-sidor innan du använder dem.
Men de delar ofta vissa gemensamma funktioner som är bra att känna till:

 - De flesta verktyg stödjer någon form av `--help`-flagga för korta användningsinstruktioner.
 - Många verktyg som kan orsaka irreversibla ändringar stödjer "dry run" (testkörning), där de bara skriver ut vad de _skulle_ ha gjort utan att faktiskt göra ändringen.
   På liknande sätt har de ofta en interaktiv flagga som frågar dig om varje destruktiv åtgärd.
 - Du kan vanligtvis använda `--version` eller `-V` för att få programmet att skriva ut sin version (praktiskt vid felrapportering).
 - Nästan alla verktyg har en `--verbose`- eller `-v`-flagga för mer utförlig utdata.
   Du kan ofta ange den flera gånger (`-vvv`) för _ännu_ mer utdata, vilket kan vara användbart vid felsökning.
   På samma sätt har många verktyg en `--quiet`-flagga för att bara skriva ut vid fel.
 - I många verktyg betyder `-` i stället för filnamn "standardindata" eller "standardutdata", beroende på argumentet.
 - Potentiellt destruktiva verktyg är i regel inte rekursiva som standard, men stödjer en rekursiv flagga (ofta `-r`) för att arbeta rekursivt.
 - Ibland vill du skicka något som _ser ut_ som en flagga som vanligt argument.
   Tänk att du vill ta bort en fil som heter `-r`.
   Eller du vill köra ett program "genom" ett annat, som `ssh machine foo`, och du vill skicka en flagga till det "inre" programmet (`foo`).
   Specialargumentet `--` gör att programmet _slutar_ tolka följande som flaggor/optioner (saker som börjar med `-`), så du kan skicka argument som ser ut som flaggor utan att de tolkas som sådana: `rm -- -r` eller `ssh machine --for-ssh -- foo --for-foo`.

## Fönsterhanterare

De flesta av er är vana vid en fönsterhanterare med "dra och släpp", som standardmiljön i Windows, macOS och Ubuntu.
Fönster ligger "flytande" på skärmen, du kan dra dem, ändra storlek och låta dem överlappa.
Men detta är bara en _typ_ av fönsterhanterare, ofta kallad "flytande".
Det finns många andra, särskilt i Linux.
Ett särskilt vanligt alternativ är "tiling"-fönsterhanterare.
I en tiling-hanterare överlappar fönster aldrig, utan arrangeras som plattor på skärmen, ungefär som paneler i tmux.
Skärmen fylls alltid av de öppna fönstren, ordnade enligt någon _layout_.
Om du bara har ett fönster tar det hela skärmen.
Om du öppnar ett till krymper det första för att göra plats (ofta något som 2/3 och 1/3).
Om du öppnar ett tredje krymper de andra igen för att ge plats.
Precis som med tmux-paneler kan du navigera mellan dessa fönster med tangentbordet, ändra storlek och flytta runt dem utan mus.
Det är definitivt värt att testa.

## VPN

VPN är väldigt hajpat i dag, men det är inte självklart att det finns [någon bra anledning](https://web.archive.org/web/20230710155258/https://gist.github.com/joepie91/5a9909939e6ce7d09e29).
Du bör veta vad en VPN gör och inte gör för dig.
I bästa fall är en VPN _egentligen_ bara ett sätt att byta internetleverantör ur internets perspektiv.
All din trafik ser ut att komma från VPN-leverantören i stället för din "riktiga" plats, och nätverket du är ansluten till ser bara krypterad trafik.

Det kan låta attraktivt, men tänk på att när du använder VPN flyttar du i praktiken bara förtroendet från din nuvarande internetleverantör till VPN-företaget.
Allt din internetleverantör _skulle_ kunna se är det VPN-leverantören nu ser _i stället_.
Om du litar _mer_ på dem än på din leverantör är det en vinst, men annars är det oklart hur mycket du egentligen vunnit.
Om du sitter på osäker, okrypterad publik Wi-Fi på en flygplats kanske du inte litar på anslutningen, men hemma är avvägningen mindre tydlig.

Du bör också veta att mycket av din trafik i dag, åtminstone känslig trafik, _redan_ är krypterad via HTTPS eller TLS mer generellt.
I så fall spelar det ofta liten roll om nätverket är "dåligt" eller inte.
Nätverksoperatören lär sig vilka servrar du pratar med, men inte datan som utbyts.

Notera att jag sa "i bästa fall" ovan.
Det är inte ovanligt att VPN-leverantörer råkar felkonfigurera sin programvara så att krypteringen blir svag eller helt avstängd.
Vissa VPN-leverantörer är illvilliga (eller åtminstone opportunistiska) och loggar all din trafik, och säljer eventuellt den informationen vidare.
Att välja en dålig VPN-leverantör är ofta sämre än att inte använda VPN alls.

I nödfall driver MIT en [VPN-tjänst](https://ist.mit.edu/vpn) för studenter, vilket kan vara värt att titta på.
Och om du ska köra eget, ta en titt på [WireGuard](https://www.wireguard.com/).

## Markdown

Chansen är stor att du kommer skriva en hel del text under din karriär.
Och ofta vill du formatera den texten på enkla sätt.
Du vill kanske ha fet eller kursiv text, eller lägga in rubriker, länkar och kodfragment.
I stället för tunga verktyg som Word eller LaTeX kan det vara värt att använda det lätta märkspråket [Markdown](https://commonmark.org/help/).

Du har troligen redan sett Markdown, eller åtminstone någon variant.
Delmängder används nästan överallt, även om det inte alltid kallas Markdown.
I grunden är Markdown ett försök att kodifiera hur människor redan brukar markera text när de skriver rena textdokument.
Betoning (*kursiv*) görs genom att omge ord med `*`.
Stark betoning (**fetstil**) görs med `**`.
Rader som börjar med `#` är rubriker (och antalet `#` avgör rubriknivå).
Rader som börjar med `-` är punktlistepunkter, och rader som börjar med ett tal + `.` är numrerade listpunkter.
Backticks används för ord i `kodstil`, och kodblock kan skrivas genom att indentera med fyra blanksteg eller omge med trippelbackticks:

    ```
    code goes here
    ```

För att lägga in en länk, placera länk_texten_ i hakparenteser och URL:en direkt efter i parentes: `[name](url)`.
Markdown är enkelt att komma igång med, och du kan använda det nästan överallt.
Faktum är att anteckningarna för den här och alla andra föreläsningar är skrivna i Markdown, och du kan se rå-Markdown [här](https://raw.githubusercontent.com/missing-semester/missing-semester/master/_2020/potpourri.md).

## Hammerspoon (skrivbordsautomation på macOS)

[Hammerspoon](https://www.hammerspoon.org/) är ett ramverk för skrivbordsautomation på macOS.
Det låter dig skriva Lua-skript som kopplar in i operativsystemets funktionalitet, så att du kan interagera med tangentbord/mus, fönster, skärmar, filsystem och mycket mer.

Några exempel på vad du kan göra med Hammerspoon:

- Binda snabbtangenter för att flytta fönster till specifika positioner.
- Skapa en menyradsknapp som automatiskt lägger ut fönster i en viss layout.
- Stänga av högtalaren när du kommer till labbet (genom att känna av Wi-Fi-nätverket).
- Visa en varning om du råkat ta med en väns strömadapter.

På hög nivå låter Hammerspoon dig köra godtycklig Lua-kod kopplad till menyknappar, tangenttryckningar eller händelser, och Hammerspoon ger ett omfattande bibliotek för systeminteraktion.
Det innebär att det i praktiken knappt finns någon gräns för vad du kan göra.
Många har gjort sina Hammerspoon-konfigurationer publika, så du kan ofta hitta det du behöver genom att söka på nätet, men du kan alltid skriva egen kod från grunden.

### Resurser

- [Getting Started with Hammerspoon](https://www.hammerspoon.org/go/)
- [Sample configurations](https://github.com/Hammerspoon/hammerspoon/wiki/Sample-Configurations)
- [Anish's Hammerspoon config](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)

## Uppstart + live-USB

När din maskin startar, innan operativsystemet laddas, initierar [BIOS](https://en.wikipedia.org/wiki/BIOS)/[UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) systemet.
Under den processen kan du trycka en viss tangentkombination för att konfigurera den inbyggda programvaran (firmware).
Din dator kan till exempel visa något i stil med "Tryck F9 för att konfigurera BIOS. Tryck F12 för att öppna startmenyn." under uppstart.
Du kan konfigurera många hårdvarurelaterade inställningar i BIOS-menyn.
Du kan också gå till startmenyn för att starta från en alternativ enhet i stället för hårddisken.

[Live-USB](https://en.wikipedia.org/wiki/Live_USB) är USB-minnen som innehåller ett operativsystem.
Du kan skapa ett genom att ladda ner ett operativsystem (t.ex. en Linux-distribution) och skriva det till USB-minnet.
Den processen är lite mer komplicerad än att bara kopiera en `.iso`-fil till disken.
Verktyg som [UNetbootin](https://unetbootin.github.io/) hjälper dig skapa en live-USB.

Live-USB-minnen är användbara för många syften.
Bland annat kan du, om din befintliga OS-installation går sönder så att den inte längre startar, använda en live-USB för att återställa data eller reparera operativsystemet.

## Docker, Vagrant, VM:ar, molnet, OpenStack

[Virtuella maskiner](https://en.wikipedia.org/wiki/Virtual_machine) och liknande verktyg som containrar låter dig emulera ett helt datorsystem, inklusive operativsystem.
Detta kan vara användbart för att skapa isolerade miljöer för testning, utveckling eller utforskning (t.ex. köra potentiellt skadlig kod).

[Vagrant](https://www.vagrantup.com/) är ett verktyg som låter dig beskriva maskinkonfigurationer (operativsystem, tjänster, paket osv.) i kod och sedan instansiera VM:ar med ett enkelt `vagrant up`.
[Docker](https://www.docker.com/) är konceptuellt likt men använder containrar i stället.

Du kan också hyra virtuella maskiner i molnet, och det är ett trevligt sätt att få direkt tillgång till:

- En billig maskin som alltid är på och har publik IP-adress, för att drifta tjänster.
- En maskin med mycket CPU, disk, RAM och/eller GPU.
- Fler maskiner än du fysiskt har tillgång till (debitering är ofta per sekund, så om du vill ha mycket beräkning under kort tid är det rimligt att hyra 1000 datorer i några minuter).

Populära tjänster är [Amazon AWS](https://aws.amazon.com/), [Google Cloud](https://cloud.google.com/), [Microsoft Azure](https://azure.microsoft.com/) och [DigitalOcean](https://www.digitalocean.com/).

Om du är medlem i MIT CSAIL kan du få gratis VM:ar för forskningsändamål via [CSAIL OpenStack-instansen](https://tig.csail.mit.edu/shared-computing/open-stack/).

## Notebook-programmering

[Notebook-miljöer](https://en.wikipedia.org/wiki/Notebook_interface) kan vara väldigt praktiska för vissa typer av interaktiv eller utforskande utveckling.
Den mest populära notebook-miljön i dag är kanske [Jupyter](https://jupyter.org/), för Python (och flera andra språk).
[Wolfram Mathematica](https://www.wolfram.com/mathematica/) är en annan notebook-miljö som är utmärkt för matematikorienterad programmering.

## GitHub

[GitHub](https://github.com/) är en av de mest populära plattformarna för utveckling av öppen källkod-projekt.
Många av verktygen vi pratat om i kursen, från [vim](https://github.com/vim/vim) till [Hammerspoon](https://github.com/Hammerspoon/hammerspoon), finns på GitHub.
Det är lätt att börja bidra till öppen källkod och förbättra verktygen du använder varje dag.

Det finns två huvudsakliga sätt att bidra till projekt på GitHub:

- Skapa ett [ärende](https://help.github.com/en/github/managing-your-work-on-github/creating-an-issue).
  Det kan användas för att rapportera programfel eller önska nya funktioner.
  Inget av detta kräver att du läser eller skriver kod, så tröskeln kan vara låg.
  Högkvalitativa felrapporter kan vara mycket värdefulla för utvecklare.
  Det hjälper också att kommentera i befintliga diskussioner.
- Bidra med kod via en [ändringsförfrågan (PR)](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
  Det är oftast mer omfattande än att skapa ett ärende.
  Du kan [skapa en avgrening](https://help.github.com/en/github/getting-started-with-github/fork-a-repo) av ett kodförråd på GitHub, klona din avgrening, skapa en ny gren, göra ändringar (t.ex. åtgärda ett programfel eller implementera en funktion), pusha grenen och sedan [skapa en ändringsförfrågan (PR)](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request).
  Efter det följer vanligtvis en dialog med projektets förvaltare, som ger återkoppling på din ändring.
  Till sist, om allt går väl, slås ändringen samman i det ursprungliga kodförrådet.
  Ofta har större projekt en guide för hur man bidrar, märker nybörjarvänliga ärenden och ibland till och med mentorprogram för att hjälpa nya bidragsgivare in i projektet.
