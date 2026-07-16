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
- **Rozvrh „na šířku" na desktopu** (dny v řádcích, hodiny ve sloupcích); na úzkém
  displeji se překlopí na hodiny pod sebou.
- Umístění kartičky **kliknutím** i **tažením** — jednotný pointer-drag funguje myší
  i na dotyku (na mobilu podržet ~200 ms a táhnout); bloky 1/2/3 h.
- **Zpět / Znovu** (↶/↷, Ctrl+Z / Ctrl+Y); Esc nebo zrušené tažení vrací zvednutou
  kartičku na původní místo.
- **Pohledy Třídy / Učitelé / Učebny** — hraje se v pohledu po třídách, učitelský
  a učebnový pohled jsou jen pro čtení (kontrola konfliktů).
- **Globální detekce konfliktů**, zvýraznění + tooltip s důvodem. Překrývající se
  hodiny stojí vedle sebe (žádná se neschová).
- Živé vyhodnocení cílů, skóre v %, progress bar, výherní stav. Bohatá sada typů cílů
  (denní okno, konkrétní den/týden, volný den, jen jeden týden, bez oken, max hodin
  denně, polední pauza, přechod mezi budovami).
- Tlačítko **Náhodně** — randomizovaný backtracking, doplní jen neumístěné karty bez
  tvrdých konfliktů (volitelně zkusí splnit i herní cíle); když se něco nevejde, hlásí
  který tvrdý požadavek to zablokoval. **Smazat** s potvrzením; **Řešitelnost?** ukáže
  v modálu, jak je level těžký (% výher a splnění jednotlivých cílů).
- **Iron-man**: level může stavět na pozici z předchozího levelu — přenesené kartičky
  jsou zamčené (🔒). Demo level 3 staví na tvém řešení levelu 2.
- **Editor levelů** — vlastní levely jako JSON (šablona z existujícího levelu,
  validace, test řešitelnosti), ukládají se lokálně a v přepínači mají `*`.
- **Ukládání** přes serializační řetězec (Base64 JSON) + pojmenované sloty; funguje
  i bez účtu a po nasazení. **Sdílení pozice odkazem** (`#p=…`). Dokončené levely
  mají ✓ v přepínači.
- Světlý/tmavý motiv (CSS proměnné), respektuje systémovou preferenci, volba se pamatuje.
- **Kampaň 6 levelů** s měřenou rostoucí obtížností (výhra v ~10 % → ~0,4 % náhodných
  platných rozvrhů): rozjezd → dvě třídy → dva týdny a dvě budovy → tři budovy →
  iron-man dostavba → finále **„Prase"**. Svět je „naslepo" inspirovaný skutečností:
  4 obory (informatika s managementem, elektronické počítače, slaboproudá
  elektrotechnika, strojírenství), ročníky 1–4, tři budovy (Š101, H59, H618),
  zástupné předměty a křestní jména — později lze 1:1 vyměnit za podvýběr skutečných.

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
