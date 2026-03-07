---
layout: lecture
title: "Anpassa operativsystemet"
presenter: Anish
date: 2019-01-29
order: 3
video:
  aspect: 62.5
  id: epSRVqQzeDo
special: true
---

Det finns mycket du kan göra för att anpassa operativsystemet utöver det
som finns i inställningsmenyerna.

# Tangentomappning

Ditt tangentbord har troligen tangenter du nästan aldrig använder.
I stället för onödiga tangenter kan du mappa om dem till något användbart.

## Mappa om till andra tangenter

Det enklaste är att mappa tangenter till andra tangenter.
Om du till exempel inte använder Caps Lock särskilt mycket kan du mappa om den till något nyttigare.
Om du använder Vim kan du till exempel vilja mappa Caps Lock till Escape.

I macOS kan du göra vissa ommappningar via tangentbordsinställningarna i Systeminställningar.
För mer avancerade mappningar behöver du särskild programvara.

## Mappa om till valfria kommandon

Du behöver inte begränsa dig till att mappa tangenter till andra tangenter.
Det finns verktyg som låter dig mappa tangenter (eller tangentkombinationer) till valfria kommandon.
Du kan till exempel låta command-shift-t öppna ett nytt terminalfönster.

# Anpassa dolda OS-inställningar

## macOS

macOS exponerar många användbara inställningar via kommandot `defaults`.
Till exempel kan du göra Dock-ikoner för dolda program genomskinliga:

```shell
defaults write com.apple.dock showhidden -bool true
```

Det finns ingen enda komplett lista över alla möjliga inställningar,
men du hittar listor med specifika anpassningar på nätet, till exempel Mathias Bynens
[.macos](https://github.com/mathiasbynens/dotfiles/blob/master/.macos).

# Fönsterhantering

## Tiling-baserad fönsterhantering

[Tiling window management](https://en.wikipedia.org/wiki/Tiling_window_manager)
är ett sätt att hantera fönster där du organiserar fönster i ramar som inte överlappar.
Om du använder ett Unix-baserat operativsystem kan du installera en tiling-fönsterhanterare.
Om du använder något som Windows eller macOS kan du installera program som efterliknar beteendet.

## Skärmhantering

Du kan sätta upp tangentbordsgenvägar som hjälper dig att hantera fönster över flera skärmar.

## Layouter

Om du ofta placerar fönster på ett visst sätt på skärmen,
kan du skripta layouten i stället för att "utföra" den manuellt varje gång.
Då blir det enkelt att återskapa layouten.

# Resurser

- [Hammerspoon](https://www.hammerspoon.org/) - skrivbordsautomation för macOS
- [Rectangle](https://rectangleapp.com/) - fönsterhanterare för macOS
- [Karabiner](https://karabiner-elements.pqrs.org/) - avancerad tangentomappning för macOS
- [r/unixporn](https://www.reddit.com/r/unixporn/) - skärmbilder och
dokumentation av andras snygga konfigurationer

# Övningar

1. Ta reda på hur du mappar om Caps Lock till något du använder oftare
   (som Escape, Ctrl eller Backspace).

1. Skapa en global anpassad tangentbordsgenväg för att öppna ett nytt terminalfönster eller ett nytt webbläsarfönster.

{% comment %}

TODO

- Bitbar / Polybar
- Clipboard Manager (stack/searchable history)

{% endcomment %}
