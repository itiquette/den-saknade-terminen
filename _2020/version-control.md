---
layout: lecture
title: "Versionshantering (Git)"
description: >
  Lär dig Gits datamodell och hur du använder Git för versionshantering och samarbete.
thumbnail: /static/assets/thumbnails/2020/lec6.png
date: 2020-01-22
ready: true
video:
  aspect: 56.25
  id: 2sjqTHE0zok
---

Versionshanteringssystem (VCS:er) är verktyg som används för att spåra ändringar i källkod (eller andra samlingar av filer och kataloger).
Som namnet antyder hjälper dessa verktyg till att bevara en historik över ändringar.
Dessutom underlättar de samarbete.
VCS:er spårar ändringar i en katalog och dess innehåll som en serie ögonblicksbilder (_snapshots_), där varje ögonblicksbild kapslar in hela tillståndet för filer och kataloger inom en toppnivåkatalog.
VCS:er lagrar också metadata som vem som skapade varje ögonblicksbild, meddelanden kopplade till den och så vidare.

Varför är versionshantering användbart?
Även när du arbetar själv kan det hjälpa dig att titta på gamla ögonblicksbilder av ett projekt, föra logg över varför vissa ändringar gjordes, arbeta på parallella utvecklingsgrenar och mycket mer.
När du arbetar med andra är det ett ovärderligt verktyg för att se vad andra har ändrat, och för att lösa konflikter i samtidig utveckling.

Moderna VCS:er låter dig också enkelt (och ofta automatiskt) svara på frågor som:

- Vem skrev den här modulen?
- När redigerades den här specifika raden i den här specifika filen?
  Av vem?
  Varför redigerades den?
- Under de senaste 1000 revisionerna, när/varför slutade ett visst enhetstest att fungera?

