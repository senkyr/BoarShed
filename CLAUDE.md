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
  - karta: `{id, class, subject, field(IT|EP|ELE|STR|BRK), teacher, room, building, duration}`
    + volitelně `kind:"break"` (nařízená pauza — viz níž)
  - placement: `{w: weekIndex, d: dayIndex 0–4, p: startPeriod 1-based}`
  - `field` řídí barvu kartičky (`FIELD_NAMES`, legenda se staví z oborů v levelu);
    `duration` = výška bloku (počet hodin).
  - **NAŘÍZENÁ PAUZA (oběd)** — `kind:"break"`, `field:"BRK"`, `subject:"Oběd"`, jedna
    kartička na třídu. Chová se jinak než výuka a je to zapečené v enginu, ne v datech:
    - **jen v obědovém okně** — `fits()` ji pustí pouze do `LVL.breakWindow` (výchozí
      `[4,5]`). Je to tvrdé pravidlo jako přesah bloku mimo den, **ne herní cíl** —
      záměrně: jako cíl by u l7 se 16 třídami srazilo šanci náhodného rozvrhu na ~10⁻⁸
      a level by se nedal ověřit vzorkováním.
    - **nemá učitele ani učebnu** (`teacher:"—"`, `room`/`building` = `Jídelna`), takže
      `computeConflicts` i `hasHardConflict` u ní přeskakují konflikt učitele a učebny —
      celá škola smí obědvat naráz. Konflikt třídy platí normálně: pauza zabírá slot.
    - **nepočítá se do výuky** v `max_per_day` (u třídy), ale **počítá se** do
      `class_no_gaps` (oběd vyplňuje mezeru — tak to má být) a nefiguruje v pohledech
      za učitele a učebnu (`teaching()` ji vyfiltruje) ani v legendě.
    - Cíl `lunch_break` (třída má hodinu volnou) tím **ztratil v kampani smysl** — s pauzou
      v rozvrhu by si protiřečily. Typ v enginu zůstává kvůli vlastním levelům.
