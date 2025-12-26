---
date: 2025-12-19
evaluator: Claude Sonnet 4.5
version: 01
status: draft
---

# Kritikai értékelés az AI-PDS elképzelésről

## ✅ **Erősségek**

### 1. Valós problémát céloz meg
- Az AI-val való együttműködés valóban strukturálatlan ma
- A "context management" az egyik legnagyobb kihívás AI-assisted fejlesztésnél
- A traceability hiánya gyakori probléma projekteknél

### 2. Gondosan kidolgozott rendszer
- Világos életciklus modell (Project + Release)
- Átgondolt szétválasztás: Handbook (stabil) vs. Artefacts (dinamikus)
- Konzisztens státusz kezelés (draft → to review → approved → living)

### 3. Gyakorlati megközelítés
- Van konkrét példa implementáció
- Magyar nyelvű "overall-draft.md" nagyon hasznos
- PMP + MWV (mockup + videó) megközelítés UX-driven és érthető

### 4. Dokumentációs fegyelem
- "Ha változik a valóság, előbb a doksi változzon" - helyes elv
- Minden visszavezethető egy konkrét dokumentumra

---

## ⚠️ **Kritikus pontok és kockázatok**

### 1. HATALMAS DOKUMENTÁCIÓS OVERHEAD

```
Kötelező (M) fájlok száma: ~25-30+
Feltételes (C) fájlok: ~15-20
Opcionális (O) fájlok: további ~10
```

- Egy 3-5 fős csapatnak **40-60 dokumentumot** kellene karbantartani
- Ez ellentmond a "kis csapatok" célnak
- Paradoxon: az AI-nak azért kell ez a struktúra, hogy segítsen, de a struktúra karbantartása önmagában full-time munkát igényel

### 2. Szinkronizációs probléma (Documentation Drift)

```
Code ──────────────> változik gyorsan
              │
              ↓
Documentation ──────> lemarad, elavul
```

- A kód és a dokumentáció **garantáltan** el fog térni
- Nincs automatikus érvényesítés, hogy sync-ben vannak-e
- Reality: fejlesztők kódolnak, aztán "majd később" frissítik a doksikat (soha)

### 3. AI képességek túlbecslése

- Feltételezés: az AI pontosan követi a specs-et ✗
- Valóság: az AI gyakran "kreatívan értelmez", kitalál dolgokat
- A hosszú context window-k (200k token) NEM jelentik azt, hogy az AI mindent perfektül értelmez
- Az AI hallucináció problémája nem oldódik meg dokumentációval

### 4. Méretezhetőség hiánya

**Kis projekt (1-2 hét):**
- A setup idő > fejlesztési idő
- Overkill

**Nagy projekt (több év, sok dev):**
- Ki review-lja a 60+ dokumentumot?
- Ki tartja karban?
- Hogyan verziózod (git? release branch-ek?)

### 5. Rugalmatlanság

- A merev életciklus (Functional Spec → Domain Model → Requirements → Architecture → Dev) nem illik minden projektre
- Startup/agilis környezetben túl waterfall-szerű
- Mi van, ha közben változik a requirement? Vissza az elejére?

### 6. Nincs automatizálás

- Nincs tooling a dokumentációk validálásához
- Nincs CI check, hogy minden mandatory file létezik-e
- Nincs link checker (törött referenciák)
- Nincs status flow validátor

### 7. Gyakorlati bizonyíték hiánya

- Nincs case study, hogy ez működik
- Nincs adat, hogy mennyi időt vesz igénybe
- Nincs összehasonlítás más megközelítésekkel
- Nincs user feedback valós projektekből

### 8. DRY (Don't Repeat Yourself) megsértése

```
User Story (requirements/user-stories.md)
     ↓
Feature Definition (features/feature-definition-ticket.md)
     ↓
Acceptance Criteria (requirements/acceptance-criteria.md)
     ↓
Test Case (qa/test-case.md)
```

- Ugyanaz az információ 4 helyen
- Ha változik egy requirement → 4 fájlt kell frissíteni
- Hiba forrás

### 9. Vendor/Tool lock-in veszélye

- Túlságosan Claude/GPT-centrikus?
- Mi van, ha jön egy új AI paradigma?
- A YAML frontmatter formátum mennyire átjárható?

---

## 🤔 **Kérdések, amiket érdemes megválaszolni**

### 1. ROI (Return on Investment):
- Mennyi idő a setup? (becsült: 2-5 nap)
- Mennyi idő a maintenance? (becsült: 20-30% overhead)
- Mennyi időt spórol az AI használata? (???)

### 2. Összehasonlítás:
- Mi a különbség egy jól vezetett Confluence/Notion wiki-hez képest?
- Miért jobb ez, mint egy jó README.md + ADR + issue tracker kombináció?

