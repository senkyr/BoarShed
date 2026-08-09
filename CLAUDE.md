# CLAUDE.md — BoarShed

Instrukce pro Claude Code při práci na tomhle repu.

## O projektu

Logická hra „Skládání rozvrhu" pro kolegy (in-joke na akci „prase"). Celá hra je
v **jediném souboru `index.html`** — čistý HTML + CSS + vanilla JS. Úplný kontext
(původní zadání, učiněná rozhodnutí a jejich důvody, TODO) je v
[docs/handoff.md](docs/handoff.md) — **přečti si ho před většími zásahy.**

## Tvrdé zásady (neporušovat bez explicitního pokynu)

- **Žádné závislosti, žádný build krok.** Vanilla JS, žádný npm, žádný bundler,
  žádný framework, žádné CDN. Vše zůstává v `index.html`.
- **Jeden soubor.** Hra je záměrně self-contained — renderuje se jako artefakt
  a zároveň je to deploy-ready `index.html` pro Render Static Site. Nerozbíjej do
  modulů, pokud o to Jakub výslovně nepožádá.
- **Jazyk UI a textů je čeština.** Anglické technické termíny ponech tam, kde jsou
  zavedené (drag-and-drop, backtracking…).
- **Konflikty se nezablokovávají, jen zvýrazní.** Hráč smí konflikt vytvořit, vidí
  ho červeně a opraví; výhra vyžaduje nula konfliktů. (Přesah bloku mimo den se ale
  blokuje.) Tohle je záměrné puzzle UX — neměň na tvrdé blokování.

## Architektura (`index.html`, sekce v `<script>` značené `=====`)

- **DATA — LEVELY**: pole `LEVELS`. Level = `{id, name, desc, weeks, periods, cards[], goals[]}`
  + volitelně `given` (zamčené výchozí pozice `{cardId:{w,d,p}}`), `carryFrom` (iron-man:
  id levelu, jehož autosave nahradí `given` jako zamčený základ) a `custom` (level z editoru).
  - karta: `{id, class, subject, field(IT|EP|ELE|STR), teacher, room, building, duration}`
  - placement: `{w: weekIndex, d: dayIndex 0–4, p: startPeriod 1-based}`
  - `field` řídí barvu kartičky (`FIELD_NAMES`, legenda se staví z oborů v levelu);
    `duration` = výška bloku (počet hodin).
- **SVĚT KAMPANĚ** (učebny a jejich zaměření skutečné; předměty a jména zástupná —
  lze 1:1 přejmenovat):
  - obory: `IT` informatika s mng., `EP` elektronické počítače, `ELE` slaboproud,
    `STR` strojírenství; třídy `<ročník>.<IT|EP|EL|ST>` (field dle oboru třídy).
  - **budovy = zaměření** (učebna má VŽDY jednu budovu — hlídá `validateLevel`):
    - `Š101` **T3, T5** kmenové · **T9** odborná (Fyzika, Hardware, Management, Ekonomika)
      · **T12, T14** počítačové laby — informatické předměty a teorie (kódy T1–T17).
    - `H59` **E2** elektro dílna · **F3** měřicí lab · **F5** lab mikroprocesorů ·
      **G1** číslicová technika · **G4** kmenová · **G7** tělocvična — elektrotechnika
      (kódy E/F/G 1–7).
    - `H618` **B7** kmenová · **B8** strojní dílna · **B26** rýsovna (CAD) · **C7**
      odborná · **C25** CNC — strojírenství (kódy B/C 7–9 a 25–27).
    - `Externí` **ZŠ 1–3** — půjčené tělocvičny, ad-hoc.
    - Odborný předmět jde do budovy svého zaměření; **všeobecné (Matematika, Angličtina,
      Fyzika) se učí na všech budovách** — obvykle tam, kde třída sedí. Právě tím vznikají
      přechody mezi budovami, na kterých stojí cíl `teacher_transition_gap`.
    - **Tělocvik je páka na obtížnost:** vlastní tělocvična G7/H59 u snadných levelů
      (l1, l2), externí ZŠ 1–3 u těžkých (l3–l6). Tomáš je jediný tělocvikář, takže
      učebna tělocviku sama o sobě nic neváže — tvrdý blok drží už učitel.
  - učitelé: Petr+Eva (IT), Karel+Ondřej (EP), Jan (EP/EL, přebíhá Š101↔H59),
    Alena (EL, zkrácený úvazek), Lucie+Martin (ST), Hana (mat, učí na všech budovách
    → přebíhá), Jitka (jazyky), Tomáš (tělocvik), Marie (management/ekonomika).
  - kampaň l1–l6, měřená obtížnost (výhry mezi náhodnými platnými rozvrhy, 4 000 vzorků):
    l1 ≈ 10,8 % · l2 ≈ 3,1 % · l3 ≈ 2,8 % · l4 ≈ 0,9 % · l5 (iron-man, záměrný oddech)
    ≈ 2,5 % · l6 „Prase" ≈ 0,4 %. Po změně dat/cílů přeměř („Řešitelnost?" nebo simulace).