- **SVĚT KAMPANĚ** (učebny a jejich zaměření skutečné; předměty a jména zástupná —
  lze 1:1 přejmenovat):
  - obory: `IT` Informatika a management, `EP` elektronické počítače, `ELE` slaboproud,
    `STR` strojírenství; třídy `<ročník>.<IT|EP|S|ST>` (field dle oboru třídy —
    **zkratka třídy ≠ `field`**: třída `2.S` má `field:"ELE"`, třída `3.ST` má `field:"STR"`).
  - **budovy = zaměření** (učebna má VŽDY jednu budovu — hlídá `validateLevel`):
    - `Š101` **T3, T5** kmenové · **T9** odborná (Fyzika, Hardware, Management, Ekonomika)
      · **T12, T14** počítačové laby — informatické předměty a teorie (kódy T1–T17).
    - `H59` **E2** elektro dílna · **F3** měřicí lab · **F5** lab mikroprocesorů ·
      **G1** číslicová technika · **G4** kmenová — elektrotechnika (kódy E/F/G 1–7 jsou
      běžné učebny) · **H1** fiktivní vlastní tělocvična (škola reálně žádnou nemá — l1/l2
      ji mají pro zjednodušení). **Tělocvična má vždy vlastní značení**, nikdy kód běžné
      učebny (Jakub, 21. 8. 2026).
    - `H618` **B7** kmenová · **B8** strojní dílna · **B26** rýsovna (CAD) · **C7**
      odborná · **C25** CNC — strojírenství (kódy B/C 7–9 a 25–27).
    - `Externí` **ZŠ 1–3** — půjčené tělocvičny, ad-hoc.
    - Odborný předmět jde do budovy svého zaměření; **všeobecné (Matematika, Angličtina,
      Fyzika) se učí na všech budovách** — obvykle tam, kde třída sedí. Právě tím vznikají
      přechody mezi budovami, na kterých stojí cíl `teacher_transition_gap`.
    - **Tělocvik je páka na obtížnost:** fiktivní vlastní tělocvična H1/H59 u snadných levelů
      (l1, l2), externí ZŠ 1–3 u těžkých (l3–l7). Tomáš je jediný tělocvikář, takže
      učebna tělocviku sama o sobě nic neváže — tvrdý blok drží už učitel.
  - učitelé: Petr+Eva (IT), Jaroslav+Ondřej (EP), Jan (EP/EL, přebíhá Š101↔H59),
    Alena (EL, zkrácený úvazek), Šárka+Martin (ST), Renata (mat, učí na všech budovách
    → přebíhá), Jitka (jazyky), Tomáš (tělocvik), Simona (management/ekonomika).
  - **ZÁSOBÁRNA JMEN** (whitelist od Jakuba, 9. 8. 2026 — jiná křestní jména do hry
    nepatří; 38 unikátů / 61 míst, odpovídá reálné distribuci). Číslo = kolikrát jméno
    v předloze je, tedy kolik různých učitelů může nést; nad rámec toho lze týž základ
    odlišit **familiarizací** (Jan → Honza → Jenda), takže strop unikátních jmen ve hře
    je 61, ne 38. V kampani je zatím obsazeno 12 (níž **tučně**).
    - 6× **Jan**, **Martin** · 4× **Jaroslav** · 3× **Petr** · 2× **Eva**, **Tomáš**,
      Josef, Pavel, Jakub, Václav, Ladislav, Marek
    - 1× **Alena**, **Jitka**, **Ondřej**, **Renata**, **Simona**, **Šárka**, Arnošt,
      Bronislav, Dan, Horst, Iva, Kateřina, Lenka, Luboš, Luděk, Martina, Max, Milan,
      Miloš, Nikola, Oldřich, Petra, Radana, Vladimír, Vladislav, Zdeňka
    - Při výběru drž **rozlišitelnost na kartičce**: vedle sebe nedávej Martin/Martina
      ani Petr/Petra a hlídej podobné tvary (Alena/Lenka).
  - **kampaň l1–l7 postupuje po ročnících:** l1 rozjezd (1.IT) · l2 první ročníky ·
    l3 druhé · l4 třetí · l5 iron-man nad l4 · l6 maturitní (čtvrté) ročníky · l7 „Celá škola".
    **Zobrazené číslo levelu = ročník** (od 21. 8. 2026): „0 · Rozjezd", „1 · První ročníky" …
    „4 · Iron-man", „5 · Maturitní ročníky", „6 · Celá škola"; interní id `l1`–`l7` se NEMĚNÍ
    (visí na nich autosave, ✓, `carryFrom`, odkazy) — číslo v `name` je o jedna nižší než v id
    (**všech 16 tříd** = 4 obory × 4 ročníky, 80 kartiček).
  - měřená obtížnost (výhry mezi náhodnými platnými rozvrhy, 20 000 vzorků; přeměřeno
    9. 8. 2026 po zpřísnění `teacher_transition_gap` — vyžaduje skutečný přechod):
    l1 ≈ 10,3 % · l2 ≈ 4,5 % · l3 ≈ 0,10 % · l4 ≈ 0,62 % · l5 (záměrný oddech) ≈ 3,1 % ·
    l6 ≈ 0,07 % · l7 ≈ 0,24 %. Po změně dat/cílů přeměř („Jde to vůbec vyhrát?" nebo simulace).
    **Měř na ≥ 20 000 vzorcích** — na 4 000 kolísá odhad u l1 o ±1,5 p. b., což stačí
    na falešný dojem, že se obtížnost posunula.
  - **Pozor, co ta metrika je:** podíl výher mezi *náhodnými* rozvrhy. Mezi levely různé
    velikosti není srovnatelná — l7 má stejný řád jako l6, ale 80 kartiček proti 24,
    takže pro člověka je řádově těžší. A cíl, který se člověku plní snadno (Alena jen
    v sudém týdnu), srazí metriku víc než cíl vyžadující koordinaci. Číslo ber jako
    kontrolu „je to vůbec vyhratelné", ne jako míru zážitku. Přesně tohle dělá
    i zpřísněný vynucený přechod (náhodně vznikne v ~8 % u Jana se 3 kartami ve
    2 týdnech, člověk ho postaví jedním záměrným tahem) — proto je od 9. 8. 2026
    metrika l3/l6 pod l4/l7 a křivka po levelech už není monotónní, ač pro člověka
    obtížnost pořád zhruba roste.
- **ÚLOŽIŠTĚ**: adaptér `Store` s feature-detekcí `window.storage` → `localStorage` → null.
  Páteř ukládání je serializační řetězec Base64(JSON), funguje vždy. Import (modál,
  autosave i URL hash `#p=<řetězec>`) validuje placementy přes `validPlacement()`
  (rozsahy w/d/p + `fits`) a vynechané kartičky spočítá do hlášky (`IMPORT_NOTE`);
  hash se po načtení maže (`history.replaceState`) a neautosavuje. Dokončené levely
  drží set `DONE` (klíč `done`, ✓ v selectu). Motiv nastavuje synchronní skript
  v `<head>` (localStorage / `prefers-color-scheme`, žádný flash), async `Store`
  (artefakt) ho jen doladí.
- **STAV**: `LVL` (aktuální level) + `S = {placements, selected, view, active, pickedFrom}`.
  - `view` ∈ `class | teacher | room` = dimenze pohledu na rozvrh; `active` = vybraná
    entita v té dimenzi. **Hraje se ve všech třech pohledech** (od 21. 8. 2026; dřív byly
    Učitelé/Učebny jen pro čtení): kartičku lze položit jen do rozvrhu, který je vidět, a jen
    když do něj patří — `belongsHere(c)` = `entityOf(c,S.view)===S.active`. Zvednutí kartičky
    (`pickUp`/`startDrag`) přepne na její rozvrh **ve stejném pohledu** (`focusHome`: v Učitelích
    u Petra zůstáváš u učitelů, ne skok do Tříd — Jakub); jen pauza (nemá učitele ani učebnu)
    skočí do Tříd. Helpery: `entitiesOf(view)`, `entityOf(card,view)`, `homeOf(card)`.
  - `pickedFrom` = původní pozice zvednuté kartičky; Esc, zrušené tažení i výběr jiné
    karty ji přes `restoreLifted()` vrací na místo. Přepnutí záložky/pohledu na rozvrh, kam
    vybraná kartička nepatří, ruší výběr (`dropSelection()`) a drop zóny svítí jen v jejím
    rozvrhu — jinak šla kartička položit „naslepo“ do rozvrhu, který nebyl vidět (oprava 9. 8. 2026).
  - `assisted` = desku poskládal řešič nebo import → výhra ukáže jiný banner a NEdá ✓
    (fér ✓). Nastavují randomFill/importStr, nulují ruční akce, undo/redo a loadLevel;
    do exportu/autosave se propisuje jako `a:1`, takže přežije i reload. **Nuluje jen
    skutečný tah** — vrácení zvednuté kartičky na totéž místo (které není ani krokem
    historie) příznak nechává; jinak byl ✓ zadarmo (nález 21. 8. 2026).
  - `locked` = Set id kartiček zamčených levelem (`given`/`carryFrom`, aplikuje
    `applyGiven()` v `loadLevel`). Zamčené nejde zvednout/táhnout/odebrat, `clearAll`
    je zachová, autosave/import je nepřepisuje, řešič i kontrola řešitelnosti je berou
    jako pevný základ. V rozvrhu mají 🔒. **Přenáší se jen dohraný zdroj** — `carryReady()`
    chce autosave kompletní, bez tvrdého konfliktu a se splněnými cíli zdrojového levelu;
    jinak zůstává vzorové `given` s hláškou (21. 8. 2026: rozehraný l4 s červeným konfliktem
    se zamykal do l5 jako neopravitelný konflikt a level tiše nešel vyhrát; částečný l4
    zase l5 zlehčil). Přenos, který by sám o sobě znemožnil některý cíl (kontroluje
    `goalDeadWithLocked` — např. zamčené 3 hodiny v den s cílem „max 2 denně“) nebo měl
    v cílovém levelu konflikt, se také nepoužije: základ spadne na vzorové `given`
    a hráč dostane hlášku s důvodem (oprava 9. 8. 2026 — vlastní výherní pozice z l4
    uměla udělat l5 neřešitelný).
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
  stropem uzlů **a časovým boxem** (1,5 s; v chytrém režimu 150 ms na pokus — `FILL_TIMED_OUT`
  přidá do hlášky „řešič vyčerpal časový limit“), delší bloky se zkoušejí dřív; bere ručně
  umístěné i zamčené jako pevné, respektuje jen tvrdé požadavky. Samotný strop uzlů nestačil:
  neřešitelný level z editoru (31 hodin do 30 slotů) zamrazil UI na ~68 s (21. 8. 2026).
  Se `smart=true` (checkbox v modálu „Náhodně“) běží multi-restart ~700 ms a vybere
  výsledek s nejvíc splněnými měkkými cíli (`goalsMetIn`). Když se vše nevejde, `blockNote()` nahlásí dominantní tvrdý blok
  (učitel/učebna/třída) u nevešlých kartiček.
- **KONTROLA ŘEŠITELNOSTI**: `checkSolvableAsync(levelId, samples, onProgress)` —
  vzorkuje náhodné kompletní rozvrhy chunkovaně (časové boxy ~25 ms, UI nezamrzá;
  globální `LVL`/`S` se prohazují jen uvnitř synchronního chunku s try/finally).
  0 výher ⇒ level je extrémně těžký / neřešitelný. Tlačítko **„Jde to vůbec vyhrát?"** (dřív
  „Řešitelnost?" — moc úsečné, Jakub 21. 8. 2026; v editoru „Jde to vyhrát?") otevírá
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
- **MODÁLY**: všechny jdou přes `openOverlay(modal, onClose)` — klik mimo i Esc zavře
  (capture listener předběhne globální Esc), fokus skočí na první tlačítko v modálu
  a po zavření se vrátí; `ov.close()` je jediná cesta ven, `onClose` uklízí (zrušení
  běžícího testu řešitelnosti). `.modal` má `max-height` + `overflow:auto`, ať na
  telefonu neutečou tlačítka za okraj. (21. 8. 2026: fokus zůstával pod overlayem
  a Enter otevřel druhý modál přes první.)