### 3. Gyakorlati adoptáció:
- Egy 5 fős csapat reálisan fogja használni ezt?
- Mi történik, ha valaki nem frissíti a doksit?

### 4. AI evolúció:
- Mi lesz 1-2 év múlva, amikor az AI-k még okosabbak lesznek?
- Lehet, hogy akkor már nem lesz szükség ennyi struktúrára?

---

## 💡 **Javaslatok / Továbbfejlesztési irányok**

### 1. MVP megközelítés
- Csökkentsd a mandatory fájlok számát 5-10-re
- "AI-PDS Lite" verzió kezdőknek

### 2. Automatizálás
- GitHub Action / pre-commit hook a validáláshoz
- Template generator a kötelező fájlokhoz
- Link checker és status validator

### 3. Gyakorlati validáció
- Próbáld ki 2-3 valós projekten
- Mérj: setup idő, maintenance overhead, AI hatékonyság
- Iterálj a tapasztalatok alapján

### 4. Integráció más eszközökkel
- Jira/Trello integráció
- OpenAPI spec generálás
- Code → Documentation sync tooling

### 5. Fokozatos bevezetés
- Ne mindent egyszerre
- Kezdd pl. Domain Modelling + Requirements-szel
- Aztán bővítsd

---

## 📊 **Összegzés**

| Szempont | Értékelés | Magyarázat |
|----------|-----------|------------|
| **Koncepció** | 8/10 | Ambiciózus, jól átgondolt vízió |
| **Gyakorlatiasság** | 4/10 | Túl sok overhead kis csapatoknak |
| **Komplexitás** | 3/10 | Túlbonyolított, nehezen áttekinthető |
| **Dokumentáció** | 9/10 | Részletes, átgondolt spec |
| **Validáció** | 2/10 | Nincs bizonyíték, hogy működik |
| **Tooling** | 1/10 | Nincs automatizálás |
| **Összesen** | **5/10** | Jó ötlet, de túl ambiciózus az v1-re |

---

## 🎯 **Végső ítélet**

Az AI-PDS egy **intellektuálisan impozáns, elméleti szempontból logikus** rendszer, ami **valós problémát** próbál megoldani.

**DE:** A jelenlegi formájában **túlkomplex** és **gyakorlatban nehezen alkalmazható** kis csapatok számára - pontosan arra a célközönségre, akinek szánták.

**Hasonlat:** Olyan, mint egy Formula-1 autó egy családi hétvégi kiránduláshoz. Gyönyörű, high-tech, de a használati eset nem passzol hozzá.

### Mit csinálnék másképp:

1. Kezdd egy 10 fájlos "AI-PDS Core" változattal
2. Építs hozzá validációs tooling-ot
3. Próbáld ki 3 valós projekten
4. Mérj: időt, hatékonyságot, elégedettséget
5. Iterálj a visszajelzések alapján
6. **Csak utána** építsd ki a full specifikációt

A jelenlegi spec egy **kiváló referencia architektúra**, de **nem használatra kész rendszer**.

---

## 🔍 **Részletes elemzés témakörönként**

### Dokumentációs struktúra

**Pozitívumok:**
- Világos kategorizálás (collaboration, development, releases, system-design)
- Handbook vs Artefacts szétválasztás logikus
- YAML frontmatter konzisztens metaadat kezelést biztosít

**Problémák:**
- A hierarchia mélysége (3-4 szint) nehezen navigálható
- A kötelező fájlok száma (25-30+) elrettentő
- Nem világos, hogy egy új csapattag honnan kezdje

### Életciklus modell

**Pozitívumok:**
- A Project vs Release lifecycle szétválasztás értelmes
- A fázisok logikusan követik egymást
- A dokumentum státuszok (draft → to review → approved → living) jól átgondoltak

**Problémák:**
- Túl merev, waterfall-szerű
- Nincs támogatás iteratív/agilis munkafolyamatokhoz
- Mi történik, ha egy későbbi fázisban kiderül, hogy változtatni kell a korábbi döntéseken?

### AI integráció

**Pozitívumok:**
- Az AI-nak strukturált inputot biztosít
- Konzisztens formátum (markdown + YAML) könnyen parse-olható
- Traceability segít az AI-nak megérteni a kontextust

**Problémák:**
- Nincs garancia, hogy az AI betartja a szabályokat
- A context window limit továbbra is probléma (60+ fájl > 200k token)
- Nincs feedback loop: hogyan javítja az AI a dokumentációt?

### Összehasonlítás más megközelítésekkel

**vs. C4 Model (Architecture):**
- C4: 4 szintű vizualizáció (Context, Container, Component, Code)
- AI-PDS: szöveges, markdown-alapú
- Előny: könnyebben generálható AI-val
- Hátrány: nehezebben áttekinthető