- **ÚLOŽIŠTĚ**: adaptér `Store` s feature-detekcí `window.storage` → `localStorage` → null.
  Páteř ukládání je serializační řetězec Base64(JSON), funguje vždy. Import (modál,
  autosave i URL hash `#p=<řetězec>`) validuje placementy přes `validPlacement()`
  (rozsahy w/d/p + `fits`); hash se po načtení maže (`history.replaceState`) a
  neautosavuje. Dokončené levely drží set `DONE` (klíč `done`, ✓ v selectu). Motiv
  nastavuje synchronní skript v `<head>` (localStorage / `prefers-color-scheme`, žádný
  flash), async `Store` (artefakt) ho jen doladí.
- **STAV**: `LVL` (aktuální level) + `S = {placements, selected, view, active, pickedFrom}`.
  - `view` ∈ `class | teacher | room` = dimenze pohledu na rozvrh; `active` = vybraná
    entita v té dimenzi. Umisťovat lze jen ve `view==="class"`; `teacher`/`room` jsou
    jen pro čtení (kontrola konfliktů). Helpery: `entitiesOf(view)`, `entityOf(card,view)`.
  - `pickedFrom` = původní pozice zvednuté kartičky; Esc, zrušené tažení i výběr jiné
    karty ji přes `restoreLifted()` vrací na místo.
  - `assisted` = desku poskládal řešič nebo import → výhra ukáže jiný banner a NEdá ✓
    (fér ✓). Nastavují randomFill/importStr, nulují ruční akce, undo/redo a loadLevel;
    do exportu/autosave se propisuje jako `a:1`, takže přežije i reload.
  - `locked` = Set id kartiček zamčených levelem (`given`/`carryFrom`, aplikuje
    `applyGiven()` v `loadLevel`). Zamčené nejde zvednout/táhnout/odebrat, `clearAll`
    je zachová, autosave/import je nepřepisuje, řešič i kontrola řešitelnosti je berou
    jako pevný základ. V rozvrhu mají 🔒.
- **HISTORIE**: `HIST` + `pushHist/undo/redo` — snapshoty placements, per-level,
  tlačítka ↶/↷ + Ctrl+Z / Ctrl+Y (ignorují fokus v input/textarea). `pushHist()` se
  volá PŘED mutací placements; snapshot počítá zvednutou kartičku na jejím původním
  místě, takže přesun kartičky je jediný krok Zpět.
- **POINTER DRAG**: jednotné tažení myší i dotykem (sekce POINTER DRAG, žádné HTML5
  DnD). Klik(-klik) zůstává plnohodnotnou cestou; tažení začíná po prahu 5 px (myš) /
  podržení 200 ms (dotyk — švih normálně scrolluje, browser pošle pointercancel).
  Klon karty visí na `<body>`, listenery na dokumentu, pointer capture drží
  `document.body` (přežije re-render), hit-test přes `elementFromPoint` na
  `.cell.drop` (buňky nesou `data-w/d/p`). Scroll na dotyku blokuje non-passive
  `touchmove` až od začátku tažení; „duchový“ click po dropu tlumí capture blocker.