- **LAYOUT A DVĚ ZÁSADY (Jakub, 21. 8. 2026):** (1) **stránka nikdy nescrolluje do stran** —
  rozvrh má sloupce `minmax(0,1fr)`, paleta zalamuje, `html,body{overflow-x:hidden}` je jen pojistka;
  (2) **ovládání dole je vždy na obrazovce** — lišta je fixní a rezervu pod obsahem počítá
  `updateDock()` z její skutečné výšky (`--tb-h`, `--dock-h`), ne media queries.
  Ovládání = lišta; požadavky jsou obsah a od 25. 8. 2026 stojí nahoře v toku ve všech šířkách. Hlavička: titul | level (`<select class="lvlsel">` vínově jako odkaz) +
  počítadlo `#placedBadge` · popis levelu pod tím · vpravo Editor, Uložit a ikonová dvojice ⓘ ◐
  (`.btngroup`). **Úzký displej (< 820 px, stejná hranice jako překlopení rozvrhu):** hlavička na
  dva řádky, Editor a Uložit v nabídce „Více“ (`#moreMenu`), záložky entit jako `<select
  class="tabsel">`, paleta jen aktivní třídy zalomená do řádků (ostatní třídy počtem), kartičky
  v rozvrhu bez × (vrací se tažením nebo ťuknutím na zásobník — `returnToPalette()`, platí i na
  desktopu); lišta na jeden řádek ikon + stavový řádek.
  **Rozpočet lišty na mobilu (změřeno 25. 8. 2026, iframe po šířkách 320–819).** Všechna tři
  významová tlačítka mají krátký popisek (`Smazat` / `Náhodně` / `Jde to?`) — rovnocenná jako
  na desktopu (Jakub). Vejdou se na jeden řádek jen tak tak: **`#checkBtn` smí být max ~73 px,
  tedy 7 znaků.** „Jde to?“ (73 px) projde, „Vyhraju?“ (83 px) už ne — proto se `Jde to vůbec?`
  (116 px) po zkušenosti vrátilo na `Jde to?`. Zkracovat `Smazat` ani `Náhodně` nemá smysl,
  na zalomení to nemá vliv.
  - **Od 390 px výš** je lišta jednořádková (87 px). **Na 360 a 375 px je dvouřádková (137 px)
    a nespraví to nic** — ověřeno i s úplně odebraným popiskem `Smazat`. Pod 320 px 154 px.
  - Zkoušené a zamítnuté: užší mezery a menší odsazení ušetří ~14 px z potřebných ~78;
    „undo/redo na vlastní řádek“ dá hezčí `[↶↷]` / `[tři tlačítka]`, ale až od 412 px —
    na 360–390 px z toho udělá **tři** řádky (187 px).
  - Nic se neztrácí ani při zalomení: rezerva pod obsahem se počítá z reálné výšky lišty.
  **Pozor při měření:** jeden iframe pro víc variant zašpiní stav (zbylý `<style>`, jiná šířka)
  a hlásí přetečení textu, které na čistém načtení není. Měř na čerstvém iframu.
  V liště nemá žádné tlačítko zvýraznění —
  „Náhodně rozmístit" je pomůcka, ne hlavní akce (Jakub, 21. 8. 2026). Překryvy jdou transponovaně **pod sebe do bloku** (`.lanegrp`, řádky hodin se natáhnou)
  — pruhy vedle sebe se v 58px sloupci rozsypaly na písmena.
