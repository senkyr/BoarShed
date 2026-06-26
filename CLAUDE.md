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
- **STAV**: `LVL` (aktuální level) + `S = {placements, selected, activeClass, over}`.
- **LOGIKA**: `cells()`, `computeConflicts()` (globálně napříč třídami),
  `hasHardConflict()` (pro řešič), `evalGoal()` (switch podle `g.type`).
- **ŘEŠIČ**: `randomFill()` — randomizovaný backtracking se stropem 200 000 uzlů,
  bere ručně umístěné jako pevné, respektuje jen tvrdé požadavky.
- **AKCE**: `tryPlace / pickUp / unplace / clearAll`.
- **RENDER**: `render()` → `renderPalette / renderTabs / renderSchedule / renderGoals`.
  Překresluje se celé (jednoduchost nad výkonem; při ~16 kartách to stačí).
- **el(tag, props, ...kids)** — mini-helper na DOM, bezpečný vůči diakritice.

### Typy herních cílů (`evalGoal`)

Přidání nového cíle = jedna `case` v `evalGoal` + položka v `goals` levelu:
`all_placed`, `subject_time`, `subject_day`, `teacher_free_day`,
`teacher_only_week`, `class_no_gaps`, `teacher_transition_gap`.

Tvrdé požadavky (konflikt učitel/učebna/třída ve stejné buňce, souvislost bloku,
rozsah dne) jsou zapečené v enginu, **ne v datech**.

## Spuštění a ověření

- **Lokálně:** otevři `index.html` v prohlížeči (žádný server není potřeba).
- **Smoke test po zásahu:** projdi obě demo úrovně — umísti kartu klikem i tahem,
  vyvolej konflikt (musí zčervenat + tooltip), spusť „Náhodně", ulož/načti řetězec,
  přepni motiv. Žádné chyby v konzoli.

## Konvence

- Zachovej stávající styl: kompaktní vanilla JS, komentáře `===== SEKCE =====`,
  hustota komentářů jako v okolí.
- Nová data (třídy, učebny, učitelé, levely) patří do pole `LEVELS`.
- Při změně chování aktualizuj i [docs/handoff.md](docs/handoff.md), pokud se tím
  mění některé z tam popsaných rozhodnutí nebo TODO.

## Co je rozpracované / TODO

Viz [docs/handoff.md → sekce 5](docs/handoff.md): iron-man režim (přenos pozic mezi
levely), pohledy za učitele/učebnu, další typy cílů, chytřejší řešič, generátor
levelů, dotykový pointer-drag. Než začneš na něčem z toho, ujasni si s Jakubem
priority a chybějící vstupy (seznamy tříd/učeben/učitelů/předmětů).
