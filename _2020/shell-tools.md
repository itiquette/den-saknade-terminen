---
layout: lecture
title: "Skalverktyg och skriptning"
description: >
  Lär dig skriva skalskript och använda kraftfulla kommandoradsverktyg.
thumbnail: /static/assets/thumbnails/2020/lec2.png
date: 2020-01-14
ready: true
video:
  aspect: 56.25
  id: kgII-YWo3Zw
---

I den här föreläsningen går vi igenom grunderna i att använda bash som skriptspråk tillsammans med ett antal skalverktyg som täcker flera av de vanligaste uppgifterna du ständigt utför i kommandoraden.

# Skalskriptning

Hittills har vi sett hur man kör kommandon i skalet och kopplar ihop dem med rör.
I många scenarier vill du dock köra en serie kommandon och använda styrflöde som villkor eller loopar.

Skalskript är nästa steg i komplexitet.
De flesta skal har ett eget skriptspråk med variabler, styrflöde och egen syntax.
Det som skiljer skalskriptning från andra skriptspråk är att det är optimerat för skalrelaterade uppgifter.
Att bygga kommandokedjor, spara resultat i filer och läsa från standard input är därför grundfunktioner i skalskriptning, vilket ofta gör det enklare att använda än allmänna skriptspråk.
I det här avsnittet fokuserar vi på bash-skriptning eftersom det är vanligast.

För att tilldela variabler i bash använder du syntaxen `foo=bar` och läser värdet med `$foo`.
Observera att `foo = bar` inte fungerar, eftersom det tolkas som att programmet `foo` körs med argumenten `=` och `bar`.
Generellt gäller att blanktecken i skalskript orsakar argumentsplittring.
Detta beteende kan vara förvirrande i början, så var alltid uppmärksam på det.

Strängar i bash kan avgränsas med `'` och `"`, men de är inte likvärdiga.
Strängar med `'` är bokstavliga och expanderar inte variabler, medan strängar med `"` gör det.

```bash
foo=bar
echo "$foo"
# skriver ut bar
echo '$foo'
# skriver ut $foo
```

Som de flesta programmeringsspråk stöder bash styrflödestekniker som `if`, `case`, `while` och `for`.
På samma sätt har `bash` funktioner som tar argument och kan arbeta med dem.
Här är ett exempel på en funktion som skapar en katalog och går in i den med `cd`.


```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

Här är `$1` det första argumentet till skriptet/funktionen.
Till skillnad från andra skriptspråk använder bash en mängd specialvariabler för att referera till argument, felkoder och andra relevanta värden.
Nedan är en lista över några av dem.
En mer komplett lista finns [här](https://tldp.org/LDP/abs/html/special-chars.html).
- `$0` - Skriptets namn
- `$1` till `$9` - Argument till skriptet. `$1` är första argumentet och så vidare.
- `$@` - Alla argument
- `$#` - Antal argument
- `$?` - Returkod för föregående kommando
- `$$` - Process-ID (PID) för det aktuella skriptet
- `!!` - Hela senaste kommandot inklusive argument. Ett vanligt mönster är att köra ett kommando som misslyckas på grund av saknade rättigheter; då kan du snabbt köra om det med sudo genom att skriva `sudo !!`
- `$_` - Sista argumentet i senaste kommandot. I ett interaktivt skal kan du också snabbt få värdet genom att skriva `Esc` följt av `.` eller `Alt+.`

Kommandon returnerar ofta utdata via `STDOUT`, fel via `STDERR` och en returkod för att rapportera fel på ett skriptvänligt sätt.
Returkoden, eller exit-status, är hur skript/kommandon kommunicerar hur körningen gick.
Värdet 0 betyder vanligtvis att allt gick bra; allt annat än 0 betyder att ett fel uppstod.

