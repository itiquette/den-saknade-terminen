---
layout: lecture
title: "Säkerhet och kryptografi"
description: >
  Lär dig om kryptografiska primitiver som hashfunktioner och nyckelhärledningsfunktioner, och förstå hur verktyg som Git och SSH använder dem.
thumbnail: /static/assets/thumbnails/2020/lec9.png
date: 2020-01-28
ready: true
video:
  aspect: 56.25
  id: tjwobAmnKTo
special: true
---

Förra årets [föreläsning om säkerhet och integritet]({{ '/2019/security/' | relative_url }}) fokuserade på hur du kan vara säkrare som dator_användare_.
I år fokuserar vi på säkerhets- och kryptografibegrepp som är relevanta för att förstå verktygen vi gått igenom tidigare i kursen, som användningen av hashfunktioner i Git eller nyckelhärledningsfunktioner och symmetriska/asymmetriska kryptosystem i SSH.

Föreläsningen ersätter inte en mer rigorös och komplett kurs i systemsäkerhet ([6.858](https://css.csail.mit.edu/6.858/)) eller kryptografi ([6.857](https://courses.csail.mit.edu/6.857/) och 6.875).
Arbeta inte med säkerhet utan formell säkerhetsutbildning.
Om du inte är expert, [implementera inte din egen kryptografi](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html).
Samma princip gäller systemsäkerhet.

Föreläsningen behandlar grundläggande kryptografibegrepp på ett högst informellt (men förhoppningsvis praktiskt) sätt.
Den räcker inte för att lära dig _designa_ säkra system eller kryptografiska protokoll, men vi hoppas att den räcker för att ge en generell förståelse för program och protokoll du redan använder.

# Entropi

[Entropi](https://en.wikipedia.org/wiki/Entropy_(information_theory)) är ett mått på slumpmässighet.
Det är användbart till exempel när man bedömer styrkan i ett lösenord.

![XKCD 936: Password Strength](https://imgs.xkcd.com/comics/password_strength.png)

Som [XKCD-serien](https://xkcd.com/936/) ovan illustrerar är ett lösenord som "correcthorsebatterystaple" säkrare än ett som "Tr0ub4dor&3".
Men hur kvantifierar man det?

Entropi mäts i _bitar_, och när man väljer helt uniformt från en mängd möjliga utfall är entropin lika med `log_2(antal möjligheter)`.
En rättvis slantsingling ger 1 bit entropi.
Ett tärningskast (med en sexsidig tärning) har cirka 2,58 bitar entropi.

Du bör utgå från att angriparen känner till lösenordets _modell_, men inte slumpen som användes för att välja det specifika lösenordet (t.ex. från [tärningskast](https://en.wikipedia.org/wiki/Diceware)).

Hur många entropibitar räcker?
Det beror på din hotmodell.
För onlinegissning är cirka 40 bitar, som XKCD-serien påpekar, ganska bra.
För att stå emot offlinegissning behövs ett starkare lösenord (t.ex. 80 bitar eller mer).

# Hashfunktioner

En [kryptografisk hashfunktion](https://en.wikipedia.org/wiki/Cryptographic_hash_function) mappar data av godtycklig storlek till en fast storlek och har vissa specialegenskaper.
En grov specifikation av en hashfunktion är:

```
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```

Ett exempel på hashfunktion är [SHA1](https://en.wikipedia.org/wiki/SHA-1), som används i Git.
Den mappar indata av godtycklig storlek till utdata på 160 bitar (som kan representeras som 40 hexadecimala tecken).
Vi kan testa SHA1 på indata med kommandot `sha1sum`:

```console
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'Hello' | sha1sum
f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0
```

På hög nivå kan en hashfunktion ses som en svårinverterad, slumpmässigt utseende (men deterministisk) funktion (och detta är [idealmodellen för en hashfunktion](https://en.wikipedia.org/wiki/Random_oracle)).
En hashfunktion har följande egenskaper:

- Deterministisk: samma indata ger alltid samma utdata.
- Icke-inverterbar: det är svårt att hitta indata `m` så att `hash(m) = h` för önskad utdata `h`.
- Målkollisionsresistent: givet indata `m_1` är det svårt att hitta en annan indata `m_2` så att `hash(m_1) = hash(m_2)`.
- Kollisionsresistent: det är svårt att hitta två indata `m_1` och `m_2` så att `hash(m_1) = hash(m_2)` (notera att detta är en strikt starkare egenskap än målkollisionsresistens).

Obs: även om den kan fungera för vissa syften anses SHA-1 [inte längre](https://web.archive.org/web/20260207211148/https://shattered.io/) vara en stark kryptografisk hashfunktion.
Du kan tycka att tabellen över [livslängder för kryptografiska hashfunktioner](https://valerieaurora.org/hash.html) är intressant.
Notera dock att rekommendationer av specifika hashfunktioner ligger utanför föreläsningens omfång.
Om du arbetar med sådant behöver du formell utbildning i säkerhet/kryptografi.

## Tillämpningar

- Git, för innehållsadresserad lagring.
  Idén om en [hashfunktion](https://en.wikipedia.org/wiki/Hash_function) är mer generell (det finns icke-kryptografiska hashfunktioner).
  Varför använder Git en kryptografisk hashfunktion?
- En kort sammanfattning av innehållet i en fil.
  Mjukvara kan ofta laddas ner från (potentiellt mindre betrodda) speglar, till exempel Linux-ISO:er, och det är trevligt att inte behöva lita helt på dem.
  Officiella sajter publicerar vanligtvis hashar bredvid nedladdningslänkarna (som pekar till tredjepartsspeglar), så att hashen kan kontrolleras efter nedladdning.
- [Commitment schemes](https://en.wikipedia.org/wiki/Commitment_scheme).
  Anta att du vill binda dig till ett visst värde men avslöja själva värdet senare.
  Exempelvis vill jag göra en rättvis slantsingling "i huvudet", utan ett betrott delat mynt som två parter ser.
  Jag kan välja `r = random()`, och sedan dela `h = sha256(r)`.
  Då kan du säga krona eller klave (vi kommer överens om att jämn `r` betyder krona och udda `r` betyder klave).
  Efter att du gissat kan jag avslöja `r`, och du kan verifiera att jag inte fuskat genom att kontrollera att `sha256(r)` matchar hashen jag delade tidigare.

# Nyckelhärledningsfunktioner

Ett närliggande begrepp till kryptografiska hashar är [nyckelhärledningsfunktioner](https://en.wikipedia.org/wiki/Key_derivation_function) (KDF:er), som används för flera tillämpningar, bland annat för att producera utdata med fast längd som kan användas som nycklar i andra kryptografiska algoritmer.
Vanligtvis är KDF:er avsiktligt långsamma för att bromsa offlineattacker med brute force.

## Tillämpningar

- Producera nycklar från lösenfraser för användning i andra kryptografiska algoritmer (t.ex. symmetrisk kryptografi, se nedan).
- Lagra inloggningsuppgifter.
  Att lagra lösenord i klartext är dåligt.
  Rätt angreppssätt är att generera och lagra ett slumpmässigt [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) `salt = random()` för varje användare, lagra `KDF(password + salt)` och verifiera inloggningsförsök genom att beräkna KDF igen med det angivna lösenordet och det lagrade saltet.

# Symmetrisk kryptografi

Att dölja meddelandeinnehåll är antagligen det första man tänker på när man tänker på kryptografi.
Symmetrisk kryptografi åstadkommer detta med följande funktionalitet:

```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```

Funktionen `encrypt` har egenskapen att givet utdata (chiffertext) är det svårt att avgöra indata (klartext) utan nyckeln.
Funktionen `decrypt` har den självklara korrekthetsegenskapen att `decrypt(encrypt(m, k), k) = m`.

Ett exempel på ett symmetriskt kryptosystem som används brett i dag är [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard).

## Tillämpningar

- Kryptera filer för lagring i en icke betrodd molntjänst. Detta kan kombineras med KDF:er så att du kan kryptera en fil med en lösenfras.
  Generera `key = KDF(passphrase)` och lagra sedan `encrypt(file, key)`.

# Asymmetrisk kryptografi

Begreppet "asymmetrisk" syftar på att det finns två nycklar med två olika roller.
En privat nyckel är, som namnet antyder, avsedd att hållas privat, medan den publika nyckeln kan delas offentligt utan att säkerheten påverkas (till skillnad från nyckeln i ett symmetriskt kryptosystem).
Asymmetriska kryptosystem erbjuder följande funktionalitet, för kryptering/dekryptering och signering/verifiering:

```
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```

`encrypt`/`decrypt` har egenskaper liknande motsvarigheterna i symmetriska system.
Ett meddelande kan krypteras med den _publika_ nyckeln.
Givet utdata (chiffertext) är det svårt att bestämma indata (klartext) utan den _privata_ nyckeln.
`decrypt` har den självklara korrekthetsegenskapen `decrypt(encrypt(m, public key), private key) = m`.

Symmetrisk och asymmetrisk kryptering kan jämföras med fysiska lås.
Ett symmetriskt kryptosystem är som ett dörrlås: alla med nyckeln kan låsa och låsa upp.
Asymmetrisk kryptering är som ett hänglås med nyckel.
Du kan ge någon det olåsta låset (publika nyckeln), de kan lägga ett meddelande i en låda och sätta på låset, och sedan kan bara du öppna det eftersom du behöll nyckeln (privata nyckeln).

`sign`/`verify` har de egenskaper man önskar av fysiska signaturer, i den meningen att det är svårt att förfalska en signatur.
Oavsett meddelande är det utan den _privata_ nyckeln svårt att skapa en signatur så att `verify(message, signature, public key)` returnerar true.
Och förstås har `verify` den självklara korrekthetsegenskapen att `verify(message, sign(message, private key), public key) = true`.

## Tillämpningar

- [PGP-kryptering av e-post](https://en.wikipedia.org/wiki/Pretty_Good_Privacy).
  Människor kan publicera sina publika nycklar på nätet (t.ex. i en PGP-nyckelserver eller på [Keybase](https://keybase.io/)).
  Vem som helst kan då skicka krypterad e-post till dem.
- Privat meddelandeutbyte.
  Appar som [Signal](https://signal.org/) och [Keybase](https://keybase.io/) använder asymmetriska nycklar för att etablera privata kommunikationskanaler.
- Signering av programvara.
  Git kan ha GPG-signerade incheckningar och taggar.
  Med en publicerad publik nyckel kan vem som helst verifiera äktheten hos nedladdad programvara.

## Nyckeldistribution

Asymmetrisk kryptografi är fantastisk, men den har en stor utmaning i att distribuera publika nycklar och mappa dem till verkliga identiteter.
Det finns många lösningar på detta.
Signal har en enkel modell: trust on first use, plus stöd för nyckelutbyte utanför bandet (du verifierar vänners "safety numbers" personligen).
PGP har en annan modell, [web of trust](https://en.wikipedia.org/wiki/Web_of_trust).
Keybase har ytterligare en modell med [sociala bevis](https://keybase.io/blog/chat-apps-softer-than-tofu) (tillsammans med andra smarta idéer).
Varje modell har sina styrkor.
Vi (föreläsarna) gillar Keybases modell.

# Fallstudier

## Lösenordshanterare

Det är ett grundläggande verktyg som alla bör försöka använda (t.ex. [KeePassXC](https://keepassxc.org/), [pass](https://git.zx2c4.com/password-store/about/) och [1Password](https://1password.com)).
Lösenordshanterare gör det enkelt att använda unika, slumpgenererade högentropilösenord för alla inloggningar, och de sparar alla lösenord på ett ställe, krypterade med ett symmetriskt chiffer med en nyckel framställd ur en lösenfras via en KDF.

Att använda lösenordshanterare gör att du kan undvika återanvändning av lösenord (så du påverkas mindre när webbplatser komprometteras), använda högentropilösenord (så risken minskar att du komprometteras) och bara behöva komma ihåg ett enda högentropilösenord.

## Tvåfaktorsautentisering

[Tvåfaktorsautentisering](https://en.wikipedia.org/wiki/Multi-factor_authentication) (2FA) kräver att du använder en lösenfras ("något du vet") tillsammans med en 2FA-autentiserare (som en [YubiKey](https://www.yubico.com/), "något du har") för skydd mot stulna lösenord och [nätfiskeattacker](https://en.wikipedia.org/wiki/Phishing).

## Heldiskkryptering

Att ha hela din bärbara dators disk krypterad är ett enkelt sätt att skydda data om datorn blir stulen.
Du kan använda [cryptsetup + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system) på Linux, [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/) på Windows eller [FileVault](https://support.apple.com/en-us/HT204837) på macOS.
Det krypterar hela disken med ett symmetriskt chiffer, med en nyckel skyddad av en lösenfras.

## Privat meddelandeutbyte

Använd [Signal](https://signal.org/) eller [Keybase](https://keybase.io/).
End-to-end-säkerhet bootstrappas från asymmetrisk kryptering.
Att få tag på kontakternas publika nycklar är det kritiska steget här.
Om du vill ha god säkerhet behöver du autentisera publika nycklar utanför bandet (med Signal eller Keybase), eller lita på sociala bevis (med Keybase).

## SSH

Vi har gått igenom SSH och SSH-nycklar i en [tidigare föreläsning]({{ '/2020/command-line/#remote-machines' | relative_url }}).
Låt oss titta på kryptografidelen av detta.

När du kör `ssh-keygen` genereras ett asymmetriskt nyckelpar, `public_key, private_key`.
Det genereras slumpmässigt med entropi från operativsystemet (insamlad från hårdvaruhändelser m.m.).
Den publika nyckeln lagras som den är (den är publik, så hemlighållande är inte viktigt), men den privata nyckeln bör i vila vara krypterad på disk.
Programmet `ssh-keygen` ber användaren om en lösenfras, och den matas genom en nyckelhärledningsfunktion för att ta fram en nyckel som sedan används för att kryptera den privata nyckeln med ett symmetriskt chiffer.

I användning, när servern känner till klientens publika nyckel (lagrad i `.ssh/authorized_keys`), kan en anslutande klient bevisa sin identitet med asymmetriska signaturer.
Det sker med [challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication).
På hög nivå väljer servern ett slumpmässigt tal och skickar det till klienten.
Klienten signerar sedan meddelandet och skickar signaturen tillbaka till servern, som kontrollerar signaturen mot den registrerade publika nyckeln.
Det bevisar i praktiken att klienten har den privata nyckeln som motsvarar den publika nyckeln i serverns `.ssh/authorized_keys`, och servern kan därför tillåta inloggning.

{% comment %}
extra topics, if there's time

security concepts, tips
- biometrics
- HTTPS
{% endcomment %}

# Resurser

- [Förra årets anteckningar]({{ '/2019/security/' | relative_url }}): från när den här föreläsningen fokuserade mer på säkerhet och integritet för datoranvändare.
- [Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html) (rätta svar om kryptografi): svarar på "vilken kryptografi ska jag använda för X?" för många vanliga X.

# Övningar

1. **Entropi.**
    1. Anta att ett lösenord väljs som en konkatenering av fyra ord i gemener, där varje ord väljs uniformt slumpmässigt från en ordlista med 100 000 ord.
       Ett exempel på ett sådant lösenord är `correcthorsebatterystaple`.
       Hur många entropibitar har detta?
    1. Tänk på ett alternativt schema där ett lösenord väljs som en sekvens av 8 slumpmässiga alfanumeriska tecken (inklusive både gemener och versaler).
       Ett exempel är `rg8Ql34g`.
       Hur många entropibitar har detta?
    1. Vilket är det starkare lösenordet?
    1. Anta att en angripare kan prova 10 000 lösenord per sekund.
       Hur lång tid tar det i genomsnitt att knäcka respektive lösenord?
1. **Kryptografiska hashfunktioner.** Ladda ner en Debian-avbildning från en [spegel](https://www.debian.org/CD/http-ftp/) (t.ex. [från denna argentinska spegel](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/)).
   Verifiera hashen (t.ex. med kommandot `sha256sum`) mot hashen hämtad från den officiella Debian-sajten (t.ex. [denna fil](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS) som ligger på `debian.org`, om du har laddat ner den länkade filen från den argentinska spegeln).
1. **Symmetrisk kryptografi.** Kryptera en fil med AES med [OpenSSL](https://www.openssl.org/): `openssl aes-256-cbc -salt -in {input filename} -out {output filename}`.
   Titta på innehållet med `cat` eller `hexdump`.
   Dekryptera med `openssl aes-256-cbc -d -in {input filename} -out {output filename}` och bekräfta med `cmp` att innehållet matchar originalet.
1. **Asymmetrisk kryptografi.**
    1. Sätt upp [SSH-nycklar](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) på en dator du har tillgång till (inte Athena, eftersom Kerberos interagerar konstigt med SSH-nycklar).
       Se till att din privata nyckel är krypterad med en lösenfras så att den är skyddad i vila.
    1. [Sätt upp GPG](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages).
    1. Skicka krypterad e-post till Anish ([publik nyckel](https://keybase.io/anish)).
    1. Signera en Git-incheckning med `git commit -S` eller skapa en signerad Git-tagg med `git tag -s`.
       Verifiera signaturen på incheckningen med `git show --show-signature` eller på taggen med `git tag -v`.
