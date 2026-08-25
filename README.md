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

Stav: **hotová a nasazená hra** — kampaň 7 levelů s měřeným balancem, editor, sdílení,
mobilní i desktopové rozhraní. Hraje se na **https://senkyr.github.io/BoarShed/**.
Úplný kontext: [docs/handoff.md](docs/handoff.md).

## Funkce

- **Požadavky** jako sbalovací pruh nahoře (na telefonu rovnou rozevřený, na desktopu sklapnutý),
  pod ním **Kartičky | Rozvrh**; responzivní (pod 1100 px nad sebe; na telefonu paleta jen
  aktivní třídy, nic nescrolluje do stran).
- **Rozvrh „na šířku" na desktopu** (dny v řádcích, hodiny ve sloupcích); na úzkém
  displeji se překlopí na hodiny pod sebou.
- Umístění kartičky **kliknutím** i **tažením** — jednotný pointer-drag funguje myší
  i na dotyku (na mobilu podržet ~200 ms a táhnout); bloky 1/2/3 h.
- **Zpět / Znovu** (↶/↷, Ctrl+Z / Ctrl+Y); Esc nebo zrušené tažení vrací zvednutou
  kartičku na původní místo.
- **Pohledy Třídy / Učitelé / Učebny** — hraje se ve všech třech: kartičku položíš jen do
  rozvrhu, který je vidět a do kterého patří; zásobník ukazuje kartičky zobrazeného rozvrhu
  plně, ostatní tlumeně.
- **Globální detekce konfliktů**, zvýraznění + tooltip s důvodem. Překrývající se
  hodiny stojí vedle sebe (žádná se neschová).
- Živé vyhodnocení cílů, skóre v %, progress bar, výherní stav. Bohatá sada typů cílů
  (denní okno, konkrétní den/týden, volný den, jen jeden týden, bez oken, max hodin
  denně, polední pauza, přechod mezi budovami).
- Tlačítko **Náhodně** — randomizovaný backtracking, doplní jen neumístěné karty bez
  tvrdých konfliktů (volitelně zkusí splnit i herní cíle); když se něco nevejde, hlásí
  který tvrdý požadavek to zablokoval. **Smazat** s potvrzením; **Jde to vůbec vyhrát?** ukáže
  v modálu, jak je level těžký (% výher a splnění jednotlivých cílů).
- **Nařízená pauza** — každá třída má šrafovanou kartičku **Oběd**, kterou engine pustí
  jen do 4.–5. hodiny (tvrdé pravidlo, ne cíl). Nemá učitele ani učebnu, takže celá škola
  smí obědvat naráz, ale třídě zabírá slot.
- **Iron-man**: level může stavět na pozici z předchozího levelu — přenesené kartičky
  jsou zamčené (🔒). V kampani staví level 4 (Iron-man) na tvém řešení levelu 3 — přenese se jen
  dohraný a vyhraný rozvrh; rozehraný, s konfliktem nebo takový, který by zdejší
  požadavky znemožnil, nahradí vzorové řešení (s hláškou).
- **Editor levelů** — vlastní levely jako JSON (šablona z existujícího levelu,
  validace, test řešitelnosti), ukládají se lokálně a v přepínači mají `*`.
  Vlastní level jde **poslat odkazem** (`#l=…`) — příjemci se uloží mezi vlastní
  a rovnou se načte.
- **Fér ✓** — odznak u levelu je jen za vlastnoruční výhru; rozvrh poskládaný
  řešičem nebo načtený z cizího odkazu ukáže výherní banner, ale ✓ nedá.
- **ⓘ O hře** — jak hra vznikla (AI vývoj, 2026), čísla balancu a tip pro zvídavé.
- **Ukládání** přes serializační řetězec (Base64 JSON) + pojmenované sloty; funguje
  i bez účtu a po nasazení. **Sdílení pozice odkazem** (`#p=…`). Dokončené levely
  mají ✓ v přepínači.
- Světlý/tmavý motiv (CSS proměnné), respektuje systémovou preferenci, volba se pamatuje.
- **Kampaň 7 levelů** s měřenou rostoucí obtížností (výhra v ~10 % náhodných platných
  rozvrhů v rozjezdu, v setinách až desetinách procenta ke konci):
  rozjezd → první ročníky → druhé → třetí → iron-man dostavba →
  maturitní ročníky → finále **„Celá škola"** se všemi 16 třídami. Svět je inspirovaný skutečností:
  4 obory (Informatika a management, elektronické počítače, slaboproudá
  elektrotechnika, strojírenství), ročníky 1–4, tři budovy se skutečnými učebnami
  (Š101 → T, H59 → E/F/G, H618 → B/C) plus půjčené tělocvičny ZŠ; zástupné zůstávají
  jen předměty a křestní jména — později lze 1:1 vyměnit za podvýběr skutečných.

## Kde se hraje

**https://senkyr.github.io/BoarShed/** — hostováno přes GitHub Pages přímo z tohoto repa
(větev `main`, žádný build). Push do `main` = automatický redeploy během ~minuty.

## Spuštění lokálně

Žádná instalace, žádný build, žádné závislosti.

```
Otevři index.html v prohlížeči.
```

(Alternativa hostingu: Render Static Site — stejný soubor, build žádný, publish root.
Není potřeba, GitHub Pages stačí.)

## Struktura projektu

```
BoarShed/
├── index.html        # celá hra (HTML + CSS + vanilla JS, jeden soubor)
├── README.md         # tenhle soubor
├── CLAUDE.md         # instrukce pro Claude Code (architektura, svět kampaně, TODO)
└── docs/
    ├── handoff.md    # předávací dokument: zadání, rozhodnutí, architektura, TODO
    └── review.md     # historické review zralosti (26. 6. 2026)
```

## Co dál

1. Volitelně **výměna zástupných jmen za podvýběr skutečných** — 12 jmen učitelů,
   19 názvů předmětů, 12 kódů učeben (přesný rozsah a role: CLAUDE.md → TODO).
2. Ruční test dotyku na reálném mobilu (na nasazené URL).

Ostatní otevřené nápady drží [docs/handoff.md → sekce 5](docs/handoff.md).
