---
layout: lecture
title: "Varför vi håller den här kursen"
---

I en traditionell datavetenskapsutbildning läser du kurser som lär ut avancerade ämnen; allt från operativsystem till programmeringsspråk och maskininlärning.
Men på många lärosäten finns det ett viktigt område som sällan täcks och ofta lämnas åt studenterna själva att lära sig: att känna sig bekväma med det digitala ekosystemet.

Under åren har vi hjälpt till att undervisa i flera kurser på MIT, och gång på gång har vi sett att många studenter har en begränsad kunskap om tillgängliga verktyg.
Studenterna utför ofta repetitiva moment, trots att datorer byggdes för att automatisera manuella uppgifter, eller så missar de att nyttja det bästa ur kraftfulla verktyg som versionshantering och textredigerare.
I bästa fall leder det till ineffektivitet och slöseri med tid; i värsta fall leder det till problem som dataförlust eller att en del uppgifter inte går att slutföra.

De här ämnena ingår sällan i universitetets ordinarie kursplan: oftast får studenterna inte lära sig hur verktygen används, eller åtminstone inte hur de används effektivt, och slösar därför tid och energi på uppgifter som _borde_ vara enkla.
Den vanliga datavetenskapsutbildningen saknar kritiska delar om det digitala ekosystemet som skulle göra studenternas vardag betydligt enklare.

# Den saknade terminen i din datavetenskapsutbildning

För att ändra på det skapade vi en kurs som täcker alla ämnen vi anser vara avgörande för att bli en effektiv datavetare och programmerare.
Kursen är pragmatisk och praktisk, och ger en handfast introduktion till verktyg och tekniker som du direkt kan använda i många olika situationer.
Den senaste versionen av kursen, med ett kraftigt omarbetat material, ges under MIT:s "Independent Activities Period" i januari 2026 — en enmånadersperiod med kortare studentdrivna kurser.
Även om föreläsningarna i sig bara är tillgängliga för MIT-gemenskapen kommer vi att publicera allt kursmaterial tillsammans med videoinspelningar av dem.

Låter det här som något för dig nämner vi här några konkreta exempel på vad kursen lär ut:

## Kommandoskal

Hur du automatiserar vanliga och repetitiva uppgifter med alias, skript och byggsystem.
Du slipper att kopiera och klistra in kommandon från ett textdokument.
Du slipper att hamna i lägen där du måste "köra de här 15 kommandona ett efter ett".
Du slipper också missar av typen "du glömde köra det här" eller "du glömde skicka med det här argumentet".

Det kan spara mycket tid att kunna söka i historiken.
Vi visar i exemplet nedan flera knep för att navigera i skalhistoriken för `convert`-kommandon.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/history.mp4' | relative_url }}" type="video/mp4">
</video>

## Versionshantering

Hur du använder versionshantering _på rätt sätt_ för att undvika katastrofer, samarbeta med andra och snabbt hitta samt isolera problematiska ändringar.
Du slipper `rm -rf; git clone`.
Du slipper att fastna i sammanslagningskonflikter (eller åtminstone får du färre av dem).
Du slipper ha stora block med utkommenterad kod.
Du slipper att fundera på vad som fick koden att gå sönder.
Du slipper att tänka "hoppsan, raderade vi den fungerande koden?!".
Vi kommer även att lära dig hur du bidrar till andras projekt med ändringsförfrågningar (pull requests).

I exemplet nedan använder vi `git bisect` för att hitta vilken incheckning som fick ett enhetstest att misslyckas, och rättar det sedan med `git revert`.
<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/git.mp4' | relative_url }}" type="video/mp4">
</video>

## Textredigering

Hur du redigerar filer effektivt från kommandoraden, både lokalt och på fjärrmaskiner, och kan dra nytta av avancerade redigerarfunktioner.
Du slipper att kopiera filer fram och tillbaka.
Du slipper repetitiva filredigeringar.

Vim-makron är en av Vims bästa egenskaper, och i exemplet nedan konverterar vi snabbt en HTML-tabell till CSV-format med hjälp av ett nästlat Vim-makro.
<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/vim.mp4' | relative_url }}" type="video/mp4">
</video>

## Fjärrmaskiner

Hur du behåller lugnet när du arbetar mot fjärrmaskiner med SSH-nycklar och terminalmultiplexering.
Du slipper att hålla flera terminaler öppna bara för att köra två kommandon samtidigt.
Du slipper att skriva in lösenordet varje gång du ansluter.
Du slipper att förlora allt bara för att internetanslutningen bröts eller att du behövde starta om din dator.

I exemplet nedan använder vi `tmux` för att hålla sessioner vid liv på fjärrservrar och `mosh` för att hantera nätverksbyten och avbrott.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/ssh.mp4' | relative_url }}" type="video/mp4">
</video>

## Hitta filer

Hur du snabbt hittar filerna du letar efter.
Du slipper att klicka dig genom filer i projektet tills du hittar filen som innehåller koden du är ute efter.

I exemplet nedan hittar vi snabbt filer med `fd` och kodsnuttar med `rg`.
Vi använder också `fasd` för att snabbt kunna använda `cd` och `vim` i nyligen och ofta använda filer och kataloger.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/find.mp4' | relative_url }}" type="video/mp4">
</video>

## Databearbetning

Hur du snabbt och enkelt ändrar, visar, tolkar, visualiserar och beräknar på data och filer direkt från kommandoraden.
Du slipper att kopiera och klistra in från loggfiler.
Du slipper att utföra manuella statistikberäkningar över data.
Du slipper att skapa diagram i kalkylblad för enkla uppgifter.

## Kodkvalitet och kontinuerlig integration (CI)

Hur du använder verktyg för autoformatering, lintning, testning och kodtäckning för att förbättra kodkvaliteten.
Du slipper fulkod.
Du slipper regressioner.
Du slipper kod som fungerar på din egen dator men kraschar hos alla andra.

## Bortom koden

Hur du skriver bra dokumentation, kommunicerar tydligt med ansvariga för öppen källkod-projekt, skickar in åtgärdbara ärenden och bidrar med ändringsförfrågningar (pull requests) som faktiskt blir sammanslagna.
Du slipper förvirrade användare som inte kommer igång med din programvara.
Du minskar risken att mötas av tystnad från de ansvariga.

# Avslutande ord

Allt det här och mer därtill täcks i kursens nio föreläsningar, där varje föreläsning innehåller övningar för att du ska kunna bekanta dig med verktygen på egen hand.
Om du inte vill vänta till januari 2026 kan du också titta på föreläsningarna från [tidigare kurstillfälle]({{ '/2020/' | relative_url }}), som täcker flera av samma ämnen.

Vi hoppas att vi ses i januari, antingen virtuellt eller på plats!

Lycka till med hackandet,<br>
[Anish](https://anish.io/), [Jon](https://thesquareplanet.com/) och [Jose](https://josejg.com/)