Även om det finns andra VCS:er är **Git** i praktiken standarden för versionshantering.
Den här [XKCD-serien](https://xkcd.com/1597/) beskriver Git på ett träffande sätt:

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

Eftersom Gits gränssnitt är en läckande abstraktion kan det vara förvirrande att lära sig Git uppifrån och ner (med start i kommandoraden).
Det går att memorera en handfull kommandon och behandla dem som magiska trollformler, och följa tillvägagångssättet i serien ovan när något går fel.

Även om Git onekligen har ett fult gränssnitt är den underliggande designen och idéerna vackra.
Ett fult gränssnitt måste _memoreras_, men en vacker design kan _förstås_.
Därför ger vi en förklaring nerifrån och upp av Git, med start i datamodellen och därefter kommandoraden.
När datamodellen väl är förstådd blir kommandona lättare att förstå i termer av hur de manipulerar den underliggande datamodellen.

# Gits datamodell

Det finns många ad hoc-sätt att närma sig versionshantering.
Git har en välgenomtänkt modell som möjliggör alla fina funktioner i versionshantering, som att bevara historik, stödja grenar och möjliggöra samarbete.

## Ögonblicksbilder {#snapshots}

Git modellerar historiken för en samling filer och kataloger i en toppnivåkatalog som en serie ögonblicksbilder.
I Git-terminologi kallas en fil en "blob", och den är bara en samling av byte.
En katalog representeras av ett trädobjekt (`tree`), som mappar namn till blobbar eller andra trädobjekt (så kataloger kan innehålla andra kataloger).
En ögonblicksbild är det toppnivåträdobjekt (`tree`) som spåras.
Till exempel kan vi ha ett träd enligt följande:

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

Toppnivåträdet innehåller två element, ett trädobjekt `foo` (som i sin tur innehåller ett element, en blob `bar.txt`) och en blob `baz.txt`.

## Modellera historik: relatera ögonblicksbilder

Hur ska ett versionshanteringssystem relatera ögonblicksbilder?
En enkel modell vore att ha en linjär historik.
En historik vore då en lista av ögonblicksbilder i tidsordning.
Av flera skäl använder Git inte en så enkel modell.

I Git är historik en riktad acyklisk graf (DAG) av ögonblicksbilder.
Det kan låta som ett fint matematikord, men låt dig inte avskräckas.
Allt det betyder är att varje ögonblicksbild i Git refererar till en uppsättning "föräldrar", ögonblicksbilderna som kom före den.
Det är en uppsättning föräldrar i stället för en enda förälder (som i en linjär historik), eftersom en ögonblicksbild kan härstamma från flera föräldrar, till exempel genom att slå samman två parallella utvecklingsgrenar.

Git kallar dessa ögonblicksbilder för incheckningar.
En visualisering av en incheckningshistorik kan se ut ungefär så här:

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

I ASCII-bilden ovan motsvarar `o`:na individuella incheckningar (ögonblicksbilder).
Pilarna pekar på föräldern till varje incheckning (det är en "kommer före"-relation, inte "kommer efter").
Efter den tredje incheckningen delar historiken upp sig i två separata grenar.
Det kan till exempel motsvara två separata funktioner som utvecklas parallellt, oberoende av varandra.
I framtiden kan dessa grenar slås samman för att skapa en ny ögonblicksbild som innehåller båda funktionerna, och ge en ny historik som ser ut så här, med den nyskapade incheckningen för sammanslagningen i fetstil:

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong> ^ / \ v --- o <-- o
</code>
</pre>

Incheckningar i Git är oföränderliga.
Det betyder dock inte att misstag inte kan rättas.
Det betyder bara att "redigeringar" av incheckningshistoriken i själva verket skapar helt nya incheckningar, och referenser (se nedan) uppdateras för att peka på de nya.

## Datamodell, som pseudokod

Det kan vara lärorikt att se Gits datamodell nedskriven i pseudokod:

```
// en fil är en samling byte
type blob = array<byte>

// en katalog innehåller namngivna filer och kataloger
type tree = map<string, tree | blob>

// en incheckning har föräldrar, metadata och toppnivåträdet
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

Det är en ren och enkel historikmodell.

## Objekt och innehållsadressering

Ett "object" är en blob, tree eller commit:

```
type object = blob | tree | commit
```

I Gits datalager är alla objekt innehållsadresserade med sin [SHA-1-hash](https://en.wikipedia.org/wiki/SHA-1).

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobbar, trädobjekt (`tree`) och incheckningar förenas på detta sätt.
De är alla objekt.
När de refererar till andra objekt _innehåller_ de dem inte i sin representation på disk, utan har en referens till dem via deras hash.

Till exempel ser trädobjektet (`tree`) för exempelkatalogstrukturen [ovan](#snapshots) (visualiserat med `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`) ut så här:

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

Trädobjektet innehåller pekare till sitt innehåll, `baz.txt` (en blob) och `foo` (ett trädobjekt).
Om vi tittar på innehållet som adresseras av hashen som motsvarar `baz.txt` med `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`, får vi följande:

```
git is wonderful
```

## Referenser

Nu kan alla ögonblicksbilder identifieras med sina SHA-1-hashar.
Det är opraktiskt, eftersom människor inte är bra på att minnas strängar med 40 hexadecimala tecken.

Gits lösning på det här problemet är referenser: lättlästa namn som pekar på SHA-1-hashar.
Referenser är pekare till incheckningar.
Till skillnad från objekt, som är oföränderliga, är referenser föränderliga (de kan uppdateras för att peka på en ny incheckning).
Till exempel pekar referensen `master` vanligtvis på den senaste incheckningen i huvudgrenen för utveckling.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

Med detta kan Git använda namn som "master" för att referera till en viss ögonblicksbild i historiken, i stället för en lång hexadecimal sträng.

En detalj är att vi ofta vill ha en uppfattning om "var vi befinner oss just nu" i historiken, så att vi när vi tar en ny ögonblicksbild vet vad den är relativ till (hur vi sätter fältet `parents` i incheckningen).
I Git är detta "var vi befinner oss" en särskild referens som kallas "HEAD".

## Kodförråd

Till sist kan vi definiera vad ett Git-kodförråd (ungefär) är.
Det är datan `objects` och `references`.

På disk är allt Git lagrar objekt och referenser.
Det är hela Gits datamodell.
Alla `git`-kommandon motsvarar någon manipulation av DAG:en för incheckningar genom att lägga till objekt och lägga till/uppdatera referenser.

Varje gång du skriver ett kommando, tänk på vilken manipulation kommandot gör i den underliggande grafdatastrukturen.
Omvänt, om du försöker göra en viss förändring i DAG:en för incheckningar, t.ex. "kasta bort icke-incheckade ändringar och låt refen 'master' peka på incheckningen `5d83f9e`", finns det sannolikt ett kommando för det (t.ex. i detta fall `git checkout master; git reset --hard 5d83f9e`).

# Köytan (staging area)

Det här är ett annat koncept som ligger vid sidan av datamodellen, men som ändå är en del av gränssnittet för att skapa incheckningar.

Ett sätt att förstå upplägget ovan är att tänka sig ett kommando som skapar ögonblicksbilder utifrån _nuvarande tillstånd_ i arbetskatalogen.
Vissa versionshanteringsverktyg fungerar så, men inte Git.
Vi vill ha rena ögonblicksbilder, och det är inte alltid optimalt att skapa en ögonblicksbild från det nuvarande tillståndet.
Tänk dig till exempel ett scenario där du implementerat två separata funktioner och vill skapa två separata incheckningar, där den första introducerar den första funktionen och nästa introducerar den andra funktionen.
Eller tänk dig ett scenario där du lagt till felsökningsutskrifter över hela koden tillsammans med en felrättning.
Du vill göra en incheckning med felrättningen samtidigt som du kastar bort alla utskriftssatser.

Git hanterar sådana scenarier genom att låta dig specificera vilka modifieringar som ska ingå i nästa ögonblicksbild via en mekanism som kallas indexet (staging area), alltså en köyta för vad som ska checkas in.

# Git-kommandoradsgränssnitt

För att undvika att duplicera information kommer vi inte förklara kommandona nedan i detalj i dessa föreläsningsanteckningar.
Se den varmt rekommenderade [Pro Git](https://git-scm.com/book/sv/v2) för mer information, eller titta på föreläsningsvideon.

## Grunder

{% comment %}

Kommandot `git init` initierar ett nytt Git-kodförråd, där metadata för kodförrådet lagras i katalogen `.git`:

```console
$ mkdir myproject
$ cd myproject
$ git init
Initialized empty Git repository in /home/missing-semester/myproject/.git/
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Hur ska vi tolka den här utdata?
"No commits yet" betyder i princip att versionshistoriken är tom.
Låt oss ändra på det.

```console
$ echo "hej, git" > hej.txt
$ git add hej.txt
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   hej.txt

$ git commit -m 'Initial commit'
[master (root-commit) 4515d17] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 hej.txt
```

Här har vi lagt till en fil i indexet med `git add`, och sedan gjort en incheckning med `git commit` med det enkla meddelandet "Initial commit".
Om vi inte anger flaggan `-m` öppnar Git vår textredigerare så att vi kan skriva ett incheckningsmeddelande.

Nu när vi har en icke-tom historik kan vi visualisera den.
Att visualisera historiken som en DAG kan vara särskilt hjälpsamt för att förstå kodförrådets nuvarande tillstånd och koppla det till datamodellen i Git.

Kommandot `git log` visualiserar historiken.
Som standard visar det en tillplattad variant som döljer grafstrukturen.
Om du använder ett kommando som `git log --all --graph --decorate` får du hela versionshistoriken visualiserad som en graf.

```console
$ git log --all --graph --decorate
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa (HEAD -> master)
  Author: Missing Semester <missing-semester@mit.edu>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

Det här ser inte särskilt grafiskt ut eftersom det bara innehåller en nod.
Låt oss göra fler ändringar, skapa en ny incheckning och visualisera historiken igen.

```console
$ echo "en rad till" >> hej.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   hej.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add hej.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   hej.txt

$ git commit -m 'Add a line'
[master 35f60a8] Add a line
 1 file changed, 1 insertion(+)
```

Om vi nu visualiserar historiken igen ser vi mer av grafstrukturen:

```
* commit 35f60a825be0106036dd2fbc7657598eb7b04c67 (HEAD -> master)
| Author: Missing Semester <missing-semester@mit.edu>
| Date:   Tue Jan 21 22:26:20 2020 -0500
|
|     Add a line
|
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa
  Author: Anish Athalye <me@anishathalye.com>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

Notera också att den visar aktuell HEAD tillsammans med aktuell gren (`master`).

Vi kan titta på gamla versioner med kommandot `git checkout`.

```console
$ git checkout 4515d17  # tidigare incheckningshash; din blir annorlunda
Note: checking out '4515d17'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 4515d17 Initial commit
$ cat hej.txt
hej, git
$ git checkout master
Previous HEAD position was 4515d17 Initial commit
Switched to branch 'master'
$ cat hej.txt
hej, git
en rad till
```

Git kan visa hur filer utvecklats (skillnader, eller diffs) med kommandot `git diff`:

```console
$ git diff 4515d17 hej.txt
diff --git c/hej.txt w/hej.txt
index 94bab17..f0013b2 100644
--- c/hej.txt
+++ w/hej.txt
@@ -1 +1,2 @@
 hej, git
 +en rad till
```

{% endcomment %}

- `git help <kommando>`: få hjälp för ett git-kommando
- `git init`: skapar ett nytt git-kodförråd, med data lagrad i katalogen `.git`
- `git status`: berättar vad som pågår
- `git add <filnamn>`: lägger till filer i indexet
- `git commit`: skapar en ny incheckning
    - Skriv [bra incheckningsmeddelanden](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
    - Ännu fler skäl att skriva [bra incheckningsmeddelanden](https://chris.beams.io/posts/git-commit/)!
- `git log`: visar en tillplattad historiklogg
- `git log --all --graph --decorate`: visualiserar historiken som en DAG
- `git diff <filnamn>`: visa ändringar du gjort relativt till indexet
- `git diff <revision> <filnamn>`: visar skillnader i en fil mellan ögonblicksbilder
- `git checkout <revision>`: uppdaterar HEAD (och aktuell gren om du checkar ut en gren)

## Grenar och sammanslagning

{% comment %}

Grenar låter dig avgrena versionshistoriken.
Det kan vara användbart för att arbeta på oberoende funktioner eller felrättningar parallellt.
Kommandot `git branch` kan användas för att skapa nya grenar.
`git checkout -b <grennamn>` skapar en gren och checkar ut den.

Sammanslagning är motsatsen till att dela upp historiken i grenar.
Det låter dig kombinera versionshistoriker från olika grenar, t.ex. slå samman en funktionsgren tillbaka in i master.
Kommandot `git merge` används för sammanslagning.

{% endcomment %}

- `git branch`: visar grenar
- `git branch <namn>`: skapar en gren
- `git checkout -b <namn>`: skapar en gren och växlar till den
    - samma som `git branch <namn>; git checkout <namn>`
- `git merge <revision>`: slår samman med aktuell gren
- `git mergetool`: använd ett avancerat verktyg för att hjälpa till att lösa sammanslagningskonflikter
- `git rebase`: basera om en uppsättning patchar på en ny bas

## Fjärrförråd

- `git remote`: lista fjärrförråd
- `git remote add <namn> <url>`: lägg till ett fjärrförråd
- `git push <fjärrförråd> <lokal gren>:<fjärrgren>`: skicka objekt till fjärrförråd och uppdatera fjärrreferens
- `git branch --set-upstream-to=<fjärrförråd>/<fjärrgren>`: sätt upp koppling mellan lokal och fjärrgren
- `git fetch`: hämta objekt/referenser från ett fjärrförråd
- `git pull`: samma som `git fetch; git merge`
- `git clone`: klona kodförråd från fjärrförråd

## Ångra

- `git commit --amend`: redigera en inchecknings innehåll eller meddelande
- `git reset HEAD <fil>`: avköa en fil
- `git checkout -- <fil>`: kasta bort ändringar

# Avancerad Git

- `git config`: Git är [mycket anpassningsbart](https://git-scm.com/docs/git-config)
- `git clone --depth=1`: ytlig klon, utan hela versionshistoriken
- `git add -p`: lägg till interaktivt på köytan
- `git rebase -i`: interaktiv ombasering
- `git blame`: visa vem som senast redigerade vilken rad
- `git stash`: ta tillfälligt bort modifieringar i arbetskatalogen
- `git bisect`: binärsök i historiken (t.ex. efter regressioner)
- `.gitignore`: [specificera](https://git-scm.com/docs/gitignore) avsiktligt ospårade filer som ska ignoreras

# Övrigt

- **Grafiska gränssnitt**: det finns många [GUI-klienter](https://git-scm.com/downloads/guis) för Git.
  Vi använder dem inte personligen utan föredrar kommandoraden.
- **Skalintegration**: det är väldigt praktiskt att visa Git-status som en del av skalprompten ([zsh](https://github.com/olivierverdier/zsh-git-prompt), [bash](https://github.com/magicmonty/bash-git-prompt)).
  Detta ingår ofta i ramverk som [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).
- **Redigerarintegration**: liknande ovan, praktiska integrationer med många funktioner.
  [fugitive.vim](https://github.com/tpope/vim-fugitive) är standardalternativet för Vim.
- **Arbetsflöden**: vi lärde ut datamodellen plus några grundläggande kommandon.
  Vi berättade inte vilka arbetssätt du ska följa i stora projekt (och det finns [många](https://nvie.com/posts/a-successful-git-branching-model/) [olika](https://www.endoflineblog.com/gitflow-considered-harmful) [angreppssätt](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)).
- **GitHub**: Git är inte GitHub.
  GitHub har ett specifikt sätt att bidra kod till andra projekt, kallat [ändringsförfrågningar (PR:er)](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
- **Andra Git-leverantörer**: GitHub är inte unikt.
  Det finns många värdar för Git-kodförråd, som [GitLab](https://about.gitlab.com/) och [BitBucket](https://bitbucket.org/).

# Resurser

- [Pro Git](https://git-scm.com/book/en/v2) är **starkt rekommenderad läsning**.
  Att gå igenom kapitel 1--5 bör lära dig det mesta du behöver för att använda Git skickligt, nu när du förstår datamodellen.
  De senare kapitlen har intressant, avancerat material.
- [Oh Shit, Git!?!](https://ohshitgit.com/) är en kort guide för hur man återhämtar sig från vanliga Git-misstag.
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) är en kort förklaring av Gits datamodell, med mindre pseudokod och fler avancerade diagram än dessa föreläsningsanteckningar.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) är en detaljerad förklaring av Gits implementationsdetaljer bortom datamodellen, för den nyfikne.
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words) (hur man förklarar Git med enkla ord)
- [Learn Git Branching](https://learngitbranching.js.org/) är ett webbläsarbaserat spel som lär dig Git.

# Övningar

1. Om du inte har tidigare erfarenhet av Git, prova antingen att läsa de första kapitlen av [Pro Git](https://git-scm.com/book/sv/v2) eller gå igenom en handledning som [Learn Git Branching](https://learngitbranching.js.org/).
   När du arbetar igenom den, koppla Git-kommandon till datamodellen.
1. Klona [kodförrådet för kursens webbplats](https://github.com/missing-semester/missing-semester).
    1. Utforska versionshistoriken genom att visualisera den som en graf.
    1. Vem var den senaste personen som modifierade `README.md`?
       (Tips: använd `git log` med ett argument).
    1. Vilket incheckningsmeddelande hörde till den senaste modifieringen av raden `collections:` i `_config.yml`?
       (Tips: använd `git blame` och `git show`).
1. Ett vanligt misstag när man lär sig Git är att göra incheckningar med stora filer som inte borde hanteras av Git eller att lägga till känslig information.
   Prova att lägga till en fil i ett kodförråd, göra några incheckningar och sedan ta bort filen från historiken.
   Du kanske vill titta på [detta](https://help.github.com/articles/removing-sensitive-data-from-a-repository/).
1. Klona ett valfritt kodförråd från GitHub och modifiera en av dess befintliga filer.
   Vad händer när du kör `git stash`?
   Vad ser du när du kör `git log --all --oneline`?
   Kör `git stash pop` för att ångra det du gjorde med `git stash`.
   I vilket scenario kan detta vara användbart?
1. Liksom många kommandoradsverktyg tillhandahåller Git en konfigurationsfil (eller dotfil) kallad `~/.gitconfig`.
   Skapa ett alias i `~/.gitconfig` så att när du kör `git graph` får du utdata från `git log --all --graph --decorate --oneline`.
   Du kan göra detta genom att direkt [redigera](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias) filen `~/.gitconfig`, eller använda kommandot `git config` för att lägga till aliaset.
   Information om git-alias finns [här](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases).
1. Du kan definiera globala ignore-mönster i `~/.gitignore_global` efter att ha kört `git config --global core.excludesfile ~/.gitignore_global`.
   Detta anger var den globala ignore-filen ligger, men du måste fortfarande skapa filen manuellt på den sökvägen.
   Sätt upp din globala gitignore-fil så att den ignorerar OS-specifika eller redigerarspecifika temporära filer, som `.DS_Store`.
1. Skapa en avgrening av [kodförrådet för kursens webbplats](https://github.com/missing-semester/missing-semester), hitta ett stavfel eller någon annan förbättring du kan göra, och skicka en ändringsförfrågan (PR) på GitHub (du kanske vill titta på [detta](https://github.com/firstcontributions/first-contributions)).
   Skicka bara ändringsförfrågningar som är användbara (spamma oss inte, tack!).
   Om du inte hittar någon förbättring att göra kan du hoppa över övningen.
