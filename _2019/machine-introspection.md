---
layout: lecture
title: "Systeminspektion"
presenter: Jon
date: 2019-01-24
order: 4
video:
  aspect: 56.25
  id: eNYT2Oq3PF8
special: true
---

Ibland beter sig datorer konstigt.
Och väldigt ofta vill du veta varför.
Låt oss titta på verktyg som hjälper dig med det.

Men först behöver vi se till att du kan göra inspektion.
Systeminspektion kräver ofta vissa behörigheter, som medlemskap i en grupp (t.ex. `power` för avstängning).
Användaren `root` har högsta behörighet och kan i princip göra vad som helst. Du kan köra ett kommando som `root` (men var försiktig) med `sudo`.

## Vad hände?

Om något går fel är första steget att titta på vad som hände när felet uppstod.
För det behöver vi läsa loggar.

Traditionellt lagrades alla loggar i `/var/log`, och många gör det fortfarande.
Ofta finns en fil eller katalog per program.
Använd `grep` eller `less` för att hitta rätt i dem.

Det finns också en kärnlogg som du kan se med kommandot `dmesg`.
Förr fanns den ofta som en vanlig textfil, men numera behöver du ofta gå via `dmesg` för att nå den.

Till sist finns "systemloggen", som i allt större utsträckning är platsen där alla loggmeddelanden hamnar.
På _de flesta_, men inte alla, Linux-system hanteras den av `systemd`, "systemdemonen", som styr alla tjänster som kör i bakgrunden (och mycket mer än så numera).
Du kommer åt loggen via det något osmidiga verktyget `journalctl` om du är root eller medlem i grupperna `admin` eller `wheel`.

För `journalctl` bör du särskilt känna till dessa flaggor:

  - `-u UNIT`: visa bara meddelanden kopplade till angiven systemd-tjänst
  - `--full`: kapa inte långa rader (den dummaste standardfunktionen)
  - `-b`: visa bara meddelanden från senaste uppstart (se även `-b -2`)
  - `-n100`: visa bara de senaste 100 posterna

## Vad händer?

Om något _är_ fel, eller om du bara vill få en känsla för vad som pågår i systemet, har du flera verktyg för att inspektera den aktuella körningen:

Först finns `top`, och den förbättrade versionen `htop`, som visar olika statistik för processer som kör i systemet.
CPU-användning, minnesanvändning, processträd, osv. Det finns många kortkommandon, men `t` är särskilt användbart för att slå på trädvy.
Du kan också se processträdet med `pstree` (+ `-p` för att visa PID:ar).
Om du vill veta vad programmen gör behöver du ofta följa deras loggfiler.
`journalctl -f`, `dmesg -w` och `tail -f` är dina vänner här.

Ibland vill du veta mer om den övergripande resursanvändningen i systemet.
[`dool`](https://github.com/scottchiefbaker/dool) är utmärkt för det.
Det ger resursmått i realtid för många olika delsystem, som I/O, nätverk, CPU-utnyttjande, context switches, med mera.
`man dool` är en bra startpunkt.

Om diskutrymmet börjar ta slut finns två huvudverktyg att kunna: `df` och `du`.
Det första visar status för alla partitioner i systemet (prova med `-h`), medan det andra mäter storleken på alla kataloger du anger, inklusive deras innehåll (se även `-h` och `-s`).

För att ta reda på vilka nätverksanslutningar du har öppna är `ss` rätt verktyg.
`ss -t` visar alla öppna TCP-anslutningar.
`ss -tl` visar alla lyssnande (alltså server-)portar i systemet.
`-p` visar också vilken process som använder anslutningen, och `-n` visar råa portnummer.


## Systemkonfiguration

Det finns _många_ sätt att konfigurera ett system, men vi går igenom två väldigt vanliga: nätverk och tjänster.
De flesta program i systemet beskriver konfiguration i sina man-sidor, och det innebär oftast att redigera filer i `/etc`, systemets konfigurationskatalog.

Om du vill konfigurera nätverket gör du det med kommandot `ip`.
Argumenten har en något udda form, men `ip help command` tar dig långt.
`ip addr` visar information om dina nätverksgränssnitt och hur de är konfigurerade (IP-adresser osv), och `ip route` visar hur nätverkstrafik routas till olika värdar.
Nätverksproblem kan ofta lösas enbart med `ip`.
Det finns också `iw` för att hantera trådlösa gränssnitt.
`ping` är ett praktiskt verktyg för att kontrollera hur trasigt något är.
Prova att pinga ett värdnamn (google.com), en extern IP-adress (1.1.1.1), och en intern IP-adress (192.168.1.1 eller standardgateway).
Du kan också behöva pilla med `/etc/resolv.conf` för att kontrollera DNS-inställningar (hur värdnamn översätts till IP-adresser).

För att konfigurera tjänster behöver du i praktiken interagera med `systemd` numera, på gott och ont.
De flesta tjänster i systemet har en systemd service-fil som definierar en systemd-_unit_.
Dessa filer anger vilket kommando som körs när tjänsten startas, hur den stoppas, var loggar skrivs, osv. De är oftast inte alltför svåra att läsa, och du hittar de flesta i `/usr/lib/systemd/system/`.
Du kan också definiera egna i `/etc/systemd/system`.

När du har en systemd-tjänst i åtanke använder du kommandot `systemctl` för att interagera med den.
`systemctl enable UNIT` gör att tjänsten startar vid uppstart (`disable` tar bort det igen), och `start`, `stop` och `restart` gör vad du förväntar dig.
Om något går fel meddelar systemd det, och du kan använda `journalctl -u UNIT` för att se programmets logg.
Du kan också använda `systemctl status` för att se hur alla systemtjänster mår.
Om uppstarten känns långsam beror det troligen på ett par långsamma tjänster, och du kan använda `systemd-analyze` (prova med `blame`) för att se vilka.

# Övningar

`locate`?
`dmidecode`?
`tcpdump`?
`/boot`?
`iptables`?
`/proc`?
