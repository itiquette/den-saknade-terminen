---
layout: lecture
title: "Bortom koden"
description: >
  Lär dig viktiga mjuka färdigheter, inklusive dokumentation, normer i öppen källkodsgemenskap och AI-etikett.
thumbnail: /static/assets/thumbnails/2026/lec8.png
date: 2026-01-22
ready: true
video:
  aspect: 56.25
  id: 2DOEATfXT8k
---

Att vara en bra programvaruingenjör handlar inte bara om att skriva fungerande kod.
Det handlar om att skriva kod som andra (även ditt framtida jag) kan förstå, underhålla och bygga vidare på.
Det handlar om att kommunicera tydligt, bidra genomtänkt och att vara en god medborgare i de ekosystem du deltar i --- oavsett om de bygger på öppen källkod eller är proprietära.

# Envägskommunikation

En stor del av programvaruingenjörsarbete handlar om att skriva för människor som saknar din nuvarande kontext: teammedlemmar som ansluter senare, de som tar över ansvaret för din kod eller du själv om sex månader när du har glömt varför du gjorde ett visst val.
Ett nyckelråd för den här typen av skrivande är att målet är att fånga och förmedla *varför*, inte bara *vad*.
Vad-frågan brukar vara självförklarande, medan *varför* är dyrköpt kunskap som lätt går förlorad över tid.

Den vanligaste formen av kommunikation mellan ingenjörer (förutom själva koden) är kanske kodkommentarer.
Jag har personligen upplevt att många kodkommentarer är värdelösa.
Men de behöver inte vara det.
Bra kommentarer förklarar sådant som koden själv inte kan: *varför* något utförs på ett visst sätt, inte *hur* det fungerar (det visar koden).
De kan spara timmar av förvirring, medan dåliga kommentarer tillför brus eller, ännu värre, vilseleder.

Typer av kommentarer som nästan alltid är värda att skriva:

- **TODOs**: Markera ofullständig eller opolerad kod, men lämna tillräckligt med kontext så att någon annan förstår vad som återstår och varför det sköts upp.
  "TODO: optimera" är värdelöst. "TODO: den här O(n²)-slingan är okej för `n<100`, men behöver indexering om vi skalar" går att agera på.
- **Referenser**: Länka till externa källor när koden implementerar en algoritm från en artikel, anpassar kod från annat håll eller kodar beteende som specificeras i dokumentation.
  Använd permanenta länkar.
  Notera eventuella avvikelser från referensen.
- **Korrekthetsargument**: Förklara *varför* icke-trivial kod ger korrekta resultat.
  Koden visar stegen.
  En kommentar förklarar varför stegen fungerar.
- **Lärdomar den hårda vägen**: Om du har lagt 30+ minuter på att felsöka något och fixen är en icke-uppenbar trollformel, dokumentera den.
  Ditt tidigare jag insåg inte att den behövdes.
  Framtida läsare gör det inte heller.
- **Motivering för konstanter**: Magiska tal förtjänar en förklaring.
  Varför 1492?
  Varför 16 bitar?
  Valdes det slumpmässigt, härlett från testning eller krävs det för korrekthet?
  Även "valdes godtyckligt" är nyttig information.
- **Bärande designval**: Om korrektheten beror på en till synes oskyldig implementationsdetalj (t.ex. "måste vara en BTreeSet eftersom iterationsordningen spelar roll nedan"), säg det uttryckligen.
- **"Varför inte"**: När du medvetet undviker den uppenbara vägen, förklara varför.
  Annars kommer någon att "fixa" det senare och förstöra saker.

README-filer (du har en, eller hur?) är också en vanlig första kontaktpunkt med andra utvecklare.
En bra README besvarar fyra frågor direkt.
Vad gör detta?
Varför ska jag bry mig?
Hur använder jag det?
Hur installerar jag det?
I den ordningen.
Strukturera den som en tratt: en kort sammanfattning och kanske en visuell demonstration högst upp så att någon på sekunder kan avgöra om detta löser problemet, och bygg sedan gradvis på med djup.
Visa användning före installation --- människor vill se vad de får innan de lägger tid på installationssteg.

