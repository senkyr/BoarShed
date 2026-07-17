# Skládání rozvrhu — předávací dokument

Logická hra pro kolegy, kde se „skládá rozvrh“ (in-joke; teoreticky může být i neřešitelně těžká).
Stav: **hotová hra** (kampaň 6 levelů s měřeným balancem, editor, sdílení, studentská edice)
v jediném souboru `index.html`. Tento dokument shrnuje původní zadání, co je hotové a jaká
rozhodnutí padla během vývoje — jako podklad pro pokračování v Claude Code.

---

## 1. Původní zadání

Hra pro kolegy, jednoduchá logická hra, kde budou „skládat rozvrh“. Je to in-joke a hra může být
teoreticky i neřešitelně těžká.

**Rozvržení obrazovky** (podle ručního nákresu):
- vlevo seznam (různobarevných) kartiček k rozložení,
- uprostřed samotný rozvrh — pracuje se s **lichým a sudým týdnem**, vždy Po–Pá,
- vpravo seznam požadavků (co smí být kde, co kde být nesmí).

Hráč přetahuje kartičky do rozvrhu a snaží se splnit současně všechny požadavky.

**Každá vyučovaná hodina (kartička) má:**
- třídu (seznam dodá zadavatel),
- učebnu (seznam dodá zadavatel),
- předmět (z oborů IT, elektrotechnika, strojírenství),
- budovu (Š101, H59, H618, OPMB),
- vyučujícího (ve hře jen křestní jména).

**Pravidla a požadavky:**
- Sestavuje se rozvrh pro třídy, čímž zároveň vzniká rozvrh učeben a vyučujících.
- Nesmí docházet ke konfliktům (vyučující na dvou místech současně; dva předměty ve stejnou hodinu
  v jedné učebně).
- Existuje velká škála dalších požadavků:
  - předmět může být v blocích 1, 2, nebo 3 hodiny dlouhých,
  - některé předměty lze učit jen na konkrétních budovách,
  - vyučující někdy musí přecházet mezi budovami a potřebují na přechod volnou hodinu,
  - některé předměty se musí učit v konkrétní den nebo konkrétní čas, atd.
- Některé požadavky jsou **stálé a napevno zapečené v designu hry** (např. blok cvičení má vždy
  2 hodiny — kartička už může být tak velká).
- Jiné jsou dané jako **cíl hry** (vyučující chce volný den; vyučující má zkrácený úvazek a učí jen
  v jednom týdnu apod.).
- Případně jako **skóre** (rozvrh je poskládán, ale splněno je jen určité procento úkolů).

**Obtížnost:**
- Rostoucí, doménově specifická — začne se s méně budovami/obory, požadavky budou nejdřív jednoduché
  a pak stále krkolomnější.
- Buď **levely nezávislé** (začne se od nuly), nebo **„realistický“ model**, kde se levely skládají
  na sebe a je třeba pracovat s tím, co už vzniklo. Mohly by být oba režimy (**na kola** vs.
  **iron man**).

**Tlačítka:**
- „smazat“ — odstraní všechny kartičky z rozvrhu ven,
- „random“ — náhodně rozmístí kartičky do rozvrhu tak, aby byly dodrženy **tvrdé (neherní)
  požadavky**; hráč může z náhodného rozmístění pokračovat.
- Obě tlačítka vyžadují potvrzení (aby nešla stisknout omylem).
- „random“ by mohlo rozmisťovat **pouze dosud neumístěné** kartičky — hráč tak může část rozmístit
  ručně a zbytek nechat náhodně (tvrdé požadavky ale mohou zabránit kompletnímu rozmístění zbylých).

**Ukládání:**
- Hráč si musí umět „poznamenávat“ dosažená (i částečná) řešení a vracet se k nim.
- Ideálně lokálně (cookies / serializační řetězec), aby se neřešily účty.
- S rostoucí obtížností nepůjde „rovnou vyhrát“ (snad kromě nejnižších úrovní).