- **PRUH POŽADAVKŮ (`#colGoals`) — ve všech šířkách stejný** (Jakub, 25. 8. 2026; postupně:
  sloupec vpravo → fixní proužek nad lištou na mobilu → pruh nahoře všude). Není sloupec ani
  překryv, ale **sbalovací pruh přes celou šířku nad obsahem**: `order:-1` + `flex:0 0 100%`
  v zalomeném `.wrap`, pod ním Kartičky | Rozvrh. Sklapnutý drží jen souhrn `#goalsSummary`
  („2 / 5 …“ + šipka) na jednom řádku (~38 px), rozbalený ~440 px.
  - **Výchozí stav dělá `setGoalsDefault()`** (volá ho `loadLevel`): **rozevřený jen v prvním
    levelu kampaně** (`LVL===LEVELS[0]`, tedy `l1`) — mobil i desktop stejně; tam hráč požadavky
    vidět musí. Ve vyšších levelech i ve vlastních **sklapnutý**: hráč už ví, co hraje, a pruh
    nemá brát místo rozvrhu (Jakub, 25. 8. 2026; dřív rozhodovala šířka — mobil rozevřený,
    desktop sklapnutý). `LEVELS[0]` je vždy `l1`, vlastní levely z editoru jdou na konec
    (`LEVELS.push`). Ruční ťuknutí na `#goalsHead` to přebije až do dalšího levelu —
    **překlopení orientace stav už nepřenastavuje**, když na šířce nezáleží.
  - **Výhra pruh rozbalí a doscrolluje na banner** (`winBox.scrollIntoView`, jen při přechodu
    do výhry) — nahoře není pod rukou jako dřív dole u lišty.
  - **Přilepená je VŽDY jen hlavička se souhrnem** (`position:sticky;top:0`, otevřeno i zavřeno,
    ve všech šířkách), s plným pozadím a stínem — pod ní projíždí zásobník i rozvrh.
  - **Tělo se rozvine jako PŘEKRYV pod hlavičkou**, ne v toku: `#colGoals.open .col-body` je
    `position:absolute;top:100%` s vlastním `max-height:60vh` a scrollem. Proto se obsah pod
    pruhem nehne ani o pixel a zavření vrátí hráči přesně tentýž obraz (Jakub, 25. 8. 2026 —
    varianta B z návrhového plátna). **Předchozí pokus rozbalovat v toku byl chyba:** panel se
    odlepil, vystřelil nad viewport a stránka pod rukou uskočila o 367 px — z rozbaleného
    obsahu bylo vidět 0 px. Zavírá i klik mimo `#colGoals` (jako u nabídky „Více“); `goalsHead`
    proto `stopPropagation()`.
  - **Výherní banner je na konci těla, které se scrolluje samo** — po `add("open")` k němu
    `gb.scrollTo({top:scrollHeight})`, jinak zůstane pod okrajem překryvu. **Odklad je nutný
    a `requestAnimationFrame` na něj NESTAČÍ** (běží před přepočtem layoutu, scroll tiše neudělá
    nic) — musí to být `setTimeout(…,0)`. Změřeno 25. 8. 2026.
  - **Zásobník se lepí POD hlavičku, ne k okraji okna** (`#colPalette{top:calc(var(--goals-h) + 14px)}`,
    ≥ 1101 px) — jinak mu pruh překryje hlavičku „KARTIČKY / n zbývá“. `--goals-h` počítá
    `updateDock()` z reálné výšky pruhu; ta je **stejná otevřeno i zavřeno**, protože tělo je
    mimo tok. Stejnou proměnnou odečítá i `max-height` zásobníku. Zásada z 21. 8. 2026
    („zásobník i požadavky přilepené a scrollují uvnitř“) tím platí dál — požadavky drží místo
    vnitřního scrollu sklapnutí do souhrnu.
  - Do rezervy `--dock-h` se pruh **nepočítá** — ta je jen z výšky lišty.