Incheckningsmeddelanden är en annan typ av "skrivande för andra" som ofta försummas.
De skrivs ofta som "fixed blah" eller "added foo", och även om det kan räcka ibland är det lätt att glömma att de utgör den historiska dokumentationen av *varför* kodbasen utvecklades som den gjorde.
När någon (inklusive du själv) kör `git blame` för att förstå en förvirrande ändring bör bra incheckningsmeddelanden ge svar.

Generellt bör meddelandets brödtext svara på:
- Vilket problem tvingade fram den här ändringen?
- Vilka alternativ övervägde du?
- Vilka avvägningar eller konsekvenser finns?
- Vad kan vara överraskande med den här lösningen?

> Skala såklart detaljnivån med komplexiteten.
> En enradig stavfelsfix behöver bara ett ämnesfält.
> En subtil fix av ett kapplöpningsproblem som tog timmar att felsöka förtjänar stycken som förklarar problem och lösning.

För komplexa ändringar kan det vara användbart att följa strukturen Problem → Lösning → Konsekvenser.
Börja med begränsningen eller den tvingande faktorn.
Förklara sedan vad som ändrades och de viktigaste designbesluten.
Lista därefter viktiga följder (positiva och negativa).
Den sista delen är särskilt viktig.
Verklig ingenjörskonst handlar om att balansera målkonflikter, och att dokumentera att en avvägning var avsiktlig förhindrar att framtida utvecklare tror att du missade problemet.

LLM:er _kan_ vara hjälpsamma när man skriver incheckningsmeddelanden.
Om du bara riktar en modell mot ändringen och ber den skriva incheckningsmeddelandet har LLM:en dock bara tillgång till _vad_, inte _varför_.
Resultatet blir då mest beskrivande (motsatsen till vad vi vill ha).
Om du använde en LLM för att hjälpa till med ändringen från början är det ofta betydligt bättre att be LLM:en skriva incheckningen i samma session, eftersom konversationen i sig är en rik källa till kontext om ändringen.
Annars, eller utöver det, är ett bra trick att uttryckligen säga att du vill ha ett incheckningsmeddelande fokuserat på "varför" (och andra nyanser enligt råden ovan), och sedan _be den fråga dig om saknad kontext_.
I praktiken agerar du som ett MCP-"verktyg" för kodagenten, som den kan använda för att "läsa" kontext.

När ändringarna blir mer komplexa ska du också se till att dela upp incheckningar logiskt (`git add -p` är din vän).
Varje incheckning bör representera en sammanhängande ändring som kan förstås och granskas självständigt.
Blanda inte refaktorering med nya funktioner och kombinera inte orelaterade felrättningar.
Det gör historien grumlig kring vilka ändringar som löste vilket problem, och det kommer nästan säkert att bromsa den slutliga granskningen av ändringarna.
Det ger dig också superkrafter via `git bisect`, men det är en historia för en annan gång.

> När du börjar skriva mer teknisk text är det värt att tänka på att respektera läsarens tid.
> Det är lätt att överförklara när man väl har börjat, men du måste stå emot impulsen, annars läser läsaren _inget_ av det du skrivit.
> Förklara "varför" och lita på att läsaren kan ta reda på "hur" för sin situation.

# Samarbete

Som ingenjörer kan vi lägga en stor del av jobbet på att koda vid vårt eget tangentbord, men en betydande del av tiden går också till kommunikation med andra.
Den tiden är oftast uppdelad i samarbete och utbildning, och vinsten av att bli bättre på båda är stor.

## Bidra

