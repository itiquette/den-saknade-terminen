---
layout: lecture
title: "Programintrospektion"
presenter: Anish
date: 2019-01-29
order: 1
video:
  aspect: 62.5
  id: 74MhV-7hYzg
---

# Felsökning (debugging)

När printf-felsökning inte räcker: använd en felsökare.

Felsökare låter dig interagera med körningen av ett program, så att du kan göra saker som:

- stoppa programkörning när den når en viss rad
- stega programmet rad för rad
- inspektera variabelvärden
- många fler avancerade funktioner

## GDB/LLDB

[GDB](https://www.gnu.org/software/gdb/) och [LLDB](https://lldb.llvm.org/).
Stödjer många C-liknande språk.

Låt oss titta på [example.c]({{ '/2019/files/example.c' | relative_url }}).
Kompilera med debug-flaggor: `gcc -g -o example example.c`.

Öppna GDB:

`gdb example`

Några kommandon:

- `run`
- `b {name of function}` - sätt en brytpunkt
- `b {file}:{line}` - sätt en brytpunkt
- `c` - fortsätt
- `step` / `next` / `finish` - stega in / stega över / stega ut
- `p {variable}` - skriv ut variabelvärde
- `watch {expression}` - sätt en bevakningspunkt som utlöses när uttryckets värde ändras
- `rwatch {expression}` - sätt en bevakningspunkt som utlöses när värdet läses
- `layout`

## PDB

[PDB](https://docs.python.org/3/library/pdb.html) är Pythons felsökare.

Infoga `import pdb; pdb.set_trace()` där du vill hoppa in i PDB.
Det är i praktiken en hybrid av felsökare (som GDB) och Python-skal.

## Utvecklarverktyg i webbläsaren

Ytterligare ett exempel på en felsökare, denna gång med grafiskt gränssnitt.

# strace

Observera systemanrop som ett program gör: `strace {program}`.

# Profilering

Typer av profilering: CPU, minne, osv.

Enklaste profileraren: `time`.

## Go

Kör testkod med CPU-profilerare: `go test -cpuprofile=cpu.out`

Analysera profil: `go tool pprof -web cpu.out`

Kör testkod med minnesprofilerare: `go test -memprofile=mem.out`

Analysera profil: `go tool pprof -web mem.out`

## Perf

Grundläggande prestandastatistik: `perf stat {command}`

Kör ett program med profileraren: `perf record {command}`

Analysera profil: `perf report`
