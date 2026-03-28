---
layout: lecture
title: "Versionshantering"
presenter: Jon
date: 2019-01-22
order: 2
video:
  aspect: 56.25
  id: 3fig2Vz8QXs
---

När du arbetar med något som förändras över tid är det användbart att kunna _spåra_ ändringarna.
Det finns flera skäl: du får en historik över vad som ändrats, hur du ångrar det, vem som ändrade det, och ibland även varför.
Versionshanteringssystem (VCS) ger dig den förmågan.
De låter dig _checka in_ ändringar i en uppsättning filer, med ett meddelande som beskriver ändringen, samt granska och ångra tidigare ändringar.

De flesta VCS stöder delning av incheckningshistorik mellan flera användare.
Det möjliggör smidigt samarbete.
Du kan se ändringarna jag gjort, och jag kan se ändringarna du gjort.
Eftersom VCS spårar _ändringar_ kan systemet ofta (men inte alltid) räkna ut hur våra ändringar kan kombineras, så länge de rör relativt separata delar.

Det finns [_väldigt många_](https://en.wikipedia.org/wiki/Comparison_of_version-control_software) VCS-system, och de skiljer sig mycket i vad de stödjer, hur de fungerar, och hur man interagerar med dem.
Här fokuserar vi på [git](https://git-scm.com/), ett av de vanligaste, men jag rekommenderar att du också tittar på [Mercurial](https://www.mercurial-scm.org/).

Med det sagt, nu till snabbversionen.

## Är git mörk magi?

Inte riktigt.
Du behöver förstå datamodellen.
Vi hoppar över vissa detaljer, men i grova drag är den centrala "saken" i git en incheckning.

  - varje incheckning har ett unikt namn, en "revisionshash"
en lång hash som `998622294a6c520db718867354bf98348ae3c7e2` förkortas ofta till ett kort (ungefär unikt) prefix: `9986222`
  - en incheckning har författare + incheckningsmeddelande
  - den har också hashen för eventuella _förfäder_ oftast bara hashen för föregående incheckning
  - en incheckning representerar också en _diff_, alltså en beskrivning av hur man går från incheckningens förfäder till incheckningen
(t.ex. ta bort den här raden i en fil, lägg till de här raderna i en annan, byt namn på en fil, osv.)
   - i praktiken lagrar git hela tillståndet före och efter
   - du vill troligen inte lagra stora filer som ändras ofta

Initialt har ett kodförråd (ungefär katalogen som git hanterar) inget innehåll och inga incheckningar.
Låt oss sätta upp det:

```console
$ git init hackers
$ cd hackers
$ git status
```

Utdatan här ger faktiskt en bra utgångspunkt.
Låt oss gå igenom den och se till att vi förstår allt.

Först: "On branch master".

 - man vill inte arbeta med hashar hela tiden
  - grenar är namn som pekar på hashar
  - master är traditionellt namnet för den "senaste" incheckningen varje gång en ny incheckning skapas flyttas master till den nya incheckningens hash
 - det särskilda namnet `HEAD` betyder "aktuellt" namn
 - du kan också skapa egna namn med `git branch` (eller `git tag`) vi kommer tillbaka till det

Vi hoppar över "No commits yet" eftersom det är självförklarande.

Sedan: "nothing to commit".

  - varje incheckning innehåller en diff med alla ändringar du gjort
men hur byggs den diffen från början?
  - man _skulle_ kunna checka in _alla_ ändringar sedan senaste incheckning
    - ibland vill du bara checka in en del (t.ex. inte `TODO`s)
    - ibland vill du dela upp en ändring i flera incheckningar för att ge separata incheckningsmeddelanden
  - git låter dig köa ändringar för att konstruera en incheckning
    - lägg till ändringar i en eller flera filer till köytan med `git add`
     - lägg till bara vissa ändringar i en fil med `git add -p`
     - utan argument arbetar `git add` på "alla kända filer"
    - ta bort en fil och köa borttagningen med `git rm`
    - töm mängden köade ändringar med `git reset`
     - notera att detta *inte* ändrar några filer det betyder *bara* att inga ändringar tas med i nästa incheckning
      - för att ta bort bara vissa köade ändringar:
    `git reset FILE` eller `git reset -p`
    - se köade ändringar med `git diff --staged`
   - se återstående ändringar med `git diff`
    - när du är nöjd med köytan, skapa en incheckning med `git commit`
      - om du vill checka in *alla* ändringar direkt: `git commit -a`
     - `git help add` har mer hjälpsam information

Medan du testar ovan, försök köra `git status` för att se vad git tycker att du gör.
Det är förvånansvärt hjälpsamt.

## En incheckning säger du...

Okej, vi har en incheckning.
Vad nu?

 - vi kan titta på senaste ändringarna: `git log` (eller `git log --oneline`)
 - vi kan titta på fullständiga ändringar: `git log -p`
  - vi kan visa en specifik incheckning: `git show master`
    - eller med `-p` för full diff/patch
  - vi kan gå tillbaka till tillståndet vid en incheckning med `git checkout NAME`
    - om `NAME` är en incheckningshash säger git att vi är "detached" det betyder bara att inget `NAME` pekar på incheckningen, så om vi gör incheckningar är det ingen som känner till dem
  - vi kan återställa en ändring med `git revert NAME`
    - det applicerar diffen i incheckningen vid `NAME` i omvänd riktning
 - vi kan jämföra en äldre version med den här via `git diff NAME..`
  - `a..b` är ett incheckningsintervall.
    Om någon sida utelämnas betyder det `HEAD`.
  - vi kan visa alla incheckningar mellan två punkter med `git log NAME..`
   - `-p` fungerar här också
  - vi kan flytta `master` till en viss incheckning (och i praktiken
ångra allt efter den) med `git reset NAME`:
   - va? var inte `reset` till för köytan?
     `reset` har en "andra" form (se `git help reset`) som sätter `HEAD` till incheckningen som namnet pekar på
   - notera att detta inte ändrar några filer, `git diff` visar nu i praktiken `git diff NAME..`

## Vad betyder ett namn?

Namn är uppenbart viktiga i git.
De är nyckeln till att förstå *mycket* av vad som händer i git.
Hittills har vi pratat om incheckningshashar, master, och `HEAD`.
Men det finns mer.

  - du kan skapa egna grenar (som master) med `git branch b`
    - det skapar ett nytt namn, `b`, som pekar på incheckningen vid `HEAD`
   - du är fortfarande "på" master, så om du gör en ny incheckning uppdateras master men inte `b`
    - byt till gren med `git checkout b`
      - incheckningar du gör nu uppdaterar namnet `b`
     - byt tillbaka till master med `git checkout master`
       - då "försvinner" ändringarna i `b` ur sikte
      - detta är ett smidigt sätt att testa ändringar
 - taggar är andra namn som aldrig ändras och som har egna meddelanden används ofta för utgåvor och ändringsloggar
  - `NAME^` betyder "incheckningen före `NAME`"
   - kan appliceras rekursivt: `NAME^^^`
   - du menar _oftast_ `~` när du använder den typen av notation
     - `~` är "temporal", medan `^` går via förfäder
     - `~~` är samma som `^^`
      - med `~` kan du också skriva `X~3` för "3 incheckningar äldre än `X`"
     - du vill inte ha `^3`
   - `git diff HEAD^`
 - `-` betyder "föregående namn"
 - de flesta kommandon arbetar på `HEAD` om du inte anger annat argument

## Städa upp historiken

Din incheckningshistorik kommer _väldigt_ ofta att se ut så här:

  - `lägg till funktion x` -- kanske till och med med ett bra incheckningsmeddelande om `x`
 - `forgot to add file`
 - `fix bug`
 - `typo`
 - `typo2`
 - `actually fix`
 - `actually actually fix`
 - `tests pass`
 - `fix example code`
 - `typo`
 - `x`
 - `x`
 - `x`
 - `x`

Det är _okej_ för git, men inte särskilt hjälpsamt för ditt framtida jag eller för andra som vill förstå vad som ändrats. git låter dig städa upp detta:

  - `git commit --amend`: vik in köade ändringar i föregående incheckning
    - notera att detta _ändrar_ föregående incheckning och ger den en ny hash
  - `git rebase -i HEAD~13` är väldigt kraftfullt för varje incheckning i de senaste 13 väljer du vad som ska göras:
   - standard är `pick`: gör inget
    - `r`: ändra incheckningsmeddelande
    - `e`: ändra incheckning (lägg till eller ta bort filer)
    - `s`: slå ihop incheckning med föregående och redigera incheckningsmeddelandet
    - `f`: "fixup" -- slå ihop med föregående och kasta incheckningsmeddelandet
    - i slutet pekar `HEAD` på det som nu är sista incheckning
    - kallas ofta för att slå ihop incheckningar
   - det som faktiskt händer är: flytta tillbaka `HEAD` till rebasens startpunkt, och återapplicera incheckningar i ordning enligt dina val
 - `git reset --hard NAME`: återställ alla filer till tillståndet i `NAME` (eller `HEAD` om inget namn anges) praktiskt för att ångra ändringar

## Arbeta med andra

Ett vanligt användningsfall för versionshantering är att låta flera personer göra ändringar i samma filsamling utan att trampa varandra på tårna.
Eller rättare sagt, att säkerställa att om de gör det, så skrivs ändringarna inte bara över tyst.

git är ett _distribuerat_ VCS.
Alla har en lokal kopia av hela kodförrådet (eller åtminstone allt andra har valt att publicera).
Vissa VCS är _centraliserade_ (t.ex. subversion): en server har alla incheckningar, och klienter har bara filerna de har "checkat ut".
I princip har de bara de _aktuella_ filerna och måste fråga servern för allt annat.

Varje kopia av ett git-kodförråd kan listas som ett fjärrförråd.
Du kan kopiera ett befintligt kodförråd med `git clone ADDRESS` (i stället för `git init`).
Det skapar ett fjärrförråd som heter _origin_ och pekar på `ADDRESS`.
Du kan hämta namn och incheckningar de pekar på från ett fjärrförråd med `git fetch REMOTE`.
Alla namn på remoten blir tillgängliga som `REMOTE/NAME`, och du kan använda dem som lokala namn.

Om du har skrivåtkomst till ett fjärrförråd kan du ändra namn på fjärren så att de pekar på incheckningar du skapat via `git push`.
Till exempel, låt oss få master på fjärren `origin` att peka på samma incheckning som vår lokala master pekar på:

   - `git push origin master:master`
   - för bekvämlighet kan du sätta `origin/master` som standardmål för `git push` från aktuell gren med `-u`
   - fundera: vad gör `git push origin master:HEAD^`?

Ofta använder du GitHub, GitLab, BitBucket, eller något annat som fjärrförråd.
Det är inget "särskilt" ur gits perspektiv.
Det är bara namn och incheckningar.
Om någon ändrar master och flyttar `github/master` till sin incheckning (vi återkommer till det strax), kan du efter `git fetch github` se deras ändringar med `git log github/master`.

## Samarbete i praktiken

Hittills verkar grenar ganska meningslösa.
Du kan skapa dem, jobba i dem, men sedan då?
Till slut flyttar du väl ändå master till dem, eller?

  - vad händer om du måste fixa något medan du arbetar på en stor funktion?
 - vad händer om någon annan under tiden gör en ändring i master?

Förr eller senare måste du slå samman ändringar i en gren med ändringar i en annan, oavsett om ändringarna gjorts av dig eller någon annan. git gör detta med `git merge NAME`.
`merge` kommer att:

  - hitta senaste punkt där `HEAD` och `NAME` delade incheckningsförfader
(alltså där de divergerade)
 - försöka applicera alla dessa ändringar på aktuell `HEAD`
  - skapa en incheckning som innehåller alla ändringar
och listar både `HEAD` och `NAME` som förfäder
  - sätta `HEAD` till den incheckningens hash

När din stora funktion är klar kan du slå samman dess gren till master, och git ser till att du inte tappar ändringar från någon gren.

Om du har använt git tidigare känner du kanske igen `merge` under ett annat namn: `pull`.
När du kör `git pull REMOTE BRANCH` händer följande:

- `git fetch REMOTE`
- `git merge REMOTE/BRANCH`
- där `REMOTE` och `BRANCH`, liksom med `push`, ofta utelämnas och då används den spårade fjärrgrenen (minns `-u`)

Det här fungerar oftast bra så länge ändringarna i grenarna är separata.
Om de inte är det får du en _sammanslagningskonflikt_.
Det låter läskigt.

- en sammanslagningskonflikt betyder bara att git inte vet hur den slutliga diffen ska se ut
- git pausar och ber dig lägga klart sammanslagningsincheckningen på köytan
- öppna den konfliktande filen i redigeraren och leta efter många vinkelparenteser (`<<<<<<<`) texten ovanför `=======` är ändringen i `HEAD` sedan gemensam förfader texten under `=======` är ändringen i `NAME` sedan samma förfader
- `git mergetool` är praktiskt, eftersom det öppnar ett diffverktyg
- när du _löst_ konflikten genom att bestämma hur filen ska se ut, köa ändringarna med `git add`
- när alla konflikter är lösta, avsluta med `git commit`
  - du kan avbryta med `git merge --abort`

Du har just löst din första git-sammanslagningskonflikt. \o/ Nu kan du publicera dina färdiga ändringar med `git push`.

## När världar krockar

När du kör `push` kontrollerar git att ingen annans arbete går förlorat när du uppdaterar namnet på fjärren du skickar till.
Det sker genom att kontrollera att fjärrnamnets nuvarande incheckning är en förfader till incheckningen du skickar.
Om så är fallet kan git säkert uppdatera namnet.
Det kallas snabb framflyttning (_fast-forwarding_).
Om inte vägrar git uppdatera fjärrnamnet och säger att det har tillkommit ändringar.

Om din push nekas, vad gör du då?

- slå samman ändringar från fjärren med `git pull` (alltså `fetch` + `merge`)
- tvinga push med `--force` Då förloras andras ändringar.
  - det finns också `--force-with-lease`, som bara tvingar om fjärrnamnet inte ändrats sedan senaste `fetch`; det är klart säkrare
  - om du har rebasat lokala incheckningar som du tidigare skickat (historikomskrivning, gör helst inte det) måste du tvångsskicka
- försök återapplicera dina ändringar "ovanpå" fjärrändringarna
  - det är en ombasering (`rebase`)
    - flytta tillbaka alla lokala incheckningar sedan gemensam förfader
    - fast-forward `HEAD` till incheckningen vid fjärrnamnet
    - applicera lokala incheckningar i ordning
      - konflikter kan uppstå och behöva lösas manuellt
      - använd `git rebase --continue` eller `--abort`
    - mer [här](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
  - `git pull --rebase` startar processen åt dig
  - om man bör slå samman eller basera om är en het diskussion några bra läsningar:
    - [den här](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
    - [den här](https://web.archive.org/web/20210106220723/https://derekgourlay.com/blog/git-when-to-merge-vs-when-to-rebase/)
    - [den här](https://stackoverflow.com/questions/804115/when-do-you-use-git-rebase-instead-of-git-merge)

# Vidare läsning

[![XKCD om git](https://imgs.xkcd.com/comics/git.png)](https://xkcd.com/1597/)

 - [Learn git branching](https://learngitbranching.js.org/)
 - [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
 - [Git from the bottom up](https://jwiegley.github.io/git-from-the-bottom-up/)
 - [Git for computer scientists](https://eagain.net/articles/git-for-computer-scientists/)
 - [Oh shit, git!](https://ohshitgit.com/)
 - [The Pro Git book](https://git-scm.com/book/en/v2)

# Övningar

1. I ett kodförråd, prova att ändra en befintlig fil.
   Vad händer när du kör `git stash`?
   Vad ser du med `git log --all --oneline`?
   Kör `git stash pop` för att ångra det du gjorde med `git stash`.
   I vilket scenario kan detta vara användbart?

1. Ett vanligt misstag när man lär sig git är att checka in stora filer som inte bör hanteras av git, eller att råka lägga till känslig information.
   Prova att lägga till en fil i ett kodförråd, skapa några incheckningar, och ta sedan bort filen ur historiken (du kan titta på [det här](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).
   Om du faktiskt vill låta git hantera stora filer, titta på [Git-LFS](https://git-lfs.github.com/).

1. Git är väldigt bra för att ångra ändringar, men man behöver känna till även ovanliga lägen.
   1. Om en fil råkar ändras i en incheckning kan den återställas med `git revert`.
      Men om incheckningen innehåller flera ändringar är `revert` kanske inte bästa val.
      Hur kan vi använda `git checkout` för att återställa en filversion från en specifik incheckning?
   1. Skapa en gren, gör en incheckning i den, och ta sedan bort branchen.
      Kan du fortfarande återställa incheckningen?
      Titta på `git reflog`.
      (Obs: återställ "hängande" saker snabbt, git städar regelbundet bort incheckningar som inget pekar på.)
   1. Om man är för snabb med `git reset --hard` i stället för `git reset` kan ändringar lätt gå förlorade.
      Eftersom ändringarna var köade kan de dock återställas.
      (Titta på `git fsck --lost-found` och `.git/lost-found`.)

1. I valfritt git-kodförråd, titta i katalogen `.git/hooks`.
   Där finns skript som slutar på `.sample`.
   Om du byter namn på dem och tar bort `.sample` körs de enligt sitt namn.
   Till exempel körs `pre-commit` före en incheckning.
   Experimentera med dem.

1. Liksom många kommandoradsverktyg har `git` en konfigurationsfil (dotfile) som heter `~/.gitconfig`.
   Skapa ett alias i `~/.gitconfig` så att `git graph` ger samma utdata som `git log --oneline --decorate --all --graph` (det här är ett bra kommando för att snabbt visualisera incheckningsgrafen).

1. Git låter dig också definiera globala ignore-mönster i `~/.gitignore_global`.
   Det är användbart för att förebygga vanliga misstag, som att lägga till RSA-nycklar.
   Skapa en `~/.gitignore_global`-fil, lägg till mönstret `*rsa`, och testa att det fungerar i ett kodförråd.

1. När du blir mer van vid `git` kommer du märka återkommande uppgifter, som att redigera `.gitignore`.
   [git extras](https://github.com/tj/git-extras/blob/master/Commands.md) erbjuder många småverktyg som integrerar med `git`.
   Till exempel lägger `git ignore PATTERN` till mönstret i kodförrådets `.gitignore`, och `git ignore-io LANGUAGE` hämtar vanliga ignore-mönster för språket från [gitignore.io](https://www.gitignore.io).
   Installera `git extras` och testa verktyg som `git alias` eller `git ignore`.

1. Git-GUI-program kan ibland vara väldigt användbara.
   Prova att köra [gitk](https://git-scm.com/docs/gitk) i ett kodförråd och utforska gränssnittets olika delar.
   Kör sedan `gitk --all`.
   Vilka skillnader ser du?

1. När man väl vant sig vid kommandoradsprogram kan GUI-verktyg kännas tunga.
   En bra kompromiss är ncurses-baserade verktyg, som kan navigeras från kommandoraden men fortfarande erbjuder ett interaktivt gränssnitt.
   Git har [tig](https://github.com/jonas/tig).
   Prova att installera det och köra det i ett kodförråd.
   Du hittar användningsexempel [här](https://www.atlassian.com/blog/git/git-tig).


{% comment %}

 - forced push + `--force-with-lease`
 - git merge/rebase --abort
 - git blame
 - exercise about why rebasing public commits is bad

{% endcomment %}