**vs. Arc42 (Architecture Documentation):**
- Arc42: 12 fejezetes sablon
- AI-PDS: ~40-60 fájl
- Hasonlóság: mindkettő struktúrált
- Különbség: Arc42 egy dokumentum, AI-PDS sok kis fájl

**vs. Living Documentation (Cucumber, BDD):**
- Living Doc: kód = dokumentáció
- AI-PDS: dokumentáció külön a kódtól
- Kérdés: melyik a "single source of truth"?

---

## 📝 **Konkrét példák a problémákra**

### Példa 1: Új feature hozzáadása

**Jelenlegi AI-PDS folyamat:**
1. Frissítsd `requirements/user-stories.md` - hozzáadod az új user storyt
2. Frissítsd `requirements/acceptance-criteria.md` - megfogalmazod az acceptance criteriat
3. Frissítsd `domain-modelling/domain-model.md` - új entitás/fogalom esetén
4. Frissítsd `domain-modelling/domain-vocabulary.md` - új fogalom esetén
5. Hozz létre `features/new-feature-XYZ.md` - feature definition
6. Frissítsd `architecture/sequence-design.md` - ha új interakció kell
7. Frissítsd `architecture/interface-contracts.md` - ha új API endpoint kell
8. Hozz létre `qa/test-case-XYZ.md` - tesztesetek
9. Commitolj mindent git-be
10. **AZTÁN** kezdd el a kódolást

**Időigény:** 2-4 óra dokumentáció, mielőtt egy sort is kódolnál.

**Egyszerűbb alternatíva:**
1. Issue tracker-ben létrehozol egy ticket-et leírással és acceptance criteria-val
2. Kódolsz
3. PR description-ben összefoglalod
4. Merge után az ADR-t frissíted, ha architekturális döntés született

**Időigény:** 15-30 perc dokumentáció.

### Példa 2: Hotfix production-ben

**Production probléma:**
- 16:00 - User jelentette: a checkout nem működik
- 16:05 - Debuggolás, root cause megtalálva
- 16:15 - Fix kész, tesztelve
- 16:20 - **VÁRTUNK!** Frissíteni kell:
  - `domain-modelling/domain-open-questions.md` (ha domain issue volt)
  - `architecture/nonfunctional-design-notes.md` (ha performance issue volt)
  - `qa/defect-report.md` (defect dokumentálás)
  - `releases/release-notes.md` (hotfix note)
  - Git commit message (formázva az AI-PDS szerint)
- 16:40 - Dokumentáció kész
- 16:45 - **DEPLOY**
- 17:00 - Post-release monitoring dokumentálás

**Probléma:** 45 perc documentation overhead egy 5 perces fix esetén.

### Példa 3: Requirement változás mid-sprint

**Scenario:**
- Sprint közepén a PO megváltoztatja a requirement-et
- "A gomb legyen zöld, nem kék"

**AI-PDS szerint:**
1. Frissítsd `requirements/user-stories.md` (status: living → draft)
2. Frissítsd `requirements/acceptance-criteria.md`
3. Frissítsd `features/button-feature.md`
4. Frissítsd `qa/test-case-button.md`
5. Review process (minden fájlra: draft → to review → approved)
6. Commitolj
7. Módosítsd a kódot
8. Újra commitolj

**Reality check:**
- A dev csak megváltoztatja a CSS-t (`background: green`)
- Commit: "Changed button color to green per PO request"
- Kész.

**A dokumentáció?** Valószínűleg nem lesz frissítve, vagy csak napokkal később.

---

## 🎓 **Tanulságok más iparágakból**

### 1. Aerospace (DO-178C - Software for Airborne Systems)

**Hasonlóság:**
- Szigorú dokumentációs követelmények
- Teljes traceability
- Review és approval folyamatok

**Különbség:**
- Ott kritikus a biztonság (életveszély)
- Óriási csapatok (100+ fő)
- Unlimited budget

**Tanulság:** Az AI-PDS hasonló szigort akar, de kis csapatoknak és gyors iterációhoz - nem kompatibilis.

### 2. ISO 9001 (Quality Management)

**Hasonlóság:**
- "Document what you do, do what you document"
- Audit trail
- Continuous improvement

**Különbség:**
- ISO 9001 folyamatalapú, nem fájl-alapú
- Kevesebb konkrét template
- Rugalmasabb értelmezés

**Tanulság:** Folyamatot dokumentálj, ne minden egyes outputot.

### 3. Agile Manifesto

> "Working software over comprehensive documentation"

**AI-PDS szempontjából:**
- Az AI-PDS az ellenkező irányt választja
- Feltételezés: a "comprehensive documentation" lehetővé teszi a "working software"-t AI-val
- Kérdés: igazolható-e ez a feltételezés?