**Nasazení:**
- Nejraději jako artefakt; pokud by per-user ukládání nešlo, pak frontendová webová aplikace bez
  databáze, nasaditelná na Renderu zdarma bez usínání a čekání na deploy.

---

## 2. Co je hotové

Vše je v jediném souboru `index.html` — čistý HTML + CSS + vanilla JS, **bez závislostí a bez build
kroku**. Renderuje se jako artefakt a zároveň je to přímo soubor pro nasazení jako Render **Static Site**.

**Doplněno po MVP (iterace 06–07/2026):** pohledy za učitele/učebnu (jen pro čtení), rozvrh
„na šířku" na desktopu, lanes pro překrývající se kartičky, nové typy cílů (`subject_week`,
`max_per_day`, `lunch_break`, `teacher_time`, aktivní `teacher_transition_gap`), kontrola
řešitelnosti (modál s průběhem a tabulkou per-cíl), **jednotný pointer-drag** (myš i dotyk,
hold-to-drag 200 ms), **undo/redo** (↶/↷, Ctrl+Z/Y), validace importovaných pozic, evidence
dokončených levelů (✓ v selectu), sdílení pozice odkazem (`#p=…`), motiv bez flashe s respektem
k systémové preferenci, **iron-man** (zamčené kartičky `given`/`carryFrom`, demo level L3
staví na pozici z L2), **editor levelů** (JSON s validací a testem řešitelnosti, vlastní levely
se ukládají do úložiště a v přepínači mají `*`) a **chytřejší řešič** (delší bloky dřív,
volitelný multi-restart na měkké cíle, hlášení kterého tvrdého požadavku se doplnění zaseklo).
Dále: opravy z adversarial bug huntu (11 nálezů) a race conditions async úložiště, mobilní
layout (rezerva pod zalomenou lištou, výraznější drop zóny i na slunci, skrytý řádek budovy
v úzkých buňkách), studentská edice (detail v §5) a **ruční playtest** — l1–l4 + l6 odehrány
a vyhrány přes UI bez řešiče, balanc sedí i pocitově (07/2026).

**UI a rozvržení**
- Tři sloupce: Kartičky / Rozvrh / Požadavky. Responsivní — pod 1100 px se skládají pod sebe.
- Hlavička: výběr levelu, Editor, Uložit/Načíst, ⓘ O hře, přepínač světlý/tmavý režim.
- Spodní lišta: Zpět/Znovu, Smazat, Náhodně, Řešitelnost?, stavový řádek.
- Světlý/tmavý motiv přes CSS proměnné, výchozí dle systémové preference, volba se ukládá.

**Herní smyčka**
- Umístění kartičky kliknutím (vyber kartičku → přepne se na záložku její třídy → klikni do buňky)
  i **tažením** (jednotný pointer-drag, myš i dotyk).
- Zvednutí už umístěné kartičky kliknutím (přesun), „×“ pro odebrání, Esc ruší výběr.
- Bloky 1/2/3 h zabírají odpovídající počet řádků; při umístění se hlídá, že se blok vejde do dne.
- Konflikty se počítají **globálně napříč třídami**, zvýrazní se červeně, důvod je v tooltipu.
- Pohled je **po třídách** (záložky tříd).

**Požadavky (pravý panel)**
- Živé vyhodnocení cílů, skóre v %, progress bar, počet konfliktů, u každého cíle stav (✓ / prázdné).
- Výherní stav, když jsou všechny cíle splněné **a** je nula konfliktů.
- Legenda barev oborů.

**Řešič — tlačítko „Náhodně“**
- Randomizovaný backtracking se stropem (200 000 uzlů).
- Bere ručně umístěné kartičky jako pevné, doplňuje **jen zbývající**.
- Respektuje **pouze tvrdé požadavky** (konflikty učitel/učebna/třída, délka bloku, rozsah dne);
  měkké cíle nechává na hráči.
- Když nedoplní vše, vrátí nejlepší částečné rozmístění a nahlásí „X z Y“.

