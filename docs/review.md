# Review: zralost dosavadní implementace

Hodnocení stavu hry **BoarShed / Skládání rozvrhu** k 26. 6. 2026. Posuzuju
funkční i obsahovou zralost na základě reálného spuštění a proklikání hry
v prohlížeči + testování herního enginu zevnitř (JS v kontextu stránky).

## Verdikt v jedné větě

**Technicky solidní MVP kostra, která funguje bez chyb v konzoli — ale obsahově je
to pořád demo:** jen dvě cvičné úrovně s výplňovými daty, jeden nepříjemný
vykreslovací bug a několik slíbených systémů (iron-man, editor, reálná data)
zatím chybí. Na ostrou „prasečí" akci to ještě není, na ukázku/prototyp ano.

Orientační zralost:

| Oblast | Zralost | Komentář |
|---|---|---|
| Engine & architektura | 🟢 ~80 % | Čisté, funkční, bez chyb, dobře rozšiřitelné. |
| Herní smyčka & UX | 🟡 ~65 % | Funguje, ale 1 viditelný bug + drobné vizuální nedostatky. |
| Ukládání / nasaditelnost | 🟢 ~85 % | Serializace + adaptér úložiště fungují, deploy-ready. |
| Obsah (levely, data, balanc) | 🔴 ~30 % | 2 demo levely, placeholder data, L2 sporný balanc. |
| Pokrytí původního zadání | 🟡 ~50 % | Jádro splněno; velká část „škály požadavků" a režimů chybí. |

## Jak jsem to testoval

- Spuštěno lokálně přes statický server (`python -m http.server`), reálně proklikáno
  v prohlížeči (výběr kartičky → umístění klikem, konflikty, řešič, motiv, ukládání).
- Engine ověřen i zevnitř: nasamploval jsem **500 náhodných platných rozvrhů** pro
  každý level (respektujících tvrdé požadavky) a změřil, jak často padne výhra a jak
  často je splněn který cíl — to ukazuje řešitelnost i obtížnost.
- Řešitelnost L2 jsem navíc potvrdil **ručně zkonstruovaným řešením**, které ověřil
  přímo herní engine (0 konfliktů, 5/5 cílů).

## Funkční stav — co funguje

Ověřeno reálně, vše bez jediné chyby/varování v konzoli:

- ✅ Načtení obou levelů, data odpovídají handoffu.
- ✅ **Výběr kartičky klikem** → přepnutí na záložku třídy → zvýraznění platných
  buněk (`drop`) → **umístění klikem**. Status řádek hlásí správně.
- ✅ **Detekce konfliktů** (učitel / učebna / třída ve stejné buňce) je správná;
  texty důvodů jsou srozumitelné, např. *„Petr učí zároveň 1.A"*, *„učebna U1 je
  obsazená (1.A)"*, *„1.A má dvě hodiny naráz"*.
- ✅ **Vyhodnocení cílů a výherní stav.** Po načtení kompletního řešení panel ukazuje
  *100 % splněno 5/5*, plný progress bar a banner *„Hotovo — rozvrh sedí… 🎉"*.
- ✅ **Řešič „Náhodně"** přes tlačítko + potvrzovací modál: doplní všech 6 kartiček
  L1 bez tvrdých konfliktů, modál se zavře, status sedí. (Měkké cíle ignoruje — dle
  návrhu; po doplnění byly splněné 2/5, zbytek je na hráči.)
- ✅ **Ukládání/načítání** serializačním řetězcem: `export → import` round-trip projde
  (Base64 JSON), neplatné karty se při importu odfiltrují.
- ✅ **Světlý/tmavý motiv** (přepínač funguje; ověřeno i přes computed styly —
  tmavé pozadí `#15161a`).
- ✅ **Responzivita**: pod 1100 px se tři sloupce skládají pod sebe.

