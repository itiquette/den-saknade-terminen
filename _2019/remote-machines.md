---
layout: lecture
title: "Fjärrmaskiner"
presenter: Jose
date: 2019-01-29
order: 4
video:
  aspect: 62.5
  id: X5c2Y8BCowM
---

Det har blivit allt vanligare att programmerare använder fjärrservrar i sitt dagliga arbete.
Om du behöver fjärrservrar för att driftsätta backendprogramvara, eller behöver en server med högre beräkningskapacitet, kommer du att använda Secure Shell (SSH).
Som med de flesta verktyg vi tar upp är SSH i hög grad konfigurerbart, så det är värt att lära sig ordentligt.


## Köra kommandon

En ofta förbisedd funktion i `ssh` är möjligheten att köra kommandon direkt.

- `ssh foobar@server ls` kör `ls` i hemkatalogen för användaren foobar.
- Det fungerar med rör.
`ssh foobar@server ls | grep PATTERN` söker lokalt med `grep` i fjärrutdata från `ls`, och `ls | ssh foobar@server grep PATTERN` söker med `grep` på fjärrmaskinen i lokal utdata från `ls`.

## SSH-nycklar

Nyckelbaserad autentisering använder publik nyckelkryptografi för att bevisa för servern att klienten äger den hemliga privata nyckeln, utan att avslöja själva nyckeln.
På så sätt behöver du inte skriva in lösenordet varje gång.
Den privata nyckeln (t.ex. `~/.ssh/id_rsa`) är dock i praktiken ditt lösenord, så behandla den därefter.

- Nyckelgenerering.
För att generera ett nyckelpar kan du köra `ssh-keygen -t rsa -b 4096`.
Om du inte väljer en lösenfras kan den som får tag i din privata nyckel nå auktoriserade servrar, så rekommendationen är att använda lösenfras och `ssh-agent` för att hantera skalsessioner.

