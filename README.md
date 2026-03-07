# Den saknade terminen i din datavetenskapsutbildning

[![Build Status](https://github.com/itiquette/den-saknade-terminen/actions/workflows/build.yml/badge.svg)](https://github.com/itiquette/den-saknade-terminen/actions/workflows/build.yml) [![Links Status](https://github.com/itiquette/den-saknade-terminen/actions/workflows/links.yml/badge.svg)](https://github.com/itiquette/den-saknade-terminen/actions/workflows/links.yml)

Webbplats för [Den saknade terminen i din datavetenskapsutbildning](https://itiquette.github.io/den-saknade-terminen/) - den svenska översättningen av MIT-kursen [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)

Bidrag är varmt välkomna! Om du har ändringar eller nytt innehåll att lägga till,
öppna gärna ett ärende eller skicka en ändringsförfrågan (PR).

## Utveckling

För att bygga och visa webbplatsen lokalt, kör:

```bash
bundle exec jekyll serve -w
```

Om du föredrar att utveckla webbplatsen i en container (t.ex. för att
slippa installera Ruby och beroenden på värdmaskinen), kör:

```bash
docker compose up --build
```

Därefter går du till <http://localhost:4000> i webbläsaren på värdmaskinen för
att visa webbplatsen. Jekyll bygger om webbplatsen automatiskt när du ändrar filer.

## Licens

Allt innehåll i kursen, inklusive webbplatsens källkod, föreläsningsanteckningar, övningar och föreläsningsvideor, är licensierat under Attribution-NonCommercial-ShareAlike 4.0 International [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Se [här](https://missing.csail.mit.edu/license) för mer information om bidrag eller översättningar.