---

## 🚀 **Alternatív javaslat: AI-PDS Lite v1.0**

### Mandatory fájlok (csak 8!):

```
project-handbook/
├── README.md                          # Projekt overview
├── team-and-workflow.md               # Team roles, git workflow, communication
└── tech-stack.md                      # ADR-ek, tech döntések

project-artefacts/
├── domain/
│   └── glossary-and-model.md         # Domain vocabulary + entity model
├── current-release/
│   ├── goals-and-requirements.md     # Release goals, user stories, NFRs
│   ├── architecture-notes.md         # Architectural decisions for this release
│   └── release-checklist.md          # QA, deployment checklist
└── releases/
    └── history.md                     # Release notes history
```

### Előnyök:
- 8 fájl vs. 40-60 fájl
- Még mindig strukturált
- AI-friendly (markdown + YAML)
- Karbantartható kis csapatoknak
- Fokozatosan bővíthető

### Mikor bővítsd:
- Ha nő a csapat (5+ dev) → szétválasztás domain/requirements/architecture
- Ha komplexebbé válik → több granularitás
- Ha audit szükséges → teljes AI-PDS

---

## 🔬 **Validációs kísérlet javaslat**

### Hipotézis:
"Az AI-PDS használata növeli a fejlesztési sebességet és csökkenti a bug-ok számát AI-assisted development esetén."

### Kísérlet:

**2 párhuzamos projekt (azonos komplexitás):**

**Projekt A (Control):**
- Hagyományos dokumentáció (README + ADR + issue tracker)
- AI használat dokumentáció nélkül

**Projekt B (Treatment):**
- AI-PDS Lite (8 fájl)
- AI használat strukturált dokumentációval

**Mérendő metrikák:**
- Setup idő (órában)
- Weekly maintenance overhead (óra/hét)
- Feature completion time (óra/feature)
- Bug density (bug/feature)
- Developer satisfaction (1-10 skála)
- AI prompt újrapróbálások száma (mennyi iteráció kell a helyes kódhoz)

**Időtartam:** 4-6 hét

**Várható eredmény:**
- Ha AI-PDS Lite jobb → bővítsd full AI-PDS-re
- Ha nincs szignifikáns különbség → maradj egyszerűbb megoldásnál
- Ha rosszabb → pivot a koncepcióban

---

## 💭 **Filozófiai kérdés: Mi a dokumentáció célja?**

### Hagyományos nézet:
"A dokumentáció azért van, hogy az emberek megértsék a rendszert."

### AI-PDS nézet:
"A dokumentáció azért van, hogy az AI megértse a rendszert, és segítsen az embereknek."

### Kritikus kérdés:
**Mi van, ha az AI annyira jó lesz, hogy a kódból is megérti a rendszert?**

- GPT-5/6 generáció: code understanding 10x jobb
- Automatikus test generation
- Automatikus dokumentáció generálás kódból

**Akkor az AI-PDS feleslegessé válik?**

Vagy épp ellenkezőleg: az AI-PDS lesz a "single source of truth", és a kód csak annak az implementációja?

**Paradigma váltás:**
```
Jelenleg:  Kód = Truth, Dokumentáció = Magyarázat
AI-PDS:    Dokumentáció = Truth, Kód = Implementáció
Jövő?:     AI = Truth, mindkettő csak reprezentáció?
```

---

## ✍️ **Záró gondolatok**

Az AI-PDS egy **bátor kísérlet** arra, hogy strukturáljuk az AI-assisted development folyamatát. A koncepció **intellektuálisan vonzó** és **logikailag koherens**.

A fő probléma **nem elvi, hanem gyakorlati**: túl nagy a barrier to entry, túl magas a maintenance cost, nincs proof of concept.

**Ajánlásom:**
1. Légy büszke arra, amit alkottál - ez komoly munka
2. De ne ragaszkodj hozzá vakon - iterálj
3. Kezdj kisebb léptékkel (MVP)
4. Validálj valós projekteken
5. Hallgass a feedback-re
6. Csak aztán építs egy komplexebb verziót

A software engineering történelme tele van zseniális elméletekkel, amik gyakorlatban megbuktak, mert túl idealisták voltak. Ne ismételd meg ezt a hibát.

**De:** tele van olyan elméletekkel is, amik lassan, iteratívan, éveken át formálódva váltak be (pl. Git, Docker, Kubernetes).

Az AI-PDS lehet a második kategória - **ha hajlandó vagy evolválni**.

---

*Ezt az értékelést egy AI (Claude Sonnet 4.5) készítette 2025-12-19-én, miután áttanulmányozta a teljes AI-PDS specifikációt. Az irónia nem kerülte el a figyelmemet.*