**Ukládání**
- Páteř: **serializační řetězec** = Base64(JSON) celé pozice; kopíruj a kdykoli vlož zpět. Funguje
  všude, i po nasazení.
- Adaptér úložiště s feature-detekcí: `window.storage` (artefakt) → `localStorage` (Render) → jinak
  jen řetězec. Vše v `try/catch`, takže výpadek jen vypne automatické ukládání.
- Pojmenované sloty per-level, autosave per-level (`auto:<levelId>`), uložení motivu.

**Potvrzovací modály** pro Smazat a Náhodně; modál Uložit/Načíst s kopírováním, načtením z řetězce
a seznamem slotů.

**Kampaň**
- 6 levelů `l1`–`l6` (Rozjezd → Dvě třídy → Dva týdny a dvě budovy → Tři budovy →
  Iron-man dostavba → finále „Prase") ve světě 4 oborů, ročníků 1–4 a 3 budov.
  Rozsah, zástupná data a naměřený balanc: §5 „Obsah" a CLAUDE.md → „Svět kampaně".

---

## 3. Architektura a kde co je

Soubor je rozdělen do logických sekcí (komentáře `=====` v `<script>`):

**Datový model** — pole `LEVELS`. Každý level:
```js
{ id, name, desc,
  weeks: ["Lichý","Sudý"],   // 1 nebo 2 týdny
  periods: 6,                 // hodin za den
  cards: [ {id, class, subject, field, teacher, room, building, duration} ],
  goals: [ {id, type, p:{...}, label} ] }
```
- `field` ∈ `IT | EP | ELE | STR` (řídí barvu kartičky; `FIELD_NAMES` + dynamická legenda).
- `duration` = výška bloku v hodinách.
- Umístění (placement): `{ w: weekIndex, d: dayIndex(0–4), p: startPeriod(1-based) }`.
- `DAYS = ["Po","Út","St","Čt","Pá"]`.
- Level navíc může mít `given` (zamčené výchozí pozice) a `carryFrom` (iron-man).

**Klíčové funkce** (detailní a aktuální popis architektury drží CLAUDE.md — jediný zdroj pravdy):
- `cells(id, pl)` — buňky, které kartička zabírá.
- `computeConflicts()` — vrací `{ cardId: Set<důvod> }`; sdílená buňka + shoda učitel/učebna/třída.
- `hasHardConflict(id, pl, placements)` — tvrdý konflikt pro řešič.
- `evalGoal(g)` — vyhodnocení jednoho cíle (switch podle `g.type`).
- `randomFill(smart)` nad `fillOnce()` — řešič; `goalsMetIn` / `blockNote` pro chytrý režim a hlášení.
- `checkSolvableAsync(...)` — chunkovaná kontrola řešitelnosti (modál + konzole).
- `tryPlace / pickUp / restoreLifted / unplace / clearAll` — herní akce; `HIST` + `undo/redo`.
- `applyGiven / loadLevel` — zamčené kartičky (`S.locked`), iron-man přenos.
- `render / renderPalette / renderTabs / renderSchedule / renderGoals` — celé překreslení (jednoduchost
  nad výkonem; při ~20 kartičkách v pohodě).
- `exportStr / importStr / handleHash / importLevelStr / autosave / loadAutosave` + adaptér `Store`.
- `validateLevel / editorModal / loadCustoms` — editor a vlastní levely; `aboutModal` — O hře.
- `el(tag, props, ...kids)` — mini-helper na tvorbu DOM (bezpečný vůči jménům s diakritikou).

**Typy cílů:** 11 typů (`all_placed`, `subject_time/day/week`, `teacher_free_day/only_week/`
`transition_gap/time`, `class_no_gaps`, `max_per_day`, `lunch_break`) — aktuální tabulku
parametrů drží CLAUDE.md → „Typy herních cílů". Přidání dalšího = jedna `case` v `evalGoal`
+ povinné parametry do `GOAL_TYPES`.

**Tvrdé požadavky** jsou zapečené v enginu (ne v datech): konflikt učitel/učebna/třída ve stejné
buňce, souvislost bloku (daná tím, že blok zabírá po sobě jdoucí hodiny), rozsah dne.

---

## 4. Rozhodnutí učiněná během vývoje (a jejich důvody)

**Formát: jeden HTML soubor místo React `.jsx`.** Renderuje se jako artefakt, ale hlavně jde rovnou
nasadit jako Render Static Site bez build kroku, a dává plnou kontrolu nad drag-and-drop, CSS grid
spany a motivem bez omezení Tailwindu.

**Render = Static Site, ne Web Service.** Free Web Service usíná po 15 min nečinnosti (cold start
30–60 s); statický web na Renderu je CDN bez časového limitu a neusíná. Protože je to čistý frontend
bez DB, je to přesně tenhle případ. Soubor stačí přejmenovat na `index.html`.

**Ukládání přes serializační řetězec jako páteř.** V artefaktu nelze použít `localStorage`
(sandbox ho blokuje). Řetězec (Base64 JSON) funguje všude; adaptér navíc využije `window.storage`
v artefaktu a `localStorage` po nasazení, bez nutnosti cokoli měnit.

**Kartičky nesou třídu/učebnu/budovu/předmět/učitele napevno; hráč rozhoduje jen o ČASE.** Výklad
zadání („každá kartička má…“): tyto atributy jsou součástí kartičky, jediný stupeň volnosti je
umístění v čase (týden/den/hodina). Budova zůstala explicitní vlastností kartičky (i když by šla
odvodit z učebny) — podle zadání.

**Konflikty se povolí a zvýrazní, neblokují se.** Lepší puzzle UX — hráč může konflikt vytvořit,
vidí ho červeně a opraví. Výhra vyžaduje nula konfliktů. (Přesah bloku mimo den se blokuje.)

**Pohled po třídách s globální detekcí konfliktů.** Konflikt v 1.A může být kvůli 2.B — důvod je
v tooltipu. Pohled „za učitele“ / „za učebnu“ zatím není. *(Překonáno: pohledy doplněny později,
jen pro čtení.)*

**Jen režim „na kola“.** Iron man (levely na sebe) není postaven; datový model je na to připravený
(předchozí pozice by se vložily jako zamčené „dané“ kartičky). *(Překonáno 07/2026: iron-man
postaven — `given`/`carryFrom`, v kampani level l5.)*

**Periody = 6 hodin/den** jako předpoklad. Demo data (učitelé, učebny) jsou výplň.

**Drag na desktopu (HTML5 DnD); na dotyku se spoléhá na klikání.** HTML5 DnD se na touchi nespouští,
proto je klik-klik plnohodnotná alternativa.

**HTML5 DnD nahrazen pointer events (07/2026).** HTML5 DnD nefungoval na dotyku a nesměl se při něm
re-renderovat DOM (drop zóny se nikdy nezvýrazňovaly); pointer-drag je jedna cesta pro myš i dotyk
a řeší i bug s „neviditelným výběrem" po přerušeném tažení. Na dotyku se táhne po podržení ~200 ms —
`touch-action:none` by zablokoval scroll stránky přes kartičky, proto se scroll blokuje až od začátku
tažení non-passive `touchmove` listenerem. Klik-klik zůstává plnohodnotná cesta.

**Esc, zrušené tažení i výběr jiné karty vrací zvednutou kartičku na původní místo** (dřív spadla do
palety a pozice byla pryč). Undo počítá zvednutou kartu na původním místě → přesun je jeden krok Zpět.

**Sdílení odkazem maže hash po načtení** (`history.replaceState`) a neautosavuje, aby otevření cizího
odkazu nepřepsalo vlastní rozehranou pozici daného levelu.

---

## 5. Otevřené body / co dořešit / TODO

Od zadavatele (stav 07/2026):
1. **Podvýběr skutečných jmen** pro 1:1 výměnu za zástupná (jediný zbývající vstup).
   Rozsah: **12 křestních jmen učitelů, 19 názvů předmětů, 12 kódů učeben, volitelně
   11 označení tříd** — přesné aktuální zástupné hodnoty a role jednotlivých učitelů
   drží CLAUDE.md → „Co je rozpracované / TODO". Po výměně přeměřit řešitelnost.
2. ~~Návrh levelů~~ — vyřešeno kampaní „naslepo" (níže); úpravy jdou dělat v editoru.
3. ~~Režim kola vs. iron-man; pohledy za učitele/učebnu~~ — postaveno obojí
   (iron-man = l5, pohledy hotové).

**Obsah (07/2026):** kampaň 6 levelů postavená „naslepo" — svět inspirovaný skutečností
(4 obory: informatika s mng. `IT`, elektronické počítače `EP`, slaboproud `ELE`,
strojírenství `STR`; ročníky 1–4; budovy Š101/H59/H618; ~12 učitelů s křestními jmény;
zástupné předměty). Obtížnost balancovaná simulacemi (podíl výher mezi náhodnými
platnými rozvrhy): l1 9,6 % → l2 3,6 % → l3 2,6 % → l4 1,1 % → l5 iron-man 2,7 %
(záměrný oddech před finále) → l6 „Prase" 0,4 %. Vzorové řešení l4 (given pro l5) bylo
nalezeno a ověřeno enginem; řešitelnost všech levelů potvrzena vzorkováním (l6 na
20 000 vzorcích). Není to simulace reality — jména a předměty jde později 1:1 vyměnit
za podvýběr skutečných (pak přeměřit řešitelnost).

**Studentská edice (07/2026):** hra má druhé publikum — studenty (hrají intenzivněji než
kolegové na akci) a roli demonstrátoru AI vývoje. Doplněno: **sdílení vlastních levelů
odkazem** (`#l=…`, editor → „Odkaz na level“; příjemci se uloží mezi vlastní), **fér ✓**
(výhra poskládaná řešičem nebo importem ukáže banner, ale odznak ✓ je jen za vlastnoruční
výhru; příznak `a:1` přežívá v autosave i exportu) a **modál ⓘ O hře** (kredit AI vývoje,
čísla balancu, tip na konzoli; `REPO_URL` doplnit po zveřejnění repa — veřejné repo je
součást demonstrátoru). Konzolové nástroje (`checkSolvable` apod.) zůstávají záměrně
přístupné — pro IT studenty easter egg, ne díra.

Technické TODO / nápady:
- Výměna zástupných předmětů/jmen za podvýběr skutečných (jen přejmenování v datech).
- Případné další typy cílů podle potřeb. Pozn.: „vázanost předmětu na budovu"
  není herní cíl — budova je pevná vlastnost kartičky, řeší se návrhem dat levelu.
- Editor levelů: případně formulářová nadstavba nad JSON (teď se edituje JSON přímo).
- Iron-man: `carryFrom` přenáší autosave zdrojového levelu (nemusí to být výherní pozice);
  případně zpřísnit na „přenést jen po výhře".

Hotové z původního TODO (viz §2): pohledy za učitele/učebnu, kontrola řešitelnosti, dotykový
pointer-drag, iron-man (zamčené kartičky + `carryFrom`), editor levelů, chytřejší řešič
(heuristika, měkké cíle, hlášení bloku), cíle max hodin denně, polední pauza
a preferované/zakázané hodiny učitele (`teacher_time`). Nad rámec: undo/redo, validace
importu, ✓ u dokončených levelů, sdílení pozice odkazem.

---

## 6. Spuštění a nasazení

- **Lokálně:** otevři `index.html` v prohlížeči. Žádná instalace ani build.
- **Render (Static Site):** repo na GitHub, na Renderu vyber *Static Site*, Free instance,
  publish directory = kořen repa. Žádný build command není potřeba. Neusíná.
- **Závislosti:** žádné. Vše je v jednom souboru. (Potřebuje prohlížeč ~2023+ — `color-mix`,
  pointer events.)