Om du har konfigurerat push till GitHub med SSH-nycklar har du sannolikt redan följt stegen [här](https://help.github.com/articles/connecting-to-github-with-ssh/) och har ett giltigt nyckelpar.
För att kontrollera om du har lösenfras och verifiera nyckeln kan du köra `ssh-keygen -y -f /path/to/key`.

- Nyckelbaserad autentisering.
`ssh` tittar i `.ssh/authorized_keys` för att avgöra vilka klienter som ska släppas in.
För att kopiera över en publik nyckel kan vi använda:

```bash
cat .ssh/id_dsa.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

En enklare lösning är `ssh-copy-id` där det finns tillgängligt.

```bash
ssh-copy-id -i .ssh/id_dsa.pub foobar@remote
```

## Kopiera filer över SSH

Det finns många sätt att kopiera filer över SSH.

- `ssh+tee`: enklast är att använda kommandokörning via `ssh` och stdin med `cat localfile | ssh remote_server tee serverfile`
- `scp`: vid kopiering av stora mängder filer/kataloger är secure copy-kommandot `scp` smidigare, eftersom det lätt kan gå rekursivt över sökvägar.
Syntaxen är `scp path/to/local_file remote_host:path/to/remote_file`
- `rsync`: förbättrar `scp` genom att upptäcka identiska filer lokalt och på fjärrmaskinen,
så att de inte kopieras igen.
Det ger även mer finkornig kontroll över symlänkar och rättigheter, samt extrafunktioner som `--partial` för att återuppta en avbruten kopiering.
`rsync` har liknande syntax som `scp`.


## Bakgrundsprocesser

Som standard dödas barnprocesser till det överordnade skalet när en SSH-anslutning avbryts.
Det finns ett par alternativ.

- `nohup` - verktyget `nohup` låter i praktiken en process leva vidare när terminalen stängs.
Även om detta ibland kan lösas med `&` och `disown` är `nohup` en bättre standard.
Mer detaljer finns [här](https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and).

- `tmux`, `screen` - även om `nohup` effektivt bakgrundssätter processen är det inte bekvämt för interaktiva skalsessioner.
I sådana fall är en terminalmultiplexer som `screen` eller `tmux` ett bra val, eftersom du enkelt kan koppla från och återansluta skalen.

Om du till sist har gjort `disown` på ett program och vill återansluta det till aktuell terminal, kan du titta på [reptyr](https://github.com/nelhage/reptyr).
`reptyr PID` tar processen med id PID och kopplar den till din nuvarande terminal.

## Portvidarebefordran

I många scenarier stöter du på program som fungerar genom att lyssna på portar på maskinen.
När det sker lokalt använder du bara `localhost:PORT` eller `127.0.0.1:PORT`, men vad gör du på en fjärrserver där portarna inte är direkt tillgängliga över nätet/internet?
Det kallas portvidarebefordran, och finns i två varianter: lokal portvidarebefordran och fjärrportvidarebefordran (se bilderna för mer detaljer, bilderna kommer från [detta SO-inlägg](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)).


**Lokal portvidarebefordran** ![Lokal portvidarebefordran](https://i.stack.imgur.com/a28N8.png)

**Fjärrportvidarebefordran** ![Fjärrportvidarebefordran](https://i.stack.imgur.com/4iK3b.png)


Det vanligaste scenariot är lokal portvidarebefordran, där en tjänst på fjärrmaskinen lyssnar på en port, och du vill koppla en port på din lokala maskin till den fjärrporten.
Om vi till exempel kör `jupyter notebook` på fjärrservern och den lyssnar på port `8888`, kan vi vidarebefordra till lokal port `9999` med `ssh -L 9999:localhost:8888 foobar@remote_server` och sedan surfa till `localhost:9999` på vår lokala maskin.

## Vidarebefordran av grafik

Ibland räcker inte portvidarebefordran, eftersom vi vill köra ett GUI-baserat program på servern.
Du kan alltid använda fjärrskrivbordsprogram som skickar hela skrivbordsmiljön (t.ex. RealVNC, Teamviewer, osv).
Men för ett enstaka GUI-verktyg erbjuder SSH ett bra alternativ: vidarebefordran av grafik.

Att använda flaggan `-X` säger åt SSH att vidarebefordra.

För betrodd X11-vidarebefordran kan flaggan `-Y` användas.

Värt att uppmärksamma: för att detta ska fungera måste `sshd_config` på servern innehålla följande inställningar.

```bash
X11Forwarding yes
X11DisplayOffset 10
```

## Roaming

Ett vanligt problem vid anslutning till en fjärrserver är frånkopplingar när datorn stängs av/går i vila eller när nätverk byts.
Dessutom kan SSH bli frustrerande om anslutningen har hög latens.
[Mosh](https://mosh.org/), det mobila skalet, förbättrar SSH, och stödjer roaminganslutningar, ojämn uppkoppling och intelligent lokal eko.

Mosh finns i vanliga distributioner och pakethanterare.
Mosh kräver att en SSH-server fungerar på servern.
Du behöver inte vara superanvändare för att installera mosh, men det kräver att portar 60000 till 60010 är öppna på servern (vilket de ofta är, eftersom de inte ligger i det privilegierade intervallet).

En nackdel med `mosh` är att det inte stöder roaming för port-/grafikvidarebefordran, så om du använder det ofta hjälper `mosh` inte särskilt mycket.

## SSH-konfiguration

### Klient

Vi har gått igenom många argument som kan skickas med.
Ett frestande alternativ är att skapa skalalias som ser ut som `alias my_server="ssh -X -i ~/.id_rsa -L 9999:localhost:8888 foobar@remote_server"`, men det finns ett bättre alternativ: använd `~/.ssh/config`.

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 22
    IdentityFile ~/.ssh/id_rsa
    RemoteForward 9999 localhost:8888

# Konfigurationen kan också använda jokertecken
Host *.mit.edu
    User foobaz
```


En ytterligare fördel med `~/.ssh/config` jämfört med alias är att andra program, som `scp`, `rsync`, `mosh`, osv, också kan läsa filen och översätta inställningarna till motsvarande flaggor.


Observera att `~/.ssh/config` kan betraktas som en dotfile, och i allmänhet är det okej att ha den tillsammans med övriga dotfiles.
Men om du gör den publik, tänk på vilken information du potentiellt ger främlingar på internet: serveradresser, användarnamn, öppna portar, osv. Detta kan underlätta vissa typer av attacker, så tänk efter noga innan du delar din SSH-konfiguration.

Varning: inkludera aldrig dina RSA-nycklar (`~/.ssh/id_rsa*`) i ett publikt kodförråd.

### Serversidan

Serversidans konfiguration finns oftast i `/etc/ssh/sshd_config`.
Här kan du göra ändringar som att stänga av lösenordsautentisering, ändra SSH-port, aktivera X11-vidarebefordran, osv. Du kan även ange inställningar per användare.

## Fjärrfilsystem

Ibland är det smidigt att montera en fjärrkatalog.
[sshfs](https://github.com/libfuse/sshfs) kan montera en katalog på en fjärrserver lokalt, så att du kan använda en lokal redigerare.

## Övningar

1. För att SSH ska fungera måste värden köra en SSH-server.
Installera en SSH-server (som OpenSSH) i en virtuell maskin så att du kan göra resten av övningarna.
För att ta reda på maskinens IP-adress kör du `ip addr` och tittar efter fältet inet (ignorera posten `127.0.0.1`, den motsvarar loopback-gränssnittet).

1. Gå till `~/.ssh/` och kontrollera om du har ett SSH-nyckelpar där.
Om inte, generera ett med `ssh-keygen -t rsa -b 4096`.
Rekommendationen är att använda lösenfras och `ssh-agent`, läs mer [här](https://www.ssh.com/ssh/agent).

1. Använd `ssh-copy-id` för att kopiera nyckeln till din virtuella maskin.
Testa att du kan SSH:a utan lösenord.
Redigera sedan `sshd_config` på servern och stäng av lösenordsautentisering genom att ändra värdet på `PasswordAuthentication`.
Stäng även av root-inloggning genom att ändra `PermitRootLogin`.

1. Redigera `sshd_config` på servern för att ändra SSH-port och kontrollera att du fortfarande kan SSH:a in.
Om du har en publik server minskar icke-standardport och nyckelbaserad inloggning mängden automatiserade angrepp betydligt.

1. Installera mosh på servern/VM:n, etablera en anslutning och koppla sedan bort nätverksadaptern för servern/VM:n.
Kan mosh återhämta sig korrekt?

1. Ett annat användningsområde för lokal portvidarebefordran är att tunnla en viss värd via servern.
Om ditt nätverk filtrerar en webbplats som `reddit.com` kan du tunnla den via servern så här:

    - Kör `ssh remote_server -L 80:reddit.com:80`
    - Sätt `reddit.com` och `www.reddit.com` till `127.0.0.1` i `/etc/hosts`
    - Kontrollera att du når webbplatsen via servern
    - Om det inte är uppenbart, använd en webbplats som [ipinfo.io](https://ipinfo.io/) där resultatet ändras beroende på värdens publika IP.


1. Portvidarebefordran i bakgrunden går lätt att göra med ett par extra flaggor.
Ta reda på vad flaggorna `-N` och `-f` gör i `ssh` och förstå vad ett kommando som `ssh -N -f -L 9999:localhost:8888 foobar@remote_server` gör.


## Referenser

- [SSH Hacks](https://matt.might.net/articles/ssh-hacks/)
- [Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html)

{% comment %}
Föreläsningsanteckningar kommer att finnas tillgängliga när föreläsningen börjar.
{% endcomment %}