Oavsett om du skickar en felrapport, bidrar med en enkel felrättning eller implementerar en stor funktion är det värt att komma ihåg att det oftast finns långt fler användare än bidragsgivare, och långt fler bidragsgivare än de som ansvarar för projekten.
Som följd är de ansvariges tid hårt belastad.
Om du vill öka sannolikheten att ditt bidrag leder någonstans produktivt behöver du se till att bidraget har hög signal och lågt brus, och är värt de ansvariges tid.

En bra felrapport respekterar till exempel den ansvariges tid genom att ge allt som behövs för att förstå och reproducera problemet:

- **Miljö**: OS, versionsnummer, relevant konfiguration.
- **Vad du förväntade dig** kontra **vad som faktiskt hände**.
- **Steg för att reproducera**: Var specifik.
  "Klicka på knappen" är mindre användbart än "Klicka på knappen Submit på sidan /settings medan du är inloggad som administratör."
- **Vad du redan har provat**: Detta förhindrar dubbla förslag och visar att du har gjort en egen undersökning.

> Om du hittar en säkerhetssårbarhet ska du inte posta den publikt.
> Kontakta de ansvariga privat först och ge dem rimlig tid att fixa den innan offentliggörande.
> Många projekt har en `SECURITY.md` eller liknande för detta ändamål.

**Se till att du söker efter befintliga ärenden.** Din felrapport eller ändringsönskan kanske redan är rapporterad, och det är bättre att tillföra information i befintliga diskussioner än att skapa dubbletter.
Det minskar dessutom brus för de ansvariga.

Minimala reproducerbara exempel är guld, om du kan få fram ett.
De sparar den ansvarige enormt mycket tid och arbete, och att reproducera programfelet pålitligt är ofta den svåraste delen av att fixa det.
Arbetet du lägger på att isolera problemet hjälper dessutom ofta dig att förstå det bättre och leder ibland till att du hittar lösningen själv.

Om du inte får svar direkt, kom ihåg att de ansvariga ofta är volontärer med begränsad tid.
Om du väntar på svar är en artig uppföljning efter ett par veckor okej.
Dagliga pingar är det inte.
På samma sätt är "jag också"-kommentarer, eller felrapporter som bara är inklistrad terminalutskrift, oftast kontraproduktiva för möjligheten att frågan får fart.

Om du vill bidra med kod bör du också sätta dig in i riktlinjerna för bidrag.
Många projekt har en `CONTRIBUTING.md` --- följ den.
Det är oftast bra att börja smått.
En stavningsrättning eller dokumentationsförbättring är ett utmärkt första bidrag eftersom det hjälper dig att lära dig projektets processer utan att samtidigt behöva gå många vändor kring innehållet.