- **RENDER**: `render()` → `renderPalette / renderTabs / renderSchedule / renderGoals`
  (+ `updateDock()`). `renderPalette` skládá kartičky do **skupin podle entity aktuálního pohledu**
  (`.pgroup`: v Třídách čtvereček oboru + třída, v Učitelích/Učebnách jméno/učebna; počet; skupina
  zobrazeného rozvrhu první a plná, ostatní tlumené `.dim` — klik na tlumenou přepne rozvrh na její
  skupinu; pauza mimo Třídy ve zvláštní tlumené skupině), plní počítadlo `#palCount`
  a legendu oborů v patičce palety (`#legend` — patří k barvám, které vysvětluje).
  `renderSchedule` plní **dovětek v hlavičce sekce Rozvrh** (`#schedOrient`): „na šířku“ na
  desktopu, „na výšku“ na úzkém displeji — týž tvar i styl (`.muted`) jako počet zbývajících
  kartiček u zásobníku, ať je orientace mřížky pojmenovaná a ne jen viditelná (Jakub, 25. 8. 2026).
  `renderGoals`
  kreslí „2 / 5 požadavků“ + segmentový ukazatel (`.segbar`, dílek = požadavek), konfliktní řádek
  s důvodem (`conflictSummary()`: první obsazená buňka → „Út 2. h — 1.IT má dvě hodiny naráz
  (Matematika, Web)“), souhrn do hlavičky panelu, počítadlo v hlavičce a barvu tečky ve stavovém
  řádku (`setStatusDot`). **Výherní banner nese tlačítko „Dál → <další level>“** (`.win .next`,
  jediné zvýrazněné tlačítko ve hře — postup dál; u posledního levelu text „kampaň dohraná“),
  jinak hráč hledal přepínač levelů (Jakub, 21. 8. 2026). Legenda oborů = zkratka jako na čipu
  třídy + plný název (`FIELD_NAMES`: „IT — informatika a management“ …), pod sebou.
  `renderTabs` kreslí přepínač pohledu (segmentový ovladač `.seg` — jeden rámeček, verzálky,
  aktivní segment vystouplý) + záložky entit (samostatné pilulky s plnou barvou) — dvě úrovně,
  dvě tvarosloví, ať se nepletou (Jakub, 21. 8. 2026); `renderSchedule` umí všechny
  tři pohledy a překrývající se kartičky řeší „pruhy" (lanes), aby se neschovaly.
  Vícehodinové bloky mají zářez na každé hranici hodiny (`.tick`, kolmo na osu hodin
  dle orientace) a v rozvrhu štítek délky `.durb` („2h“) vpravo dole — dvojhodinovka
  musí být poznat na první pohled i uvnitř pruhu. **Popisek kartičky** je na šířku ve dvou
  sloupcích (`.info`: předmět | učebna, učitel/třída | budova — levý dolní je to, podle
  čeho pohled NEfiltruje; v pohledu učeben je vpravo nahoře učitel), na úzkém displeji
  pod sebou (předmět, „kdo · kde“, budova; budova se pod 600 px skrývá). Tři řádky pod
  sebou se do buňky, natož do pruhu, nevešly (Jakub, 21. 8. 2026). Hodiny a dny jsou tlumené
  štítky (`.gh`/`.gp`), prázdná buňka má jen tenký plný rámeček — čárkovaný dostane až drop zóna,
  a ta je **zelená** (`--ok`, „sem smíš“): vínová/červená znamená ve hře konflikt a zóny v ní
  vypadaly jako zákaz (Jakub, 21. 8. 2026). Totéž platí pro cíl „zpět do zásobníku“.
  **Paleta drží
  stejnou mřížku** (předmět | učebna, čip třídy + učitel | budova; pauza: „4.–5. h“ a „nařízená
  pauza“ vpravo, čip vlevo) a **délka kartičky = ŠÍŘKA podle hodin** (`--dur`: 1h 165 px,
  2h 222 px, 3h 279 px, výška pořád stejná, svislá čárkovaná hranice hodiny) — bez štítku
  „2h“. Není to proporční (1h by se s popiskem do půlky nevešla), jen zřetelně kratší;
  dvojnásobná VÝŠKA byla slepá ulička (délka bloku je v rozvrhu vodorovná). Sloupec palety
  má 250 px, ať se 2h kartička vejde. Položená kartička má sytou výplň (30 % barvy
  oboru), rámeček v barvě oboru a stín, aby se nepletla s volnou buňkou; hranice hodin
  jsou jen zářezy na okrajích (`.tick` s gradientem), ne linka přes blok.
  Transponovaná mřížka má sloupce dnů `minmax(0,1fr)` (vejít se do šířky, viz zásady výš);
  čitelnost drží `hyphens:auto` (html má `lang="cs"`), menší písmo a užší odsazení pod 820 px.
  Pozor na pořadí CSS: přepisy pro úzký displej jsou v **posledním** `@media` bloku na konci
  stylů — dřívější blok obecná pravidla nepřebil (chyba 21. 8. 2026).
  **Orientace** dle šířky (`isHorizontal()`, breakpoint 820 px): desktop „na šířku"
  (dny = řádky, hodiny = sloupce), úzký displej transponovaně (hodiny = řádky, dolů).
  Pruhy se proto skládají kolmo na osu hodin (vodorovně vs. svisle). Při překlopení
  orientace překresluje listener na `resize`. Překresluje se celé (jednoduchost nad
  výkonem; při ~16 kartách to stačí).
