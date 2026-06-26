# BoarShed — Skládání rozvrhu

Logická hra pro kolegy, kde se „skládá rozvrh". In-joke připravený na firemní/školní
akci přezdívanou **„prase"** (podle opékaného prasete a guláše). Hra může být
teoreticky i neřešitelně těžká — to je součást vtipu.

Název **BoarShed** je dvojitá slovní hříčka: *Boar* (kanec → prase) + *Shed*, které zní
líp než *Sched*(uler).

## Co to je

Hráč přetahuje barevné kartičky (vyučované hodiny) do rozvrhu lichého a sudého týdne
(Po–Pá) a snaží se splnit současně všechny požadavky. Tvrdé požadavky (konflikty
učitel/učebna/třída, délka bloku) hlídá engine; měkké herní cíle (volný den učitele,
zkrácený úvazek, předmět dopoledne…) se počítají do skóre.

Stav: **funkční kostra (MVP)** — viz [docs/handoff.md](docs/handoff.md) pro úplný kontext.

## Funkce

- Tři sloupce: **Kartičky / Rozvrh / Požadavky**, responzivní (pod 1100 px nad sebe).
- Umístění kartičky **kliknutím** i **drag-and-drop**; bloky 1/2/3 h.
- **Pohledy Třídy / Učitelé / Učebny** — hraje se v pohledu po třídách, učitelský
  a učebnový pohled jsou jen pro čtení (kontrola konfliktů).
- **Globální detekce konfliktů**, zvýraznění + tooltip s důvodem. Překrývající se
  hodiny stojí vedle sebe (žádná se neschová).
- Živé vyhodnocení cílů, skóre v %, progress bar, výherní stav. Bohatá sada typů cílů
  (denní okno, konkrétní den/týden, volný den, jen jeden týden, bez oken, max hodin
  denně, polední pauza, přechod mezi budovami).
- Tlačítko **Náhodně** — randomizovaný backtracking, doplní jen neumístěné karty bez
  tvrdých konfliktů; **Smazat** s potvrzením; **Řešitelnost?** zkontroluje, zda je
  level vůbec vyhratelný a jak je těžký.
- **Ukládání** přes serializační řetězec (Base64 JSON) + pojmenované sloty; funguje
  i bez účtu a po nasazení.
- Světlý/tmavý motiv (CSS proměnné), volba se pamatuje.
- Dvě demo úrovně (L1 rozjezd, L2 dva týdny / dvě budovy s přechody mezi budovami).

## Spuštění

Žádná instalace, žádný build, žádné závislosti.

```
Otevři index.html v prohlížeči.
```

## Nasazení (Render — Static Site)

Soubor `index.html` je rovnou nasaditelný jako **Static Site** na Renderu (Free):

1. Repo nahraj na GitHub / GitLab.
2. Na Renderu vyber **Static Site**, Free instance.
3. Build command: *(žádný)*. Publish directory: *kořen repa* (`.`).

Static Site je CDN bez časového limitu — **neusíná** (na rozdíl od Free Web Service).
Protože je to čistý frontend bez DB, je to přesně tenhle případ.

## Struktura projektu

```
BoarShed/
├── index.html        # celá hra (HTML + CSS + vanilla JS, jeden soubor)
├── README.md         # tenhle soubor
├── CLAUDE.md         # instrukce pro Claude Code
└── docs/
    └── handoff.md    # předávací dokument: zadání, rozhodnutí, architektura, TODO
```

## Co dál

Otevřené body a nápady (seznamy tříd/učeben/učitelů, návrh levelů, iron-man režim,
pohledy za učitele/učebnu, generátor levelů…) jsou v
[docs/handoff.md → sekce 5](docs/handoff.md).
