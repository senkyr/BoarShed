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

- **DATA — LEVELY**: pole `LEVELS`. Level = `{id, name, desc, weeks, periods, cards[], goals[]}`.
  - karta: `{id, class, subject, field(IT|ELE|STR), teacher, room, building, duration}`
  - placement: `{w: weekIndex, d: dayIndex 0–4, p: startPeriod 1-based}`
  - `field` řídí barvu kartičky; `duration` = výška bloku (počet hodin).
- **ÚLOŽIŠTĚ**: adaptér `Store` s feature-detekcí `window.storage` → `localStorage` → null.
  Páteř ukládání je serializační řetězec Base64(JSON), funguje vždy.
- **STAV**: `LVL` (aktuální level) + `S = {placements, selected, view, active, over}`.
  - `view` ∈ `class | teacher | room` = dimenze pohledu na rozvrh; `active` = vybraná
    entita v té dimenzi. Umisťovat lze jen ve `view==="class"`; `teacher`/`room` jsou
    jen pro čtení (kontrola konfliktů). Helpery: `entitiesOf(view)`, `entityOf(card,view)`.
- **LOGIKA**: `cells()`, `computeConflicts()` (globálně napříč třídami),
  `hasHardConflict()` (pro řešič), `evalGoal()` (switch podle `g.type`).
- **ŘEŠIČ**: `randomFill()` — randomizovaný backtracking se stropem 200 000 uzlů,
  bere ručně umístěné jako pevné, respektuje jen tvrdé požadavky.
- **KONTROLA ŘEŠITELNOSTI**: `checkSolvable(levelId, samples)` — vzorkuje náhodné
  kompletní rozvrhy a měří % výher (a % splnění u každého cíle). 0 výher ⇒ level je
  extrémně těžký / neřešitelný. Volá ho tlačítko „Řešitelnost?" i konzole.
- **AKCE**: `tryPlace / pickUp / unplace / clearAll`.
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
| `subject_time` | `subject, cls?, maxPeriod?/minPeriod?/exact?` | předmět v denním okně |
| `subject_day` | `subject, cls?, day` | předmět v konkrétní den (0–4) |
| `subject_week` | `subject, cls?, week` | předmět v konkrétním týdnu (0/1) |
| `teacher_free_day` | `teacher` | učitel má někdy den volno |
| `teacher_only_week` | `teacher, week` | celý úvazek v jednom týdnu |
| `teacher_transition_gap` | `teacher` | po přechodu mezi budovami volná hodina |
| `class_no_gaps` | `cls` | třída bez oken |
| `max_per_day` | `cls?` **nebo** `teacher?`, `max` | max hodin denně |
| `lunch_break` | `cls, period` | třída má danou hodinu volnou (oběd) |

Pozn.: `subject_*` a další, co potřebují „vše umístěno", používají interní `allPlaced(fn)`.
`teacher_transition_gap` a `subject_week`/`building` mají smysl jen když data tu vlastnost
opravdu mají (učitel učí ve dvou budovách; víc týdnů).

Tvrdé požadavky (konflikt učitel/učebna/třída ve stejné buňce, souvislost bloku,
rozsah dne) jsou zapečené v enginu, **ne v datech**.

## Spuštění a ověření

- **Lokálně:** otevři `index.html` v prohlížeči (žádný server není potřeba).
- **Smoke test po zásahu:** projdi obě demo úrovně — umísti kartu klikem i tahem,
  vyvolej konflikt (musí zčervenat + tooltip), překryj dvě karty stejné třídy (musí
  být vidět vedle sebe / nad sebou, ne zmizet), přepni pohled Třídy/Učitelé/Učebny
  (učitel/učebna jen pro čtení), zúži okno pod ~820 px (rozvrh se překlopí na hodiny
  dolů), spusť „Náhodně" a „Řešitelnost?", ulož/načti řetězec, přepni motiv.
  Žádné chyby v konzoli.
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
(`subject_week`, `max_per_day`, `lunch_break`, aktivované `teacher_transition_gap`),
kontrola řešitelnosti (`checkSolvable` / tlačítko „Řešitelnost?"), oprava
překrývajících se kartiček (lanes).

Zbývá (viz [docs/handoff.md → sekce 5](docs/handoff.md)): **reálná data** (seznamy
tříd/učeben+budov/učitelů/předmětů) a návrh dalších levelů, iron-man režim (přenos
pozic mezi levely), chytřejší řešič (volitelně i měkké cíle), generátor/editor levelů,
dotykový pointer-drag, případně další typy cílů (zakázané/preferované hodiny učitele).
Než začneš na obsahu, ujasni si s Jakubem priority a chybějící vstupy.
