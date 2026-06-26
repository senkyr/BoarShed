# Skládání rozvrhu — předávací dokument

Logická hra pro kolegy, kde se „skládá rozvrh“ (in-joke; teoreticky může být i neřešitelně těžká).
Stav: **funkční kostra (MVP)** v jediném souboru `rozvrh.html`. Tento dokument shrnuje původní zadání,
co je hotové a jaká rozhodnutí padla během vývoje — jako podklad pro pokračování v Claude Code.

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

## 2. Co je hotové (stav MVP)

Vše je v jediném souboru `rozvrh.html` — čistý HTML + CSS + vanilla JS, **bez závislostí a bez build
kroku**. Renderuje se jako artefakt a zároveň je to přímo soubor pro nasazení jako Render **Static Site**.

**UI a rozvržení**
- Tři sloupce: Kartičky / Rozvrh / Požadavky. Responsivní — pod 1100 px se skládají pod sebe.
- Hlavička: výběr levelu, tlačítko Uložit/Načíst, přepínač světlý/tmavý režim.
- Spodní lišta: Smazat, Náhodně, stavový řádek.
- Světlý/tmavý motiv přes CSS proměnné, výchozí světlý, volba se ukládá.

**Herní smyčka**
- Umístění kartičky kliknutím (vyber kartičku → přepne se na záložku její třídy → klikni do buňky)
  i přes **drag-and-drop** (HTML5).
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

**Demo levely**
- L1 „Rozjezd“ — jedna třída, jeden týden, jedna budova, jen IT.
- L2 „Dva týdny, dvě budovy“ — lichý i sudý týden, dvě třídy, dvě budovy, mix oborů, sdílené učebny.

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
- `field` ∈ `IT | ELE | STR` (řídí barvu kartičky).
- `duration` = výška bloku v hodinách.
- Umístění (placement): `{ w: weekIndex, d: dayIndex(0–4), p: startPeriod(1-based) }`.
- `DAYS = ["Po","Út","St","Čt","Pá"]`.

**Klíčové funkce:**
- `cells(id, pl)` — buňky, které kartička zabírá.
- `computeConflicts()` — vrací `{ cardId: Set<důvod> }`; sdílená buňka + shoda učitel/učebna/třída.
- `hasHardConflict(id, pl, placements)` — tvrdý konflikt pro řešič.
- `evalGoal(g)` — vyhodnocení jednoho cíle (switch podle `g.type`).
- `randomFill()` — řešič (backtracking).
- `tryPlace / pickUp / unplace / clearAll` — herní akce.
- `render / renderPalette / renderTabs / renderSchedule / renderGoals` — celé překreslení (jednoduchost
  nad výkonem; při ~16 kartičkách v pohodě).
- `exportStr / importStr / autosave / loadAutosave` + adaptér `Store` — ukládání.
- `el(tag, props, ...kids)` — mini-helper na tvorbu DOM (bezpečný vůči jménům s diakritikou).

**Typy cílů, které engine umí** (přidání dalšího = jedna `case` v `evalGoal`):
- `all_placed` — vše umístěno.
- `subject_time` — `{subject, cls?, maxPeriod?/minPeriod?/exact?}` — denní okno.
- `subject_day` — `{subject, cls?, day}` — konkrétní den.
- `teacher_free_day` — `{teacher}` — některý den bez hodin.
- `teacher_only_week` — `{teacher, week}` — celý úvazek v jednom týdnu.
- `class_no_gaps` — `{cls}` — třída bez oken.
- `teacher_transition_gap` — `{teacher}` — přechod mezi budovami vyžaduje volnou hodinu
  (implementováno, v demu zatím nepoužito).

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
v tooltipu. Pohled „za učitele“ / „za učebnu“ zatím není.

**Jen režim „na kola“.** Iron man (levely na sebe) není postaven; datový model je na to připravený
(předchozí pozice by se vložily jako zamčené „dané“ kartičky).

**Periody = 6 hodin/den** jako předpoklad. Demo data (učitelé, učebny) jsou výplň.

**Drag na desktopu (HTML5 DnD); na dotyku se spoléhá na klikání.** HTML5 DnD se na touchi nespouští,
proto je klik-klik plnohodnotná alternativa.

---

## 5. Otevřené body / co dořešit / TODO

Od zadavatele:
1. **Seznamy:** třídy; učebny (a ke každé budova Š101/H59/H618/OPMB); učitelé (křestní jména);
   předměty s oborem.
2. **Návrh levelů:** pro každý level co obsahuje (třídy/budovy/obory) a které cíle se mají splnit;
   rozlišit tvrdé vs. herní.
3. **Režim:** kola vs. iron man (nebo oba); zda chtít i pohled za učitele/učebnu.

Technické TODO / nápady k dořešení:
- Iron man: přenos pozic z předchozího levelu jako zamčené kartičky.
- Pohledy „za učitele“ a „za učebnu“ (jen pro čtení, kontrola konfliktů).
- Další typy cílů: vázanost předmětu na budovu, max hodin denně pro třídu/učitele, polední pauza,
  preferované/zakázané hodiny učitele.
- Řešič: zohlednit i část měkkých požadavků (volitelně), lepší heuristika výběru slotů, hlášení,
  který tvrdý požadavek doplnění zablokoval.
- Dotyk: případně vlastní pointer-drag pro plynulé tažení na tabletu.
- Generátor/editor levelů (aby se levely nemusely psát ručně v poli `LEVELS`).
- Kontrola, zda je level vůbec řešitelný (užitečné při ladění obtížnosti).

---

## 6. Spuštění a nasazení

- **Lokálně:** otevři `rozvrh.html` v prohlížeči. Žádná instalace ani build.
- **Render (Static Site):** přejmenuj na `index.html`, dej do repa, na Renderu vyber *Static Site*,
  Free instance. Žádný build command není potřeba. Neusíná.
- **Závislosti:** žádné. Vše je v jednom souboru.
