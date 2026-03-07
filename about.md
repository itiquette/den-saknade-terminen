---
layout: lecture
title: "Varför vi undervisar i den här kursen"
---

I en traditionell utbildning i datavetenskap läser du sannolikt många kurser som lär ut avancerade ämnen, allt från operativsystem till programmeringsspråk och maskininlärning.
Men på många lärosäten finns ett viktigt område som sällan täcks och ofta lämnas åt studenterna att lära sig själva: att bli trygga med verktygen.

Under åren har vi hjälpt till att undervisa i flera kurser på MIT, och gång på gång har vi sett att många studenter har begränsad kunskap om de verktyg de kan använda.
Datorer byggdes för att automatisera manuella uppgifter, men studenter utför ofta repetitiva moment för hand eller missar att dra full nytta av kraftfulla verktyg som versionshantering och textredigerare.
I bästa fall leder det till ineffektivitet och bortkastad tid; i värsta fall leder det till problem som dataförlust eller att vissa uppgifter inte går att slutföra.

De här ämnena ingår sällan i universitetets ordinarie kursplan: studenter får ofta inte lära sig hur verktygen används, eller åtminstone inte hur de används effektivt, och lägger därför tid och energi på uppgifter som _borde_ vara enkla.
Den vanliga datavetenskapsutbildningen saknar kritiska delar om datorernas ekosystem som skulle kunna göra studenters vardag betydligt enklare.

# Den saknade terminen i din datavetenskapsutbildning

För att ändra på det skapade vi en kurs som täcker alla ämnen vi anser vara avgörande för att bli en effektiv datavetare och programmerare.
Kursen är pragmatisk och praktisk, och den ger en handfast introduktion till verktyg och tekniker som du direkt kan använda i många olika situationer.
Den senaste versionen av kursen, med kraftigt omarbetat material, ges under MIT:s "Independent Activities Period" i januari 2026 — en enmånadersperiod med kortare studentdrivna kurser.
Även om själva föreläsningarna bara är tillgängliga för MIT-gemenskapen kommer vi att publicera allt kursmaterial tillsammans med videoinspelningar av föreläsningarna.

Om det här låter intressant kommer här några konkreta exempel på vad kursen lär ut:

## Kommandoskal

Hur du automatiserar vanliga och repetitiva uppgifter med alias, skript och byggsystem.
Du slipper att kopiera och klistra in kommandon från ett textdokument.
Du slipper att hamna i lägen där du måste "köra de här 15 kommandona ett efter ett".
Du slipper också missar av typen "du glömde köra det här" eller "du glömde skicka med det här argumentet".

Att snabbt söka i historiken kan till exempel spara mycket tid.
I exemplet nedan visar vi flera knep för att navigera i skalhistoriken för `convert`-kommandon.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/history.mp4' | relative_url }}" type="video/mp4">
</video>

## Versionshantering

Hur du använder versionshantering _på rätt sätt_ för att undvika katastrofer, samarbeta med andra och snabbt hitta och isolera problematiska ändringar.
Du slipper `rm -rf; git clone`.
Du slipper att fastna i sammanslagningskonflikter (eller åtminstone får du färre av dem).
Du slipper stora block med utkommenterad kod.
Du slipper att fundera på vad som fick koden att gå sönder.
Du slipper tänka "åh nej, raderade vi den fungerande koden?!".
Vi lär dig till och med hur du bidrar till andras projekt med ändringsförfrågningar (pull requests).

I exemplet nedan använder vi `git bisect` för att hitta vilken incheckning som fick ett enhetstest att fallera, och rättar det sedan med `git revert`.
<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/git.mp4' | relative_url }}" type="video/mp4">
</video>

## Textredigering

Hur du redigerar filer effektivt från kommandoraden, både lokalt och på fjärrmaskiner, och drar nytta av avancerade redigerarfunktioner.
Du slipper att kopiera filer fram och tillbaka.
Du slipper repetitiv filredigering.

Vim-makron är en av Vims bästa funktioner, och i exemplet nedan konverterar vi snabbt en HTML-tabell till CSV-format med ett nästlat Vim-makro.
<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/vim.mp4' | relative_url }}" type="video/mp4">
</video>

## Fjärrmaskiner

Hur du behåller arbetsron när du arbetar med fjärrmaskiner med SSH-nycklar och terminalmultiplexering.
Du slipper att hålla många terminaler öppna bara för att köra två kommandon samtidigt.
Du slipper skriva lösenordet varje gång du ansluter.
Du slipper förlora allt bara för att internetanslutningen bröts eller du behövde starta om din dator.

I exemplet nedan använder vi `tmux` för att hålla sessioner vid liv på fjärrservrar och `mosh` för att hantera nätverksbyten och avbrott.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/ssh.mp4' | relative_url }}" type="video/mp4">
</video>

## Hitta filer

Hur du snabbt hittar filerna du letar efter.
Du slipper att klicka dig genom filer i projektet tills du hittar den som innehåller koden du vill ha.

I exemplet nedan hittar vi snabbt filer med `fd` och kodsnuttar med `rg`.
Vi använder också `fasd` för att snabbt `cd` och `vim` i nyligen och ofta använda filer och mappar.

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="{{ '/static/media/demos/find.mp4' | relative_url }}" type="video/mp4">
</video>

## Databearbetning

Hur du snabbt och enkelt ändrar, visar, parsar, plottar och beräknar på data och filer direkt från kommandoraden.
Du slipper att kopiera och klistra in från loggfiler.
Du slipper manuella statistikberäkningar över data.
Du slipper diagram i kalkylblad för enkla uppgifter.

## Kodkvalitet och kontinuerlig integration

Hur du använder verktyg för autoformatering, lintning, testning och kodtäckning för att förbättra kodkvaliteten.
Du slipper ful kod.
Du slipper regressioner.
Du slipper kod som fungerar på din dator men kraschar hos alla andra.

## Bortom koden

Hur du skriver bra dokumentation, kommunicerar tydligt med förvaltare i öppen källkod, skickar in åtgärdbara ärenden och bidrar med ändringsförfrågningar (pull requests) som faktiskt blir sammanslagna.
Du slipper förvirrade användare som inte kommer igång med din programvara.
Du minskar risken att mötas av tystnad från förvaltare.

# Avslut

Allt detta och mer täcks i kursens nio föreläsningar, där varje föreläsning innehåller övningar så att du kan bli mer bekant med verktygen på egen hand.
Om du inte vill vänta till januari 2026 kan du också titta på föreläsningarna från [kursens tidigare omgång]({{ '/2020/' | relative_url }}), som täcker många av samma ämnen.

Vi hoppas att vi ses i januari, antingen virtuellt eller på plats!

Lycka till med hackandet,<br>
[Anish](https://anish.io/), [Jon](https://thesquareplanet.com/) och [Jose](https://josejg.com/)
