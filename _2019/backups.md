---
layout: lecture
title: "Säkerhetskopiering"
presenter: Jose
date: 2019-01-24
order: 2
video:
  aspect: 56.25
  id: lrpqYF8tcYQ
special: true
---

Det finns två sorters människor:

- De som säkerhetskopierar
- De som kommer att säkerhetskopiera

All data du äger som du inte har säkerhetskopierat kan försvinna när som helst, för alltid.
Här går vi igenom goda grunder för säkerhetskopiering och fallgropar i vissa upplägg.

## 3-2-1-regeln

[3-2-1-regeln](https://www.us-cert.gov/sites/default/files/publications/data_backup_options.pdf) är en allmänt rekommenderad strategi för säkerhetskopiering av data.
Den säger att du bör ha:

- minst **3 kopior** av dina data
- **2** kopior på **olika medier**
- **1** av kopiorna **på annan plats**

Grundtanken bakom rekommendationen är att inte lägga alla ägg i samma korg.
Att ha två olika enheter/diskar gör att ett enskilt hårdvarufel inte tar all din data.
På samma sätt, om din enda säkerhetskopia finns hemma och huset brinner ner eller blir rånat, förlorar du allt.
Det är därför kopian utanför hemmet finns.
Lokala säkerhetskopior ger tillgänglighet och hastighet, medan kopior på annan plats ger motståndskraft om en katastrof inträffar.

## Testa dina säkerhetskopior

En vanlig fallgrop vid säkerhetskopiering är att blint lita på vad systemet säger att det gör, utan att verifiera att data faktiskt går att återställa.
Toy Story 2 var nära att gå förlorad eftersom deras säkerhetskopior inte fungerade,
och [tur](https://www.youtube.com/watch?v=8dhp_20j0Ys) räddade dem till slut.

## Versionshantering

Det är viktigt att förstå att [RAID](https://en.wikipedia.org/wiki/RAID) inte är en säkerhetskopia,
och i allmänhet är **spegling ingen säkerhetskopieringslösning**.
Att bara synkronisera filer någonstans hjälper inte i flera scenarier, till exempel:

- Datakorruption
- Skadlig programvara
- Filer som raderas av misstag

Om ändringarna i dina data förs vidare till säkerhetskopian kan du inte återställa i dessa scenarier.
Detta gäller många molnlagringstjänster som Dropbox, Google Drive, One Drive, osv.
Vissa behåller raderad data en kort tid,
men gränssnittet för återställning är oftast inte något du vill använda för stora mängder filer.

Ett ordentligt säkerhetskopieringssystem bör vara versionshanterat för att undvika detta felmönster.
Med olika ögonblicksbilder över tid kan du enkelt gå tillbaka och återställa det som gått förlorat.
Den mest kända programvaran av den typen är macOS Time Machine.

## Deduplicering

Att göra flera kopior av data kan dock bli mycket dyrt i diskutrymme.
Samtidigt är de flesta data identiska mellan versioner och behöver inte överföras igen.
Här kommer [datadeduplicering](https://en.wikipedia.org/wiki/Data_deduplication) in.
Genom att hålla reda på vad som redan lagrats kan man göra **inkrementella säkerhetskopior**, där bara ändringarna mellan två versioner behöver sparas.
Det minskar utrymmesbehovet kraftigt efter den första kopian.

## Kryptering

Eftersom vi kan säkerhetskopiera till otillförlitliga tredje parter, som molnleverantörer, bör du tänka på att data som kopieras *som den är* i princip kan läsas av obehöriga.
Dokument som deklarationsuppgifter är känslig information och bör inte säkerhetskopieras i klartext.
För att förebygga detta erbjuder många lösningar **kryptering på klientsidan**, där data krypteras innan den skickas till servern.
På så sätt kan servern inte läsa det den lagrar, medan du kan dekryptera med din hemliga nyckel.

Som en sidonotering:
om din disk (eller hempartition) inte är krypterad kan den som får fysisk tillgång till datorn ofta kringgå användarbehörigheter och läsa dina data.
Modern hårdvara hanterar krypterad läsning och skrivning snabbt och effektivt,
så du bör överväga att aktivera **full diskkryptering**.


## Endast append

Egenskaperna ovan fokuserar på hårdvarufel och användarmisstag,
men tar inte fullt höjd för vad som händer om en angripare vill radera dina data.
Om någon tar sig in i ditt system, kan de då radera alla kopior av det du bryr dig om?
Om du oroar dig för det scenariot behöver du någon form av säkerhetskopiering med endast append.
I praktiken betyder det en server som låter dig lägga till ny data,
men vägrar radera befintlig data.
Vanligen har användare två nycklar:
en append-nyckel som kan skapa nya säkerhetskopior,
och en nyckel med full åtkomst som även kan radera gamla kopior som inte längre behövs.
Den senare förvaras offline.

Observera att detta är ett ganska svårt scenario,
eftersom du behöver kunna göra ändringar samtidigt som du förhindrar att en angripare raderar dina data.
Befintliga kommersiella lösningar inkluderar [Tarsnap](https://www.tarsnap.com/) och [Borgbase](https://www.borgbase.com/).


## Ytterligare överväganden

Några andra saker du kan vilja titta på är:

- **Periodiska säkerhetskopior**: föråldrade säkerhetskopior blir snabbt ganska värdelösa.
Regelbunden säkerhetskopiering bör ingå i din systemstrategi.
- **Startbara säkerhetskopior**: vissa program kan klona hela disken.
Då får du en avbild som innehåller en full kopia av systemet som du kan starta direkt från.
- **Differentierade säkerhetskopieringsstrategier**: all data är inte lika viktig.
Du kan definiera olika säkerhetspolicys för olika typer av data.
- **Säkerhetskopior med endast append**: en extra åtgärd är att tvinga append-only-operationer i dina backup-arkiv,
så att angripare inte kan radera dem om de får kontroll över din maskin.


## Webbtjänster

All data du använder finns inte på din hårddisk.
Om du använder **webbtjänster** kan data du bryr dig om, som Google Docs-presentationer eller Spotify-spellistor, ligga online.
Ett annat lättglömt exempel är e-postkonton med webbåtkomst, som Gmail.
Att hitta en säkerhetskopieringslösning i de fallen är lite svårare.
Men många tjänster låter dig ladda ner dina data, antingen direkt eller via API.
Verktyg som [gmvault](https://github.com/gaubert/gmvault) för Gmail finns för att ladda ner e-postfiler till din dator.


## Webbsidor

På samma sätt finns mycket högkvalitativt innehåll online i form av webbsidor.
Om innehållet är statiskt kan du enkelt säkerhetskopiera det genom att spara sidan och alla dess bilagor.
Ett annat alternativ är [Wayback Machine](https://archive.org/web/),
ett enormt digitalt arkiv över webben som drivs av [Internet Archive](https://archive.org/),
en ideell organisation med fokus på att bevara olika typer av media.
Wayback Machine låter dig fånga och arkivera webbsidor och senare hämta alla ögonblicksbilder som arkiverats för den webbplatsen.
Om du tycker tjänsten är värdefull kan du överväga att [donera](https://archive.org/donate/) till projektet.


## Resurser

Några bra backup-program och tjänster som vi har använt och uppriktigt kan rekommendera:

- [Tarsnap](https://www.tarsnap.com/) - deduplicerad, krypterad säkerhetskopieringstjänst online för den verkligt paranoida.
- [Borg Backup](https://borgbackup.readthedocs.io) - deduplicerat backup-program med stöd för komprimering och autentiserad kryptering.
Om du behöver en molnleverantör är [BorgBase](https://www.borgbase.com/) ett populärt alternativ.
- [rsync](https://rsync.samba.org/) är ett verktyg för snabb inkrementell filöverföring.
Det är inte en fullständig säkerhetskopieringslösning.
- [rclone](https://rclone.org/) är som rsync men för molnlagringsleverantörer som Amazon S3, Dropbox, Google Drive, rsync.net, osv.
Stödjer kryptering på klientsidan av fjärrmappar.

## Övningar

1. Fundera på hur du (inte) säkerhetskopierar dina data i dag och förbättra upplägget.

1. Ta reda på hur du säkerhetskopierar dina e-postkonton.

1. Välj en webbtjänst du använder ofta (Spotify, Google Music, osv.) och ta reda på vilka möjligheter som finns för att säkerhetskopiera dina data.
Ofta har andra redan byggt verktyg (som [youtube-dl](https://ytdl-org.github.io/youtube-dl/)) utifrån tillgängliga API:er.

1. Tänk på en webbplats du återkommit till genom åren och slå upp den i [archive.org](https://archive.org/web/).
Hur många versioner finns det?

1. Ett sätt att effektivt implementera deduplicering är att använda hårda länkar.
En symbolisk länk (kallas också mjuk länk eller symlink) är en fil som pekar på en annan fil eller mapp,
medan en hård länk är en exakt kopia av pekaren (den använder samma inode och pekar på samma plats på disken).
Om originalfilen tas bort slutar därför en symlink att fungera, medan en hård länk fortsätter fungera.
Hårda länkar fungerar dock bara för filer.
Prova kommandot `ln` för att skapa hårda länkar och jämför med symlänkar skapade med `ln -s`.
(I macOS behöver du installera GNU coreutils eller paketet hln.)