- **LOGIKA**: `cells()`, `computeConflicts()` (globálně napříč třídami),
  `hasHardConflict()` (pro řešič), `evalGoal()` (switch podle `g.type`).
- **ŘEŠIČ**: `randomFill(smart)` nad `fillOnce()` — randomizovaný backtracking se
  stropem, delší bloky se zkoušejí dřív; bere ručně umístěné i zamčené jako pevné,
  respektuje jen tvrdé požadavky. Se `smart=true` (checkbox v modálu „Náhodně“) běží
  multi-restart ~700 ms a vybere výsledek s nejvíc splněnými měkkými cíli
  (`goalsMetIn`). Když se vše nevejde, `blockNote()` nahlásí dominantní tvrdý blok
  (učitel/učebna/třída) u nevešlých kartiček.
- **KONTROLA ŘEŠITELNOSTI**: `checkSolvableAsync(levelId, samples, onProgress)` —
  vzorkuje náhodné kompletní rozvrhy chunkovaně (časové boxy ~25 ms, UI nezamrzá;
  globální `LVL`/`S` se prohazují jen uvnitř synchronního chunku s try/finally).
  0 výher ⇒ level je extrémně těžký / neřešitelný. Tlačítko „Řešitelnost?" otevírá
  modál s průběhem, zrušením a tabulkou per-cíl; z konzole `await checkSolvable("lX")`.
- **AKCE**: `tryPlace / pickUp / unplace / clearAll` (všechny respektují `S.locked`).
- **EDITOR LEVELŮ**: tlačítko „Editor“ v hlavičce → `editorModal()`. Vlastní level =
  JSON stejného tvaru jako `LEVELS` (šablona z existujícího levelu); `validateLevel()`
  kontroluje strukturu, id, typy cílů (`GOAL_TYPES`), `given` i `carryFrom`;
  řešitelnost jde otestovat před uložením (`checkSolvableAsync` přijímá i objekt
  levelu). Uložené levely drží Store klíč `customLevels`, v přepínači mají `*`,
  načítá (a validuje) je `loadCustoms()`; přepínač staví `buildLevelSel()`.
  Level jde sdílet odkazem `#l=<Base64 JSON>` (tlačítko „Odkaz na level“);
  příjem řeší `importLevelStr()` — validace, uložení mezi vlastní, načtení.
  Hash v initu: `#p=` pozice / `#l=` level, obojí až po `loadCustoms()`; vložení
  odkazu nad běžící hrou chytá `hashchange` (společná cesta `handleHash()`).