- **el(tag, props, ...kids)** — mini-helper na DOM, bezpečný vůči diakritice. Styl
  z objektu: custom property (`--f`, `--dur`) jde přes `style.setProperty` — `Object.assign`
  ji tiše zahodí, a tak barva oboru na kartičkách do 21. 8. 2026 vůbec nefungovala.

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
| `teacher_transition_gap` | `teacher` | aspoň jednou přejde mezi budovami a po KAŽDÉM přechodu má volnou hodinu (9. 8. 2026: bez povinného přechodu byl cíl zadarmo — stačilo hodiny rozházet do různých dnů) |
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
  puštěním mimo rozvrh a Escapem (karta se vrátí na původní místo), s vybranou kartou
  přepni záložku jiné třídy (výběr se musí zrušit a drop zóny zhasnout — kartička nesmí
  jít položit do rozvrhu cizí třídy), vyvolej konflikt
  (musí zčervenat + tooltip), překryj dvě karty stejné třídy (musí být vidět vedle
  sebe / nad sebou, ne zmizet), zkontroluj dvojhodinovku (čárka na hranici hodiny +
  štítek „2h“, v obou orientacích), přepni pohled Třídy/Učitelé/Učebny a polož kartičku
  i v pohledu Učitelé (zvednutí nesmí přepnout do Tříd; tlumená kartička cizího učitele
  přepne rozvrh na něj), zúži okno pod ~820 px (rozvrh se překlopí na hodiny dolů), spusť
  „Náhodně" (i s checkboxem měkkých cílů) a „Jde to vůbec vyhrát?" (modál s průběhem, jde
  zrušit), vyzkoušej ↶ Zpět / ↷ Znovu (i Ctrl+Z/Y), ulož/načti řetězec i pojmenovaný
  slot, zkopíruj odkaz a otevři ho (načte pozici, hash zmizí z URL), přepni motiv
  (tmavý nesmí při načtení bliknout světlým). U libovolného modálu stiskni Esc (zavře)
  a Enter hned po otevření (jen zavře/zruší, nesmí otevřít druhý). V l5 ověř zamčené
  kartičky (🔒 — nejde zvednout, přežijí Smazat) a že se rozehraný/konfliktní l4 nepřenese
  (hláška, základ je vzorový) a v Editoru ulož/smaž zkušební level (objeví se s `*`
  v přepínači) a pošli level odkazem („Odkaz na level" → otevřít v novém tabu).
  Ověř fér ✓ (výhra přes „Náhodně + zkusit cíle" ukáže banner „poskládal řešič" bez ✓;
  ruční tah pak povýší na pravou výhru) a otevři ⓘ O hře. Projdi světlý i tmavý motiv
  a úzké okno (< 820 px): stránka nesmí scrollovat do stran (`scrollWidth ≤ innerWidth`, i v l7),
  fixní lišta nesmí zakrývat spodek obsahu (rezerva z `--dock-h`), paleta
  ukazuje jen aktivní třídu, kartička zvednutá z rozvrhu se vrátí ťuknutím na zásobník, překryv
  dvou kartiček je pod sebou a čitelný, „Více“ otevře Editor/Uložit, pruh požadavků je na mobilu
  v prvním levelu rovnou rozevřený (ve vyšších sklapnutý) a ťuknutím se přepne, hlavička Rozvrhu
  hlásí „na výšku“ (na desktopu „na šířku“), a modál Editoru se vejde do výšky okna. Žádné chyby v konzoli.
  Dotyk otestuj i na reálném zařízení, ne jen v emulaci.
- **Po úpravě dat/cílů levelu** spusť „Jde to vůbec vyhrát?" (nebo `checkSolvable("lX")`
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

Opravy z hraní 9. 8. 2026: kartičku už nejde položit „naslepo“ přes záložku cizí třídy
(přepnutí záložky ruší výběr + drop zóny jen ve vlastní třídě), `teacher_transition_gap`
vyžaduje aspoň jeden skutečný přechod (dřív šel splnit rozházením hodin do různých dnů —
po změně přeměřena obtížnost l3/l4/l6/l7), dvojhodinovky dostaly čárky na hranicích hodin
+ štítek „2h“ a carryFrom přenos, který by znemožnil cíl levelu, padá na vzorové `given`
s hláškou (`goalDeadWithLocked`).

Opravy z testu 21. 8. 2026 (detailní průchod desktop + emulovaný mobil): barva oboru se na
kartičky nikdy nepropsala (`el()` a custom property), iron-man přenáší jen dohraný zdroj
(`carryReady`), řešič má časový box, fér ✓ už nejde obejít zvednutím a vrácením kartičky,
modály přes `openOverlay` (Esc, fokus, výška na mobilu), import hlásí vynechané kartičky,
transponovaný rozvrh se vejde do šířky. Designově: dvousloupcový popisek
kartičky (i v paletě), šířka kartičky v paletě podle hodin (výška stejná), sytější výplň
a zářezy místo dělicí linky, segmentový přepínač pohledu. Doladění z návrhového plátna (týž den,
všech šest bodů — viz LAYOUT výš): hlavička s levelem v titulku a počítadlem, paleta po třídách,
tišší mřížka, „2 / 5“ + segmenty + konflikt s důvodem, lišta s ikonami a tečkou, mobilní chrome
(Více, select tříd, paleta aktivní třídy, fixní proužek požadavků — od 25. 8. 2026 pruh nahoře
ve všech šířkách, lišta na jeden řádek).
Zůstává: klávesnicové ovládání.

Obsah je hotový „naslepo": kampaň 7 levelů (l1–l7, viz SVĚT KAMPANĚ výš), obtížnost
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
   - ~~12 křestních jmen učitelů~~ — **hotovo 9. 8. 2026**: jména jdou ze skutečné
     zásobárny, viz „Svět kampaně" výš. Role: Petr + Eva (IT), Jaroslav + Ondřej (EP),
     Jan (EP/EL, přebíhá Š101↔H59), Alena (EL, zkrácený úvazek — jen sudý týden),
     Šárka + Martin (ST), Renata (matematika, učí na všech budovách → přebíhá),
     Jitka (jazyky), Tomáš (tělocvik, jediný tělocvikář), Simona (management/ekonomika).
   - **19 názvů předmětů**: všeobecné — Matematika, Angličtina, Tělocvik, Fyzika;
     IT — Programování, Databáze, Web, Management, Ekonomika; EP — Číslicová technika,
     Mikroprocesory, Hardware; EL — Elektronika, Měření; ST — CAD, CNC, Praxe,
     Stavba strojů, Technologie.
   - ~~kódy učeben~~ — **hotovo 9. 8. 2026**: učebny i zaměření budov jsou skutečné,
     viz „Svět kampaně" výš. Budova OPMB ze zadání zatím nepoužita.
   - ~~označení tříd~~ — **hotovo 9. 8. 2026**: schéma `<ročník>.<obor>` i zkratky oborů
     jsou skutečné (`IT`, `EP`, `S` slaboproud, `ST`) a v kampani je všech 16 tříd.
2. Ruční otestování dotyku na reálném mobilu/tabletu (hold ~200 ms = tažení,
   švih = scroll) — na nasazené URL.

Pozn.: „vázanost předmětu na budovu" z původního zadání není herní cíl — budova je
pevná vlastnost kartičky, řeší se návrhem dat levelu.
