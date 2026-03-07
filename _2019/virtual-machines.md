---
layout: lecture
title: "Virtuella maskiner och containrar"
presenter: Anish, Jon
date: 2019-01-15
order: 2
video:
  aspect: 56.25
  id: LJ9ki5zq6Ik
---

# Virtuella maskiner

Virtuella maskiner är simulerade datorer.
Du kan konfigurera en gästmaskin med ett operativsystem och valfri konfiguration,
och använda den utan att påverka värdmiljön.

I den här kursen kan du använda VM:ar för att experimentera med operativsystem,
programvara och konfigurationer utan risk.
Du påverkar inte din primära utvecklingsmiljö.

Generellt har VM:ar många användningsområden.
De används ofta för att köra program som bara fungerar på ett visst operativsystem
(t.ex. en Windows-VM på Linux för Windows-specifik programvara).
De används också ofta för att experimentera med potentiellt skadlig programvara.

## Användbara egenskaper

- **Isolering**: hypervisorer gör oftast ett bra jobb med att isolera gästen från
värden,
så du kan köra buggig eller otillförlitlig programvara i VM relativt säkert.

- **Ögonblicksbilder**: du kan ta "snapshots" av din virtuella maskin,
som fångar hela maskintillståndet (disk, minne osv),
göra ändringar,
och sedan återställa till ett tidigare läge.
Det är användbart för att testa potentiellt destruktiva åtgärder, bland annat.

## Nackdelar

Virtuella maskiner är generellt långsammare än att köra direkt på hårdvaran,
så de kan vara olämpliga för vissa tillämpningar.

## Konfiguration

- **Resurser**: delas med värdmaskinen.
Tänk på detta när du allokerar fysiska resurser.

- **Nätverk**: många alternativ.
Standard-NAT fungerar bra i de flesta fall.

- **Gästtillägg**: många hypervisorer kan installera programvara i gästen för
bättre integration med värdsystemet.
Använd detta om du kan.

## Resurser

- Hypervisorer
    - [VirtualBox](https://www.virtualbox.org/) (öppen källkod)
    - [Virt-manager](https://virt-manager.org/) (öppen källkod, hanterar KVM-virtuella maskiner och LXC-containrar)
    - [VMWare](https://www.vmware.com/) (kommersiellt, tillgängligt från IS&T [för
    MIT-studenter](https://ist.mit.edu/vmware-fusion))

Om du redan är bekant med populära hypervisorer/VM:ar kan du vilja lära dig ett mer kommandoradsvänligt arbetssätt.
Ett alternativ är verktygssviten [libvirt](https://wiki.libvirt.org/page/UbuntuKVMWalkthrough),
som låter dig hantera flera olika virtualiseringsleverantörer/hypervisorer.

## Övningar

1. Ladda ner och installera en hypervisor.

1. Skapa en ny virtuell maskin och installera en Linux-distribution (t.ex.
[Debian](https://www.debian.org/)).

1. Experimentera med snapshots.
Prova saker du alltid velat testa,
som att köra `sudo rm -rf --no-preserve-root /`,
och se om du enkelt kan återställa.

1. Läs om vad en [fork-bomb](https://en.wikipedia.org/wiki/Fork_bomb) (`:(){ :|:& };:`) är och kör den i VM:n för att se att resursisoleringen (CPU, minne, osv.) fungerar.

1. Installera gästtillägg och experimentera med olika fönsterlägen, fildelning och andra funktioner.

# Containrar

Virtuella maskiner är relativt tungviktiga.
Men vad händer om du vill starta upp miljöer automatiserat?
Då kommer containrar in i bilden.

 - Amazon Firecracker
 - Docker
 - rkt
 - lxc

Containrar är _mest_ en sammansättning av olika Linux-säkerhetsfunktioner,
som virtuella filsystem,
virtuella nätverksgränssnitt,
chroots,
virtuellt minne,
med mera,
som tillsammans ger ett virtualiseringsliknande beteende.

Inte riktigt lika säkert eller isolerat som en VM,
men ganska nära och blir bättre.
Vanligtvis högre prestanda och mycket snabbare uppstart,
men inte alltid.

Prestandavinsten kommer av att containrar, till skillnad från VM:ar som kör en hel kopia av operativsystemet,
delar Linux-kärna med värden.
Observera dock att om du kör Linux-containrar på Windows/macOS behöver en Linux-VM vara aktiv som mellanlager.

![Docker vs VM]({{ '/2019/files/containers-vs-vms.png' | relative_url }})
_Jämförelse mellan Docker-containrar och virtuella maskiner.
Källa: blog.docker.com_

Containrar är praktiska när du vill köra en automatiserad uppgift i en
standardiserad miljö:

  - Byggsystem
  - Utvecklingsmiljöer
  - Förpaketerade servrar
  - Köra otillförlitliga program
    - Rätta studentinlämningar
    - (Viss) molnberäkning
  - Kontinuerlig integration
   - Travis CI
   - GitHub Actions

Dessutom har containerprogramvara som Docker använts mycket som lösning på [beroendehelvete](https://en.wikipedia.org/wiki/Dependency_hell).
Om en maskin måste köra många tjänster med konfliktande beroenden kan de isoleras med containrar.

Vanligtvis skriver du en fil som definierar hur containern byggs.
Du börjar med en minimal _basavbild_ (som Alpine Linux),
och lägger sedan till en lista med kommandon för att sätta upp önskad miljö
(installera paket, kopiera filer, bygga saker, skriva konfigurationsfiler osv).
Normalt finns också ett sätt att ange externa portar som ska vara tillgängliga,
samt en _entrypoint_ som bestämmer vilket kommando som körs när containern startar
(t.ex. ett rättningsskript).

På samma sätt som kodförrådssajter (som [GitHub](https://github.com/)) finns containerförråd (som [DockerHub](https://hub.docker.com/))
där många programtjänster har färdigbyggda avbilder som är enkla att driftsätta.

## Övningar

1. Välj en containerprogramvara (Docker, LXC, …) och installera en enkel Linux-avbild.
Försök SSH:a in i den.

1. Sök upp och ladda ner en färdigbyggd containeravbild för en populär webbserver (nginx, apache, …).