- **O HŘE**: `aboutModal()` (ⓘ v hlavičce) — kredit AI vývoje, čísla balancu,
  odkaz na repo (`REPO_URL` = https://github.com/senkyr/BoarShed).
- **RENDER**: `render()` → `renderPalette / renderTabs / renderSchedule / renderGoals`.
  `renderTabs` kreslí přepínač pohledu + záložky entit; `renderSchedule` umí všechny
  tři pohledy a překrývající se kartičky řeší „pruhy" (lanes), aby se neschovaly.
  **Orientace** dle šířky (`isHorizontal()`, breakpoint 820 px): desktop „na šířku"
  (dny = řádky, hodiny = sloupce), úzký displej transponovaně (hodiny = řádky, dolů).
  Pruhy se proto skládají kolmo na osu hodin (vodorovně vs. svisle). Při překlopení
  orientace překresluje listener na `resize`. Překresluje se celé (jednoduchost nad
  výkonem; při ~16 kartách to stačí).
- **el(tag, props, ...kids)** — mini-helper na DOM, bezpečný vůči diakritice.

### Typy herních cílů (`evalGoal`)

Přidání nového cíle = jedna `case` v `evalGoal` + položka v `goals` levelu. Cíl má
tvar `{id, type, p:{...parametry...}, label}`. Dostupné typy a jejich `p`:

| type | parametry `p` | význam |
|---|---|---|
| `all_placed` | — | všechny kartičky umístěné |
| `subject_time` | `subject, cls?, maxPeriod?/minPeriod?/exact?` | předmět v denním okně (`maxPeriod` = hodina, do které blok **končí**) |
| `subject_day` | `subject, cls?, day` | předmět v konkrétní den (0–4) |
| `subject_week` | `subject, cls?, week` | předmět v konkrétním týdnu (0/1) |
| `teacher_free_day` | `teacher` | učitel má někdy den volno |
| `teacher_only_week` | `teacher, week` | celý úvazek v jednom týdnu |
| `teacher_transition_gap` | `teacher` | po přechodu mezi budovami volná hodina |
| `class_no_gaps` | `cls` | třída bez oken |
| `max_per_day` | `cls?` **nebo** `teacher?`, `max` | max hodin denně |
| `lunch_break` | `cls, period` | třída má danou hodinu volnou (oběd) |
| `teacher_time` | `teacher, minPeriod?/maxPeriod?, forbidPeriods?[], forbidDays?[]` | okno / zakázané hodiny či dny učitele |

Pozn.: `subject_*` a další, co potřebují „vše umístěno", používají interní `allPlaced(fn)`.
`teacher_transition_gap` a `subject_week`/`building` mají smysl jen když data tu vlastnost
opravdu mají (učitel učí ve dvou budovách; víc týdnů).

Tvrdé požadavky (konflikt učitel/učebna/třída ve stejné buňce, souvislost bloku,
rozsah dne) jsou zapečené v enginu, **ne v datech**.

## Spuštění a ověření

- **Lokálně:** otevři `index.html` v prohlížeči (žádný server není potřeba).
- **Smoke test po zásahu:** projdi kampaň (minimálně l1, l2 a l5) — umísti kartu klikem i tahem
  (myší; na dotyku podržet ~200 ms, švih přes kartu musí scrollovat), zruš tažení
  puštěním mimo rozvrh a Escapem (karta se vrátí na původní místo), vyvolej konflikt
  (musí zčervenat + tooltip), překryj dvě karty stejné třídy (musí být vidět vedle
  sebe / nad sebou, ne zmizet), přepni pohled Třídy/Učitelé/Učebny (učitel/učebna jen
  pro čtení), zúži okno pod ~820 px (rozvrh se překlopí na hodiny dolů), spusť
  „Náhodně" (i s checkboxem měkkých cílů) a „Řešitelnost?" (modál s průběhem, jde
  zrušit), vyzkoušej ↶ Zpět / ↷ Znovu (i Ctrl+Z/Y), ulož/načti řetězec i pojmenovaný
  slot, zkopíruj odkaz a otevři ho (načte pozici, hash zmizí z URL), přepni motiv
  (tmavý nesmí při načtení bliknout světlým). V l5 ověř zamčené kartičky (🔒 — nejde
  zvednout, přežijí Smazat) a v Editoru ulož/smaž zkušební level (objeví se s `*`
  v přepínači) a pošli level odkazem („Odkaz na level" → otevřít v novém tabu).
  Ověř fér ✓ (výhra přes „Náhodně + zkusit cíle" ukáže banner „poskládal řešič" bez ✓;
  ruční tah pak povýší na pravou výhru) a otevři ⓘ O hře. Projdi světlý i tmavý motiv
  a úzké okno (<600 px): fixní lišta nesmí zakrývat spodek obsahu (padding-bottom
  media queries) a umístěné kartičky skrývají řádek budovy. Žádné chyby v konzoli.
  Dotyk otestuj i na reálném zařízení, ne jen v emulaci.
- **Po úpravě dat/cílů levelu** spusť „Řešitelnost?" (nebo `checkSolvable("lX")`
  v konzoli) — ať víš, že je level vůbec vyhratelný a jak je těžký.

## Konvence

- Zachovej stávající styl: kompaktní vanilla JS, komentáře `===== SEKCE =====`,
  hustota komentářů jako v okolí.
- Nová data (třídy, učebny, učitelé, levely) patří do pole `LEVELS`.
- Při změně chování aktualizuj i [docs/handoff.md](docs/handoff.md), pokud se tím
  mění některé z tam popsaných rozhodnutí nebo TODO.

## Co je rozpracované / TODO

Hotovo nad rámec původního MVP: pohledy za učitele/učebnu, nové typy cílů
(`subject_week`, `max_per_day`, `lunch_break`, `teacher_time`, aktivované
`teacher_transition_gap`), kontrola řešitelnosti (chunkovaný `checkSolvableAsync`
+ modál s tabulkou per-cíl), oprava překrývajících se kartiček (lanes), jednotný
pointer-drag (myš i dotyk), undo/redo, validace importovaných pozic, evidence
dokončených levelů (✓), sdílení pozice přes URL hash, motiv bez flashe, **iron-man**
(zamčené kartičky `given`/`carryFrom`, v kampani level l5), **editor levelů** (JSON,
vlastní levely ve Store, sdílení levelů odkazem `#l=`), **chytřejší řešič**
(heuristika delších bloků, volitelné plnění měkkých cílů, hlášení co zablokovalo
doplnění), **fér ✓** (výhra řešičem/importem nedává odznak) a **modál ⓘ O hře**.

Obsah je hotový „naslepo": kampaň 6 levelů (l1–l6, viz SVĚT KAMPANĚ výš), obtížnost
vybalancovaná simulacemi a ověřená ručním odehráním přes UI bez řešiče (l1–l4 + l6
vyhrány, 07/2026).

Nasazeno (27. 7. 2026): veřejné repo `senkyr/BoarShed`, hosting GitHub Pages
(https://senkyr.github.io/BoarShed/, větev `main`, push = redeploy); `REPO_URL`
v `aboutModal` doplněna. Pozor: gh účet `senkyr` je projektový — Cortex jede na
`jakub-senkyr` (pravidla přepínání: `cortex-meta:setup.md` → „GitHub účty").

Zbývá:

1. **Výměna zástupných jmen za podvýběr skutečných** — čistě 1:1 přejmenování v datech
   (pole `LEVELS`), struktura ani cíle se nemění; po výměně přeměřit řešitelnost.
   Rozsah, který je potřeba dodat (aktuální zástupné hodnoty a jejich role):
   - **12 křestních jmen učitelů**: Petr + Eva (IT), Karel + Ondřej (EP), Jan (EP/EL,
     přebíhá Š101↔H59), Alena (EL, zkrácený úvazek — jen sudý týden), Lucie + Martin (ST),
     Hana (matematika, učí na všech budovách → přebíhá), Jitka (jazyky), Tomáš
     (tělocvik, jediný tělocvikář), Marie (management/ekonomika).
   - **19 názvů předmětů**: všeobecné — Matematika, Angličtina, Tělocvik, Fyzika;
     IT — Programování, Databáze, Web, Management, Ekonomika; EP — Číslicová technika,
     Mikroprocesory, Hardware; EL — Elektronika, Měření; ST — CAD, CNC, Praxe,
     Stavba strojů, Technologie.
   - ~~kódy učeben~~ — **hotovo 9. 8. 2026**: učebny i zaměření budov jsou skutečné,
     viz „Svět kampaně" výš. Budova OPMB ze zadání zatím nepoužita.
   - volitelně **11 označení tříd** (teď `1.IT`, `1.EP`, `2.EL`, `2.ST`, `3.IT`,
     `3.EP`, `3.ST`, `4.IT`, `4.EP`, `4.EL`, `4.ST`).
2. Ruční otestování dotyku na reálném mobilu/tabletu (hold ~200 ms = tažení,
   švih = scroll) — na nasazené URL.

Pozn.: „vázanost předmětu na budovu" z původního zadání není herní cíl — budova je
pevná vlastnost kartičky, řeší se návrhem dat levelu.