> Kontrollera vilken licens projektet använder, eftersom all kod du bidrar med hamnar under samma licens.
> Var särskilt uppmärksam på copyleft-licenser (som GPL), som kräver att derivat också är öppen källkod och kan få konsekvenser för din arbetsgivare om du rör sådan kod.
> [choosealicense.com](https://choosealicense.com/) har ytterligare användbar information.

När du har bestämt dig för att öppna en ändringsförfrågan (PR), se först till att isolera ändringen du faktiskt vill få accepterad.
Om din ändringsförfrågan samtidigt ändrar en massa andra orelaterade saker är chansen stor att granskaren skickar tillbaka den och ber dig städa upp.
Det liknar hur du bör dela upp git-incheckningar i semantiskt relaterade delar.

I vissa fall, om du har många till synes spretiga ändringar men alla behövs för att möjliggöra en funktion, kan det vara okej att öppna en större ändringsförfrågan som fångar allt.
I så fall är disciplin i incheckningar extra viktig så att de ansvariga kan välja att granska ändringen en incheckning i taget.

Se sedan till att du förklarar "varför" bakom ändringen väl.
Beskriv inte bara _vad_ som ändrades --- förklara _varför_ ändringen behövs och _varför_ detta är ett bra sätt att lösa problemet.
Du bör också proaktivt lyfta delar av ändringen som förtjänar särskild uppmärksamhet i granskningen, om sådana finns.
Beroende på `CONTRIBUTING.md` och typen av ändring kan granskare också förvänta sig ytterligare information, till exempel vilka avvägningar du gjort eller hur ändringen testas.

> Vi rekommenderar att bidra tillbaka till ursprungsprojektet i stället för att skapa en egen avgrening, åtminstone som första strategi.
> Att skapa en avgrening (om licensen tillåter) bör reserveras för fall där de bidrag du vill göra ligger utanför ramarna för originalprojektet.
> Om du skapar en avgrening, se till att du erkänner originalprojektet.

AI gör det väldigt enkelt att snabbt generera kod och ändringsförfrågningar som ser rimliga ut.
Det ursäktar dig dock inte från att förstå vad du bidrar med.
Om du skickar in AI-genererad kod som du inte kan förklara belastar du de ansvariga med att granska och eventuellt underhålla kod som inte ens författaren förstår.
Det är okej att använda AI för att hjälpa dig identifiera problem och ta fram lösningar/funktioner, **så länge du fortfarande gör ditt grundarbete** och förädlar resultatet till ett bidrag som är värt att ta in, i stället för att lämna det arbetet till (redan hårt belastade) ansvariga.

Kom ihåg att när de ansvariga accepterar en ändringsförfrågan accepterar de också ett långsiktigt ansvar.
De kommer att underhålla koden långt efter att bidragsgivaren har gått vidare, och kan därför tacka nej till ändringar som är välmenande men inte passar projektets riktning, tillför komplexitet de inte vill underhålla eller där behovet helt enkelt inte är tillräckligt väl dokumenterat.
Det är _du_ som bidragsgivare som måste argumentera för varför det är värt underhållsbördan att acceptera bidraget.

> När du får återkoppling på en ändringsförfrågan, skilj på dig själv och din kod.
> Granskare försöker göra koden bättre, inte kritisera dig personligen.
> Ställ förtydligande frågor om du inte håller med.
> Du eller granskaren kanske lär dig något.

## Granska

Du kanske tänker att kodgranskning är något seniora utvecklare gör, men du kommer sannolikt att bli ombedd att granska kod betydligt tidigare än du tror, och ditt perspektiv är värdefullt.
Nya ögon fångar sådant erfarna utvecklare missar, och frågor från någon som är mindre bekant med koden avslöjar ofta antaganden som borde dokumenteras eller förenklas.

Granskning är också ett av de snabbaste sätten att lära sig.
Du ser hur andra angriper problem, plockar upp mönster och idiom och utvecklar intuition för vad som gör kod läsbar.
Utöver personlig utveckling fångar kodgranskningar programfel innan de når produktion, sprider kunskap i teamet och förbättrar kodkvaliteten genom samarbete.
De är inte bara byråkrati.

Bra kodgranskning är en färdighet du behöver träna upp över tid, men det finns några tips som kan göra den betydligt bättre på kort tid:

- **Granska koden, inte personen**: "Den här funktionen är svår att förstå" jämfört med "Du skrev förvirrande kod."
- **Föredra handlingsbara kommentarer**: "Kan du ersätta dessa globala variabler med en konfigurations-dataklass" är lättare att agera på än "Använd inte globala variabler här"
- **Ställ frågor i stället för att ställa krav**: "Vad händer om X är nullvärde här?" bjuder in till diskussion bättre än "Hantera fallet med nullvärde."
- **Förklara "varför"**: "Överväg att använda en konstant här" är mindre användbart än "Överväg att använda en konstant här så att vi enkelt kan justera tidsgränsen utifrån miljö."
- **Skilj blockerande problem från förslag**: Var tydlig med vad som måste ändras kontra vad som är en smakfråga.
- **Uppmärksamma det som är bra**: Att lyfta smarta lösningar eller rena implementationer är uppmuntrande och hjälper författaren att veta vad hen ska fortsätta med.
- **Vet när du ska sluta**: Bidragsgivare har begränsat med tid och tålamod, och den tiden används inte alltid bäst till att hantera alla smånitar.
  Fokusera på de stora sakerna och överväg att städa upp småsaker själv i efterhand.

> AI-verktyg kan fånga vissa problem, men de ersätter inte mänsklig granskning.
> De missar kontext, förstår inte produktkrav och kan självsäkert föreslå fel saker.
> De är värda att använda som första pass, men inte som ersättning för tänkande mänsklig granskning.

# Utbildning

En stor del av vår icke-kodtid som ingenjörer går till att antingen ställa eller besvara frågor, ibland en blandning av båda, under samarbete, i dialog med kollegor eller när vi försöker lära oss.
Att ställa bra frågor är en färdighet som gör dig bättre på att lära av vem som helst, inte bara av perfekta pedagoger.
Julia Evans har utmärkta bloggposter om "[How to ask good questions](https://jvns.ca/blog/good-questions/)" (hur man ställer bra frågor) och "[How to get useful answers to your questions](https://jvns.ca/blog/2021/10/21/how-to-get-useful-answers-to-your-questions/)" (hur man får användbara svar på sina frågor), och de är värda att läsa.

Några särskilt värdefulla råd är:

- **Beskriv din förståelse först**: Säg vad du tror att du vet och fråga "stämmer det?".
  Det hjälper den som svarar att identifiera dina verkliga kunskapsluckor.
- **Ställ ja/nej-frågor**: "Är X sant?" förhindrar utsvävande förklaringar och leder ofta ändå till nyttig utveckling.
- **Var specifik**: "Hur fungerar SQL-joins?" är för vagt.
  "Omfattar en LEFT JOIN rader där högra tabellen inte har någon match?" går att besvara.
- **Säg till när du inte förstår**: Avbryt för att fråga om obekanta termer.
  Det signalerar självförtroende, inte svaghet.
  På samma sätt, om någon ställer frågor till dig som du inte kan svaret på, är det bäst att säga "jag vet inte", och eventuellt följa upp med "men jag tror ..." eller "men jag kan ta reda på det".
- **Acceptera inte ofullständiga svar**: Fortsätt ställa följdfrågor tills du faktiskt förstår.
- **Gör lite förarbete först**: Grundläggande förarbete hjälper dig att ställa mer träffsäkra frågor (även om informella frågor mellan kollegor förstås är helt okej).

Kom ihåg att välformulerade frågor gynnar hela gemenskaper.
De synliggör dolda antaganden som andra också behöver förstå.

> Observera att råden gäller lika mycket när du kommunicerar med LLM:er.

# AI-etikett

I takt med den växande användningen av LLM:er och AI inom programvaruområdet är de sociala och professionella normerna fortfarande i förändring.
Vi har redan täckt många taktiska överväganden i [föreläsningen om agentdriven kodning]({{ '/2026/agentic-coding/' | relative_url }}), men det finns också "mjukare" delar av användningen som är värda att diskutera.

Den första är att om AI bidrog meningsfullt till ditt arbete ska du **berätta det**.
Det handlar inte om skam.
Det handlar om ärlighet, att sätta rätt förväntningar och att säkerställa att resultatet får rätt granskningsnivå.
Det är också värt att berätta vilka _delar_ du använder AI till.
Det finns en betydande skillnad mellan "det här är helt vibekodat" och "jag skrev det här säkerhetskopieringsverktyget och använde en LLM för att utforma webbgränssnittet".
Vi har till exempel använt LLM:er för att hjälpa till att skriva delar av dessa föreläsningsanteckningar, inklusive korrekturläsning, brainstorming och framtagning av första utkast till kodsnuttar och övningar.

Du behöver också följa normerna i teamen och projekten du bidrar till.
Vissa team har striktare policyer kring användning av AI än andra (t.ex. av compliance- eller data residency-skäl), och du vill inte råka bryta mot dem.
Öppenhet om hur du använder AI hjälper till att förebygga potentiellt kostsamma misstag.

> Om ditt mål är att lära dig i arbetet, tänk på att om du låter AI göra allt eller nästan allt arbete åt dig kan det motverka syftet.
> Du lär dig sannolikt mer om hur man skriver uppmaningar (och kanske granskning av AI-utdata) än om själva uppgiften.
> Särskilt i lärandesituationer kan poängen vara resan, inte destinationen, så att använda AI för att "snabbt få lösningen" är ett anti-mål.

En relaterad fråga uppstår i intervjuer och andra bedömningssituationer.
De är ofta avsedda att utvärdera _dina_ färdigheter och förmågor, inte en LLM:s.
Fler företag tillåter nu att du använder LLM:er och andra AI-assisterade verktyg i intervjuer så länge de får observera interaktionerna som del av intervjun (dvs. de bedömer även din förmåga att använda dessa verktyg), men detta är fortfarande en minoritet.
Om du är osäker på om AI-hjälp ingår i ramarna för en viss uppgift, fråga.

> Det borde vara självklart att om en bedömningssituation uttryckligen säger inga externa verktyg, inga LLM:er och så vidare, så ska du inte använda dem.
> Försök att göra det diskret utan att bli upptäckt **kommer** att slå tillbaka.

# Övningar

1. Bläddra i källkoden för ett välkänt projekt (t.ex. [Redis](https://github.com/redis/redis) eller [curl](https://github.com/curl/curl)).
1. Hitta exempel på några av kommentarstyperna som nämns i föreläsningen: en användbar TODO, en referens till extern dokumentation, en "varför inte"-kommentar som förklarar en undviken lösning eller en hårt lärd läxa.
1. Vad skulle gå förlorat om den kommentaren inte fanns?

1. Välj ett öppen källkod-projekt du är intresserad av och titta på dess senaste incheckningshistorik (`git log`).
1. Hitta en incheckning med ett bra meddelande som förklarar *varför* ändringen gjordes, och en incheckning med ett svagt meddelande som bara beskriver *vad* som ändrades.
1. För den svaga incheckningen, titta på diffen (`git show <hash>`) och försök skriva ett bättre incheckningsmeddelande enligt strukturen Problem → Lösning → Konsekvenser.
1. Lägg märke till hur mycket arbete som krävs för att återskapa nödvändig kontext i efterhand.

1. Jämför README-filerna för tre GitHub-projekt med 1000+ stjärnor.
1. Är alla lika användbara?
1. Leta efter sådant som mest känns som brus för dig som en lärdom inför framtida README-filer du själv skriver.

1. Hitta ett öppet ärende i ett projekt du använder (kolla etiketter som "good first issue" eller "help wanted" om de finns).
1. Utvärdera ärendet mot kriterierna från föreläsningen.
1. Verkar det värdera den ansvariges tid och innehålla all information som behövs för felsökning, eller förväntar du dig att den ansvarige behöver flera frågerundor med den som rapporterat för att nå rotproblemet?

1. Tänk på ett programfel du har stött på i programvara du använder (eller hitta ett i ett ärendehanteringssystem).
1. Öva på att skapa ett minimalt reproducerbart exempel.
1. Skala bort allt som inte är relaterat till programfelet tills du har minsta fallet som fortfarande demonstrerar problemet.
1. Skriv ner vad du tog bort och varför.

1. Hitta en sammanslagen ändringsförfrågan (PR) i ett projekt du känner till som har substantiella granskningskommentarer (inte bara "LGTM").
1. Läs igenom granskningen.
1. Var alla kommentarer lika produktiva?
1. Om du vore författaren till ändringsförfrågan, hur skulle du uppleva att få alla dessa kommentarer?

1. Gå till Stack Overflow och hitta en fråga inom en teknik du kan som har ett högt uppröstat svar.
1. Hitta sedan en fråga som stängts eller röstats ned kraftigt.
1. Jämför dem med råden från föreläsningen.
1. Gick det att förutse vilken fråga som skulle få bättre svar?