Exit-koder kan användas för villkorad körning av kommandon med `&&` (och-operator) och `||` (eller-operator), som båda är [kortslutande](https://en.wikipedia.org/wiki/Short-circuit_evaluation) operatorer.
Kommandon kan också separeras på samma rad med semikolon `;`.
Programmet `true` returnerar alltid 0 och kommandot `false` returnerar alltid 1.
Låt oss se några exempel.

```bash
false || echo "Oj, fel"
# skriver ut: Oj, fel

true || echo "Skrivs inte ut"
# skriver inte ut något

true && echo "Allt gick bra"
# skriver ut: Allt gick bra

false && echo "Skrivs inte ut"
# skriver inte ut något

true ; echo "Det här körs alltid"
# Det här körs alltid

false ; echo "Det här körs alltid"
# Det här körs alltid
```

Ett annat vanligt mönster är att vilja få utdata från ett kommando till en variabel.
Det kan göras med _command substitution_ (kommandosubstitution).
Varje gång du skriver `$( CMD )` körs `CMD`, kommandots utdata hämtas och ersätter uttrycket på plats.
Om du till exempel skriver `for file in $(ls)` kommer skalet först att köra `ls` och sedan iterera över värdena.
En mindre känd liknande funktion är _process substitution_ (processsubstitution), där `<( CMD )` kör `CMD`, lägger utdata i en temporär fil och ersätter `<()` med filens namn.
Detta är användbart när kommandon förväntar sig värden via fil i stället för via STDIN.
Till exempel visar `diff <(ls foo) <(ls bar)` skillnader mellan filer i katalogerna `foo` och `bar`.


Eftersom det var mycket information på en gång tar vi ett exempel som visar några av funktionerna.
Det itererar över argumenten vi skickar in, kör `grep` efter strängen `foobar` och appenderar den till filen som en kommentar om den inte hittas.

```bash
#!/bin/bash

echo "Startar programmet $(date)" # Datumet ersätts här

echo "Kör programmet $0 med $# argument och PID $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # När mönstret inte hittas får grep exit-status 1
    # Vi omdirigerar STDOUT och STDERR till /dev/null eftersom vi inte bryr oss om utdata här
    if [[ $? -ne 0 ]]; then
        echo "Filen $file innehåller inte foobar, lägger till en rad"
        echo "# foobar" >> "$file"
    fi
done
```

I jämförelsen testade vi om `$?` inte var lika med 0.
Bash implementerar många jämförelser av detta slag; en detaljerad lista finns i manualsidan för [`test`](https://www.man7.org/linux/man-pages/man1/test.1.html).
När du gör jämförelser i bash, försök använda dubbla hakparenteser `[[ ]]` i stället för enkla `[ ]`.
Risken för misstag är mindre, även om det då inte blir portabelt till `sh`.
En mer detaljerad förklaring finns [här](https://mywiki.wooledge.org/BashFAQ/031).

När du startar skript vill du ofta skicka in liknande argument.
Bash har sätt att underlätta detta genom att expandera uttryck via filnamnsexpansion.
Dessa tekniker kallas ofta skalglobbing.
- Jokertecken - När du vill matcha med jokertecken kan du använda `?` och `*` för att matcha ett respektive valfritt antal tecken. Givet filerna `foo`, `foo1`, `foo2`, `foo10` och `bar` tar kommandot `rm foo?` bort `foo1` och `foo2`, medan `rm foo*` tar bort alla utom `bar`.
- Måsvingar `{}` - När du har en gemensam delsträng i en serie kommandon kan du använda måsvingar för att låta bash expandera detta automatiskt. Detta är mycket praktiskt när du flyttar eller konverterar filer.

```bash
convert image.{png,jpg}
# Expanderar till
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Expanderar till
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbingtekniker kan kombineras
mv *{.py,.sh} folder
# Flyttar alla *.py- och *.sh-filer


mkdir foo bar
# Detta skapar filerna foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Visa skillnader mellan filer i foo och bar
diff <(ls foo) <(ls bar)
# Skriver ut
# < x
# ---
# > y
```

<!-- Slutligen är rör `|` en kärnfunktion i skriptning. Rör kopplar utdata från ett program till indata för nästa program. Vi går igenom dem mer i detalj i föreläsningen om datahantering. -->

Att skriva `bash`-skript kan vara knepigt och ibland ointuitivt.
Det finns verktyg som [shellcheck](https://github.com/koalaman/shellcheck) som hjälper dig hitta fel i dina sh/bash-skript.

Observera att skript inte måste skrivas i bash för att kunna anropas från terminalen.
Här är till exempel ett enkelt Python-skript som skriver ut sina argument i omvänd ordning:

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

Kärnan vet att skriptet ska köras med en Python-tolk i stället för som ett skalkommando eftersom vi inkluderade en [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) längst upp i skriptet.
Det är god praxis att skriva shebang-rader med kommandot [`env`](https://www.man7.org/linux/man-pages/man1/env.1.html), som pekar på var kommandot finns i systemet och ökar portabiliteten i dina skript.
För att hitta platsen använder `env` miljövariabeln `PATH` som vi introducerade i första föreläsningen.
I detta exempel skulle shebang-raden se ut som `#!/usr/bin/env python`.

Några skillnader mellan skalfunktioner och skript att ha i åtanke är:
- Funktioner måste skrivas i samma språk som skalet, medan skript kan skrivas i vilket språk som helst. Därför är shebang viktig för skript.
- Funktioner laddas en gång när deras definition läses in. Skript laddas varje gång de körs. Detta gör funktioner något snabbare att ladda, men när du ändrar dem måste du ladda om definitionen.
- Funktioner körs i den aktuella skalmiljön medan skript körs i en egen process. Därför kan funktioner ändra miljövariabler, t.ex. byta aktuell katalog, medan skript inte kan det. Miljövariabler som exporterats med [`export`](https://www.man7.org/linux/man-pages/man1/export.1p.html) skickas med värde till skript.
- Som i alla programmeringsspråk är funktioner en kraftfull konstruktion för modularitet, kodåteranvändning och tydlighet i skalkod. Ofta innehåller skalskript egna funktionsdefinitioner.

# Skalverktyg

## Ta reda på hur kommandon används

Vid det här laget kanske du undrar hur man hittar flaggorna för kommandon i avsnittet om alias, som `ls -l`, `mv -i` och `mkdir -p`.
Mer generellt: givet ett kommando, hur tar du reda på vad det gör och vilka alternativ som finns?
Du kan alltid börja googla, men eftersom UNIX är äldre än StackOverflow finns inbyggda sätt att få den informationen.

Som vi såg i föreläsningen om skalet är förstahandsmetoden att köra kommandot med flaggan `-h` eller `--help`.
En mer detaljerad metod är kommandot `man`.
Kort för manual ger [`man`](https://www.man7.org/linux/man-pages/man1/man.1.html) en manualsida (manpage) för kommandot du anger.
Till exempel skriver `man rm` ut beteendet för kommandot `rm` tillsammans med flaggorna det accepterar, inklusive `-i`-flaggan vi visade tidigare.
Faktum är att det jag har länkat till för varje kommando hittills är onlineversionen av Linux-manpages.
Även externa kommandon du installerar kan få manpage-poster om utvecklaren har skrivit och inkluderat dem i installationsprocessen.
För interaktiva verktyg, som de som bygger på ncurses, går hjälp för kommandon ofta att nå inifrån programmet med kommandot `:help` eller genom att trycka `?`.

Ibland kan manpages vara väldigt detaljerade, vilket gör det svårt att snabbt se vilka flaggor/syntax som behövs i vanliga situationer.
[TLDR pages](https://tldr.sh/) är ett smidigt komplement som fokuserar på konkreta exempel så att du snabbt kan se vilka alternativ du ska använda.
Jag märker till exempel att jag går tillbaka till tldr-sidorna för [`tar`](https://tldr.inbrowser.app/pages/common/tar) och [`ffmpeg`](https://tldr.inbrowser.app/pages/common/ffmpeg) mycket oftare än till manpages.


## Hitta filer

En av de vanligaste repetitiva uppgifterna för programmerare är att hitta filer eller kataloger.
Alla UNIX-liknande system levereras med [`find`](https://www.man7.org/linux/man-pages/man1/find.1.html), ett utmärkt skalverktyg för att hitta filer.
`find` söker rekursivt efter filer som matchar vissa kriterier.
Några exempel:

```bash
# Hitta alla kataloger som heter src
find . -name src -type d
# Hitta alla Python-filer som har en mapp med namnet test i sökvägen
find . -path '*/test/*.py' -type f
# Hitta alla filer som ändrats det senaste dygnet
find . -mtime -1
# Hitta alla zip-filer med storlek mellan 500k och 10M
find . -size +500k -size -10M -name '*.tar.gz'
```
Utöver att lista filer kan find också utföra åtgärder på filer som matchar din sökning.
Den egenskapen är mycket hjälpsam för att förenkla uppgifter som annars kan bli monotona.
```bash
# Ta bort alla filer med filändelsen .tmp
find . -name '*.tmp' -exec rm {} \;

# Hitta alla PNG-filer och konvertera dem till JPG
find . -name '*.png' -exec magick {} {}.jpg \;
```

Trots att `find` finns överallt kan syntaxen ibland vara svår att komma ihåg.
För att bara hitta filer som matchar ett mönster `PATTERN` måste du till exempel köra `find -name '*PATTERN*'` (eller `-iname` om matchningen ska vara skiftlägesokänslig).
Du kan bygga alias för sådana scenarier, men en del av skalfilosofin är att det är bra att utforska alternativ.
Kom ihåg att en av skalets bästa egenskaper är att du bara anropar program, så du kan hitta (eller skriva själv) ersättare för vissa verktyg.
Till exempel är [`fd`](https://github.com/sharkdp/fd) ett enkelt, snabbt och användarvänligt alternativ till `find`.
Det har bra standardval som färgad utdata, regexmatchning som standard och Unicode-stöd.
Det har också, enligt min mening, en mer intuitiv syntax.
Syntaxen för att hitta mönstret `PATTERN` är till exempel bara `fd PATTERN`.

Många håller med om att `find` och `fd` är bra, men vissa undrar om det är effektivare att söka filer varje gång jämfört med att bygga ett index eller en databas för snabb sökning.
Det är just vad [`locate`](https://www.man7.org/linux/man-pages/man1/locate.1.html) är till för.
`locate` använder en databas som uppdateras med [`updatedb`](https://www.man7.org/linux/man-pages/man1/updatedb.1.html).
I de flesta system uppdateras `updatedb` dagligen via [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html).
En avvägning mellan verktygen är därför hastighet kontra färskhet.
Dessutom kan `find` och liknande verktyg söka på attribut som filstorlek, ändringstid eller filrättigheter, medan `locate` bara använder filnamn.
En mer djupgående jämförelse finns [här](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other).

## Hitta kod

Att hitta filer via namn är användbart, men ofta vill du söka baserat på *innehåll*.
Ett vanligt scenario är att hitta alla filer som innehåller ett visst mönster, inklusive var i filerna mönstret förekommer.
För detta erbjuder de flesta UNIX-liknande system [`grep`](https://www.man7.org/linux/man-pages/man1/grep.1.html), ett generellt verktyg för att matcha mönster i indata.
`grep` är ett otroligt värdefullt skalverktyg som vi går in djupare på i föreläsningen om datahantering.

För nu räcker det att veta att `grep` har många flaggor som gör det mycket mångsidigt.
Några jag ofta använder är `-C` för **C**ontext runt matchande rad och `-v` för att in**v**ertera matchningen, dvs. skriva ut alla rader som **inte** matchar mönstret.
Till exempel skriver `grep -C 5` ut 5 rader före och efter matchningen.
När du snabbt vill söka genom många filer vill du använda `-R` eftersom det går **R**ekursivt in i kataloger och letar i filer efter matchsträngen.

Men `grep -R` kan förbättras på många sätt, som att ignorera `.git`-mappar, använda flera CPU-kärnor, &c.
Många alternativ till `grep` har utvecklats, bland annat [ack](https://github.com/beyondgrep/ack3), [ag](https://github.com/ggreer/the_silver_searcher) och [rg](https://github.com/BurntSushi/ripgrep).
Alla är utmärkta och erbjuder i stort sett samma funktionalitet.
Just nu håller jag mig till ripgrep (`rg`) tack vare dess hastighet och intuitiva användning.
Några exempel:
```bash
# Hitta alla Python-filer där jag använder biblioteket requests
rg -t py 'import requests'
# Hitta alla filer (inklusive dolda) utan shebang-rad
rg -u --files-without-match "^#\!"
# Hitta alla träffar på foo och skriv ut de följande 5 raderna
rg foo -A 5
# Skriv ut statistik för träffarna (antal matchande rader och filer)
rg --stats PATTERN
```

Observera att precis som med `find`/`fd` är det viktigaste att känna till att problemen snabbt kan lösas med något av verktygen; exakt vilket verktyg du använder är mindre viktigt.

## Hitta skalkommandon

Hittills har vi sett hur man hittar filer och kod, men när du lägger mer tid i skalet kan du vilja hitta särskilda kommandon du skrev tidigare.
Det första att känna till är att uppåtpilen ger dig senaste kommandot tillbaka, och om du fortsätter trycka går du gradvis bakåt i skalhistoriken.

Kommandot `history` låter dig komma åt skalhistoriken programmatiskt.
Det skriver ut historiken till standard output.
Om vi vill söka i den kan vi skicka utdata genom ett rör till `grep` och leta efter mönster.
`history | grep find` skriver ut kommandon som innehåller delsträngen "find".

I de flesta skal kan du använda `Ctrl+R` för bakåtsökning i historiken.
Efter att ha tryckt `Ctrl+R` kan du skriva en delsträng som ska matcha kommandon i historiken.
Om du fortsätter trycka cyklar du genom träffarna.
Detta kan också aktiveras med UP/DOWN-pilar i [zsh](https://github.com/zsh-users/zsh-history-substring-search).
Ett trevligt tillägg till `Ctrl+R` är bindningar med [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r).
`fzf` är en generell fuzzy finder som kan användas med många kommandon.
Här används den för fuzzy-matchning i historiken och presenterar resultat på ett smidigt och visuellt tilltalande sätt.

Ett annat historiktrick jag gillar mycket är **history-based autosuggestions**.
Funktionen introducerades först i skalet [fish](https://fishshell.com/) och autokompletterar dynamiskt aktuellt kommando med det senaste kommandot du skrivit som delar prefix.
Den kan aktiveras i [zsh](https://github.com/zsh-users/zsh-autosuggestions) och är ett riktigt bra livskvalitetstrick för skalet.

Du kan ändra historikbeteendet i skalet, till exempel förhindra att kommandon med inledande blanksteg sparas.
Det är praktiskt när du skriver kommandon med lösenord eller annan känslig information.
För att göra det, lägg till `HISTCONTROL=ignorespace` i `.bashrc` eller `setopt HIST_IGNORE_SPACE` i `.zshrc`.
Om du glömmer inledande blanksteg kan du alltid manuellt ta bort posten genom att redigera `.bash_history` eller `.zsh_history`.

## Katalognavigering

Hittills har vi antagit att du redan befinner dig där du behöver vara för att utföra dessa åtgärder.
Men hur navigerar man snabbt mellan kataloger?
Det finns många enkla sätt, som att skriva skalalias eller skapa symlänkar med [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html), men sanningen är att utvecklare redan har tagit fram ganska smarta och sofistikerade lösningar.

Som så ofta i den här kursen vill du optimera för det vanliga fallet.
Att hitta frekventa och/eller nyligen använda filer och kataloger går med verktyg som [`fasd`](https://github.com/clvv/fasd) och [`autojump`](https://github.com/wting/autojump).
Fasd rankar filer och kataloger efter [_frecency_](https://web.archive.org/web/20210421120120/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm), alltså både _frequency_ och _recency_.
Som standard lägger `fasd` till kommandot `z` som låter dig göra snabb `cd` med en delsträng av en _frecent_ katalog.
Om du ofta går till `/home/user/files/cool_project` kan du till exempel bara skriva `z cool` för att hoppa dit.
Med autojump kan samma katalogbyte göras med `j cool`.

Mer avancerade verktyg finns för att snabbt få en översikt av katalogstrukturer: [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) eller fullfjädrade filhanterare som [`nnn`](https://github.com/jarun/nnn) och [`ranger`](https://github.com/ranger/ranger).

<span id="exercises"></span>
# Övningar

1. Läs [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) och skriv ett `ls`-kommando som listar filer enligt följande

    - Inkluderar alla filer, även dolda filer
    - Storlekar visas i människoläsbart format (t.ex. 454M i stället för 454279954)
    - Filer sorteras efter hur nyligen de ändrats
    - Utdata är färgsatt

    En exempelutdata kan se ut så här

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. Skriv bash-funktionerna `marco` och `polo` som gör följande.
När du kör `marco` ska nuvarande arbetskatalog sparas på något sätt, och när du sedan kör `polo` ska `polo` göra `cd` tillbaka till katalogen där du körde `marco`, oavsett var du befinner dig.
För enklare felsökning kan du skriva koden i en fil `marco.sh` och (om)ladda definitionerna i skalet genom att köra `source marco.sh`.

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

1. Säg att du har ett kommando som sällan misslyckas.
För att felsöka det behöver du fånga utdata, men det kan ta tid innan du får en körning som faktiskt fallerar.
Skriv ett bash-skript som kör följande skript tills det misslyckas, fångar standard output och felström till filer och skriver ut allt i slutet.
Bonuspoäng om du också rapporterar hur många körningar det tog innan skriptet misslyckades.

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Något gick fel"
       >&2 echo "Felet var användning av magiska tal"
       exit 1
    fi

    echo "Allt gick enligt plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0
until [[ "$?" -ne 0 ]];
do
  count=$((count+1))
  ./random.sh &> out.txt
done

echo "hittade fel efter $count körningar"
cat out.txt
{% endcomment %}

1. Som vi tog upp i föreläsningen kan `find` med `-exec` vara mycket kraftfullt för att utföra operationer på filer vi söker efter.
Men vad händer om vi vill göra något med **alla** filer, till exempel skapa en zip-fil?
Som du sett hittills tar kommandon indata både via argument och STDIN.
När vi kopplar ihop kommandon med rör kopplar vi STDOUT till STDIN, men vissa kommandon som `tar` tar indata via argument.
För att överbrygga detta finns kommandot [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html), som kör ett kommando med STDIN som argument.
Till exempel tar `ls | xargs rm` bort filerna i aktuell katalog.

    Din uppgift är att skriva ett kommando som rekursivt hittar alla HTML-filer i mappen och gör en zip av dem.
    Observera att kommandot ska fungera även om filnamnen innehåller blanksteg (tips: titta på flaggan `-d` för `xargs`).
    {% comment %}
    find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf archive.tar.gz
    {% endcomment %}

    Om du använder macOS, notera att standardversionen av BSD `find` skiljer sig från den som ingår i [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands).
    Du kan använda `-print0` på `find` och flaggan `-0` på `xargs`.
    Som macOS-användare bör du känna till att kommandoradsverktygen som levereras med macOS kan skilja sig från GNU-motsvarigheterna; du kan installera GNU-versionerna om du vill genom att [använda brew](https://formulae.brew.sh/formula/coreutils).

1. (Avancerad) Skriv ett kommando eller skript som rekursivt hittar den senast ändrade filen i en katalog.
Mer allmänt, kan du lista alla filer efter hur nyligen de ändrats?
