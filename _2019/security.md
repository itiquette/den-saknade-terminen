---
layout: lecture
title: "Säkerhet och integritet"
presenter: Jon
date: 2019-01-31
order: 2
video:
  aspect: 56.25
  id: OBx_c-i-M8s
---

Världen kan vara en skrämmande plats, och alla verkar vara ute efter dig.

Okej, kanske inte riktigt så.
Men det betyder inte att du vill skylta med alla dina hemligheter.
Säkerhet (och integritet) handlar i allmänhet om att höja ribban för angripare.
Ta reda på vilken hotmodell du har, och utforma sedan dina skydd utifrån den.
Om hotmodellen är NSA eller Mossad kommer du _sannolikt_ få det svårt.

Det finns _många_ sätt att göra din digitala närvaro säkrare.
Vi tar upp en rad övergripande saker här, men detta är en process, och att utbilda dig själv är en av de bästa investeringarna du kan göra.
Så:

## Följ rätt personer

Ett av de bästa sätten att förbättra ditt säkerhetskunnande är att följa personer som är aktiva i säkerhetsfrågor.
Några förslag:

 - [@TroyHunt](https://twitter.com/TroyHunt)
 - [@SwiftOnSecurity](https://twitter.com/SwiftOnSecurity)
 - [@taviso](https://twitter.com/taviso)
 - [@thegrugq](https://twitter.com/thegrugq)
 - [@tqbf](https://twitter.com/tqbf)
 - [@mattblaze](https://twitter.com/mattblaze)
 - [@moxie](https://twitter.com/moxie)

Se också [den här listan](https://heimdalsecurity.com/blog/best-twitter-cybersec-accounts/) för fler förslag.

## Allmänna säkerhetsråd

Tech Solidarity har en bra lista med [do's and don'ts för journalister](https://web.archive.org/web/20221123204419/https://techsolidarity.org/resources/basic_security.htm) som innehåller sunt råd och är ganska uppdaterad.
[@thegrugq](https://medium.com/@thegrugq) har också ett bra blogginlägg om [säkerhet vid resor](https://medium.com/@thegrugq/stop-fabricating-travel-security-advice-35259bf0e869) som är värt att läsa.
Vi upprepar mycket av råden därifrån här, plus en del till.
Skaffa också en [USB-data blockerare](https://www.amazon.com/dp/B00QRRZ2QM/), eftersom [USB kan vara läskigt](https://www.bleepingcomputer.com/news/security/heres-a-list-of-29-different-types-of-usb-attacks/).

## Autentisering

Det första du bör göra, om du inte redan gjort det, är att skaffa en lösenordshanterare.
Några bra alternativ är:

 - [1password](https://1password.com/)
 - [KeePass](https://keepass.info/)
 - [BitWarden](https://bitwarden.com/)
 - [`pass`](https://git.zx2c4.com/password-store/about/)

Om du är extra försiktig, använd en som krypterar lösenorden lokalt på din dator, i stället för att lagra dem i klartext på servern.
Använd den för att generera lösenord till alla webbplatser du bryr dig om redan nu.
Slå sedan på tvåfaktorsautentisering, helst med en [FIDO/U2F](https://fidoalliance.org/)-dongel (till exempel en [YubiKey](https://www.yubico.com/quiz/), som har [20% rabatt för studenter](https://www.yubico.com/why-yubico/for-education/)).
TOTP (som Google Authenticator eller Duo) fungerar också i nödfall, men [skyddar inte mot nätfiske](https://twitter.com/taviso/status/1082015009348104192).
SMS är i stort sett värdelöst, om inte din hotmodell enbart består av slumpmässiga främlingar som snappar upp lösenord i transit.

Värt att uppmärksamma om pappersnycklar: tjänster ger dig ofta en "reservnyckel" som kan användas som andra faktor om du tappar din riktiga.
(Ha alltid en reservdongel på en säker plats.)
Du _kan_ lägga dessa nycklar i lösenordshanteraren, men då är du helt körd om någon får tillgång till den (om du inte accepterar den hotmodellen).
Om du är riktigt paranoid, skriv ut pappersnycklarna, lagra dem aldrig digitalt, och lägg dem i ett fysiskt kassaskåp.

## Privat kommunikation

Använd [Signal](https://www.signal.org/) ([installationsguide](https://medium.com/@mshelton/signal-for-beginners-c6b44f76a1f0)).
[Wire](https://wire.com/en/) är [också okej](https://www.securemessagingapps.com/).
WhatsApp är okej.
[Använd inte Telegram](https://twitter.com/bascule/status/897187286554628096).
Skrivbordsmeddelandeprogram är ganska trasiga (delvis eftersom de ofta bygger på Electron, vilket ger en stor tillitsyta).

E-post är särskilt problematiskt, även med PGP-signering.
Det är normalt inte forward-secure, och nyckeldistributionsproblemet är betydande.
[keybase.io](https://keybase.io/) hjälper, och är användbart av flera andra skäl.
PGP-nycklar hanteras dessutom oftast på skrivbordsdatorer, vilket är en av de minst säkra datormiljöerna.
I samma anda kan du överväga en Chromebook, eller att arbeta på en surfplatta med tangentbord.

## Filsäkerhet

Filsäkerhet är svårt och sker på många nivåer.
Vad är det egentligen du försöker skydda dig mot?

[![$5 wrench](https://imgs.xkcd.com/comics/security.png)](https://xkcd.com/538/)

 - Offline-attacker (någon stjäl din laptop när den är avstängd): slå på full diskkryptering.
   ([cryptsetup + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system) på Linux, [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/) på Windows, [FileVault](https://support.apple.com/en-us/HT204837) på macOS.
   Observera att detta inte hjälper om angriparen _också_ har dig och verkligen vill åt dina hemligheter.
 - Online-attacker (någon har din laptop och den är igång): använd filkryptering.
   Det finns två huvudsakliga mekanismer:
    - Krypterade filsystem: lagerbaserad filsystemskryptering krypterar filer individuellt i stället för krypterade blockenheter.
      Du kan "montera" dessa filsystem med dekrypteringsnyckeln och sedan bläddra fritt i filerna.
      När du avmonterar dem blir filerna otillgängliga.
      Moderna lösningar är [gocryptfs](https://github.com/rfjakob/gocryptfs) och [eCryptFS](https://www.ecryptfs.org/).
      Mer detaljerade jämförelser finns [här](https://nuetzlich.net/gocryptfs/comparison/) och [här](https://wiki.archlinux.org/index.php/disk_encryption#Comparison_table).
    - Krypterade filer: kryptera enskilda filer med symmetrisk kryptering (se `gpg -c`) och en hemlig nyckel.
      Eller, som `pass`, kryptera också nyckeln med din publika nyckel så att bara du kan läsa tillbaka den med din privata nyckel.
      Exakta krypteringsinställningar spelar stor roll.
 - [Plausible deniability](https://en.wikipedia.org/wiki/Plausible_deniability) ("vad verkar vara problemet, konstapeln?"): vanligtvis sämre prestanda, och lättare att tappa data.
   Svårt att faktiskt bevisa att det ger [deniable encryption](https://en.wikipedia.org/wiki/Deniable_encryption).
   Se [diskussionen här](https://security.stackexchange.com/questions/135846/is-plausible-deniability-actually-feasible-for-encrypted-volumes-disks), och överväg sedan om du vill prova [VeraCrypt](https://www.veracrypt.fr/en/Home.html) (den underhållna avgreningen av gamla goda TrueCrypt).
 - Krypterade säkerhetskopior: använd [Tarsnap](https://www.tarsnap.com/) eller [Borgbase](https://www.borgbase.com/)
    - Tänk på om en angripare kan radera dina säkerhetskopior om de får tag i din laptop.

## Internetsäkerhet och integritet

Internet är en _mycket_ skrämmande plats.
Öppna WiFi-nätverk [är](https://www.troyhunt.com/the-beginners-guide-to-breaking-website/) [skrämmande](https://www.troyhunt.com/talking-with-scott-hanselman-on/).
Se till att du tar bort dem efteråt, annars kommer telefonen glatt annonsera och återansluta till något med samma namn senare.

Om du någon gång är på ett nätverk du inte litar på kan VPN _möjligen_ vara värt det, men tänk på att du då litar _väldigt mycket_ på VPN-leverantören.
Litar du verkligen mer på dem än på din internetleverantör?
Om du verkligen vill ha VPN, använd en leverantör du är säker på att du litar på, och du bör sannolikt betala för tjänsten.
Eller sätt upp [WireGuard](https://www.wireguard.com/) själv -- det är [utmärkt](https://web.archive.org/web/20210526211307/https://latacora.micro.blog/there-will-be/).

Det finns också säkra konfigurationsinställningar för många internetanslutna program på [cipherlist.eu](https://cipherlist.eu/).
Om du är särskilt integritetsinriktad är [privacytools.io](https://privacytools.io) också en bra resurs.

Vissa undrar säkert över [Tor](https://www.torproject.org/).
Kom ihåg att Tor _inte_ är särskilt motståndskraftigt mot kraftfulla globala angripare, och är svagt mot trafikanalysattacker.
Det kan vara användbart för att dölja trafik i liten skala, men ger inte så mycket integritetsvinst totalt sett.
Du är bättre hjälpt av att använda säkrare tjänster från början (Signal, TLS + certifikatspinning, osv.).

## Webbsäkerhet

Så, du vill ut på webben också?
Du testar verkligen gränserna här.

Installera [HTTPS Everywhere](https://www.eff.org/https-everywhere).
SSL/TLS är [kritiskt](https://www.troyhunt.com/ssl-is-not-about-encryption/), och det handlar _inte_ bara om kryptering, utan också om att kunna verifiera att du faktiskt pratar med rätt tjänst. Om du kör en egen webbserver, [testa den](https://www.ssllabs.com/ssltest/index.html).
TLS-konfiguration [kan bli stökig](https://wiki.mozilla.org/Security/Server_Side_TLS).
HTTPS Everywhere gör sitt bästa för att aldrig navigera dig till HTTP-sidor när det finns ett alternativ.
Det räddar dig inte helt, men det hjälper.
Om du är riktigt paranoid, svartlista SSL/TLS-CA:er du absolut inte behöver.

Installera [uBlock Origin](https://github.com/gorhill/uBlock).
Det är en [bredspektrumblockerare](https://github.com/gorhill/uBlock/wiki/Blocking-mode) som inte bara stoppar annonser, utan även olika typer av tredjepartskommunikation som sidor försöker göra.
Och inline-skript och liknande.
Om du är villig att lägga tid på konfiguration, gå till [medium mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-medium-mode) eller till och med [hard mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-hard-mode).
De lägena _kommer_ göra att vissa webbplatser inte fungerar förrän du justerat inställningarna, men de förbättrar också din säkerhet online avsevärt.

Om du använder Firefox, aktivera [Multi-Account Containers](https://support.mozilla.org/en-US/kb/containers).
Skapa separata containrar för sociala nätverk, bank, shopping osv. Firefox håller cookies och annat tillstånd helt separerat mellan containrar, så att webbplatser i en container inte kan snoka i känslig data från andra.
I Google Chrome kan du använda [Chrome Profiles](https://support.google.com/chrome/answer/2364824) för liknande resultat.

Övningar

TODO

1. Kryptera en fil med PGP.
1. Använd VeraCrypt för att skapa en enkel krypterad volym.
1. Aktivera 2FA för dina mest datakänsliga konton, t.ex. GMail, Dropbox, GitHub, osv.