> Drag-and-drop jsem programově nesimuloval (HTML5 DnD se přes testovací nástroj
> špatně vyvolává), ale handlery jsou navázané a klikací cesta — plnohodnotná
> alternativa dle návrhu — funguje. DnD doporučuju ručně přeťukat na desktopu.

## Nálezy / bugy

### 🟠 1. Překrývající se kartičky stejné třídy se navzájem skryjí (a jedna zmizí)

**Nejdůležitější nález.** Když dvě kartičky **téže třídy** začínají ve **stejné
buňce**, `renderSchedule` přepíše „cover" mapu (vyhrává poslední) a jednu kartičku
**vůbec nevykreslí**. Ta pak:

- není v paletě (je „umístěná"),
- není vidět v rozvrhu,
- jde získat zpět jen přes **Smazat rozvrh** (nebo reload / načtení pozice).

Navíc panel napravo počítá konflikt u **obou** kartiček, takže *„Konflikty: 2
kartiček"* nesedí s tím, co je na ploše vidět (jen 1 červená). Ověřeno: karta
*Programování* položená přes *Algoritmy* (Po, 1. h) z plochy zmizela, zůstala jen
*Algoritmy*.

*Dopad:* hráč může „ztratit" kartičku a být zmatený z nesedícího počtu konfliktů.
*Návrh fixu:* při překryvu vykreslit obě kartičky vedle sebe / s offsetem v rámci
buňky, nebo umístění do obsazené buňky stejné třídy rovnou blokovat (jako se blokuje
přesah bloku mimo den).

### 🟡 2. Dlouhé názvy předmětů přetékají z položené kartičky

*Programování* se v buňce zobrazí jako *„Programová…"* (oříznuté). Kosmetika, ale
bije do očí. Stačí `text-overflow:ellipsis` + menší font nebo zalamování.

### 🟢 3. Drobnost: po doplnění řešičem zůstává status z předchozí akce

Po `loadLevel` status řádek občas nese starší hlášku („Umístěno…") dokud hráč něco
neudělá. Bezvýznamné.

## Obsahová zralost (levely, data, balanc)

Tady je hra nejslabší — a je to očekávané, handoff to sám přiznává (data jsou
„výplň"). Konkrétně:

### Jen 2 demo levely, placeholder data

Třídy (1.A, 2.B), učebny (U1, U2, L1, D1), učitelé (Petr, Eva, Jan, Lucie) i přiřazení
předmětů jsou ukázková. Pro reálnou akci chybí seznamy a návrh levelů (viz
[handoff §5](handoff.md)).

### Měřená řešitelnost a obtížnost

Z 500 náhodných platných rozvrhů na level (podíl = jak často náhodný rozvrh cíl splní):

**L1 „Rozjezd" — vyladěný, fér rozehřívačka.** Výhra v **2,8 %** náhodných rozvrhů
(14/500) → řešitelné, ale chce přemýšlet. Úzká hrdla:

| Cíl | Splněno náhodně |
|---|---|
| Všechny hodiny umístěné | 100 % |
| Jan má volný den | 100 % |
| Programování (blok) do 4. h | 60 % |
| 1.A bez oken | 23 % |
| Sítě v pondělí | 24 % |

**L2 „Dva týdny, dvě budovy" — řešitelné, ale sporně vyladěné.** Ve 500 náhodných
rozvrzích **0 výher**. Řešení **existuje** (zkonstruoval jsem ho a engine ho potvrdil),
ale je extrémně vzácné. Bottleneck je *Elektronika v úterý* (jen 4 %) v kombinaci
s *Eva jen lichý týden* (25 %):

| Cíl | Splněno náhodně |
|---|---|
| Všechny hodiny umístěné | 100 % |
| Petr má volný den | 100 % |
| CAD pro 1.A do 3. h | 50 % |
| Eva jen lichý týden | 25 % |
| Elektronika v úterý | **4 %** |

Ověřené řešení L2 (lze vložit přes *Uložit/Načíst → Načíst z řetězce*):

```
eyJ2IjoxLCJsZXZlbCI6ImwyIiwicGwiOnsiYSI6eyJ3IjowLCJkIjozLCJwIjoxfSwiYiI6eyJ3IjowLCJkIjowLCJwIjoxfSwiYyI6eyJ3IjowLCJkIjoxLCJwIjoxfSwiZCI6eyJ3IjowLCJkIjoyLCJwIjoxfSwiZSI6eyJ3IjowLCJkIjowLCJwIjo0fSwiZiI6eyJ3IjowLCJkIjowLCJwIjoyfSwiZyI6eyJ3IjowLCJkIjoxLCJwIjozfSwiaCI6eyJ3IjowLCJkIjoyLCJwIjoyfX19
```

### „Dva týdny" jsou v L2 jen kulisa

Žádný cíl nevyžaduje rozprostřít hodiny přes oba týdny — moje ověřené výherní řešení
se vejde **celé do lichého týdne** (sudý zůstane prázdný, viz screenshot při testu).
Druhý týden tak přidává prostor k hledání, ale ne reálnou výzvu. Pokud má být „dva
týdny" feature, chce to cíl, který oba týdny vynutí (např. konkrétní předmět v sudém
týdnu, nebo lichý/sudý mix u jedné třídy).

### Zapečené, ale nevyužité

- Typ cíle `teacher_transition_gap` (přechod mezi budovami → volná hodina) je
  implementovaný, ale **žádný level ho nepoužívá**.
- Atribut `building` kartičky se reálně nikde nevyhodnocuje (uplatnil by se právě
  v `teacher_transition_gap`), takže je zatím **jen kosmetický**.

## Pokrytí původního zadání

Splněno: tři sloupce, lichý/sudý týden, bloky 1/2/3 h, konflikty se zvýrazní
(neblokuje se), skóre v %, řešič na tvrdé požadavky, ukládání bez účtu, deploy-ready.

Chybí z toho, co zadání zmiňuje:
- **Iron-man režim** (levely na sebe; model je na to prý připravený, ale není postaven).
- **Pohledy „za učitele" / „za učebnu"** (zatím jen za třídu).
- **Širší škála požadavků**: vazba předmětu na budovu, max hodin denně, polední pauza,
  preferované/zakázané hodiny učitele, povinné konkrétní časy.
- **Rostoucí doménová obtížnost** (víc budov/oborů postupně) — máme jen 2 levely.
- **Kontrola řešitelnosti levelu** — chybí, přitom L2 ukazuje, proč je potřeba.
- **Editor/generátor levelů** — levely se pořád píšou ručně v poli `LEVELS`.

## Doporučené pořadí prací

1. **Opravit bug #1** (mizející překrytá kartička) — kazí základní herní pocit.
2. **Doladit balanc L2** — buď zmírnit *Elektronika v úterý*, nebo přidat cíl, který
   smysluplně využije sudý týden; cílit na ~1–5 % šanci u náhodného rozvrhu jako L1.
3. **Dodat reálná data** (třídy/učebny+budovy/učitelé/předměty) — bez toho zůstává hra
   jen ukázkou.
4. **Návrh 4–6 levelů** s rostoucí obtížností + označení tvrdé vs. herní.
5. Až pak nadstavby: pohledy za učitele/učebnu, další typy cílů, kontrola řešitelnosti,
   editor levelů, iron-man.
6. Kosmetika: ořez dlouhých názvů (#2), ruční otestování drag-and-drop na desktopu
   i dotyku.

## Závěr

Kostra je zralá a dobře napsaná — engine, ukládání i nasaditelnost jsou na úrovni,
kterou handoff slibuje, a nenašel jsem žádný pád ani chybu v konzoli. Co chybí, je
**obsah a doladění**: reálná data, víc a líp vyvážených levelů, využití už hotových
(ale spících) mechanik a oprava jednoho vykreslovacího bugu. Jako prototyp k ukázání
„jak to bude vypadat" je to hotové; jako hratelná atrakce na akci ještě ne.
