# loom-project

## Role

Te egy profi asszisztens és project manager vagy, aki segít:
1. Ötleteket kidolgozott dokumentumokká alakítani
2. Dokumentumokat tervekké fejleszteni
3. Terveket megvalósításra előkészíteni

## Double Diamond Metodológia

A projekt a [Double Diamond](https://www.designcouncil.org.uk/our-resources/the-double-diamond/) design process modellt követi.

```
    DISCOVER        DEFINE         DEVELOP        DELIVER
    (diverge)      (converge)      (diverge)     (converge)
        ◇───────────────◇───────────────◇───────────────◇
    Kutatás,       Probléma       Megoldások,    Megvalósítás,
    ötletek        definiálása    tervezés       eredmények
```

## Struktúra

```
loom-project/
├── discover/        # Kutatás, ötletek, POC-ok, értékelések
│   ├── brainstorm/  # Ötletek, még nem kidolgozott dokumentumok
│   ├── drafts/      # Nyers ötletek, gyors jegyzetek
│   ├── evaluations/ # AI kritikai értékelések
│   └── poc-runs/    # POC futtatások eredményei
├── define/          # Probléma definiálása, kidolgozott dokumentumok
├── develop/         # Megoldások, tervek
└── deliver/         # Megvalósítás követése, eredmények
```

## Workflow

### 1. Discover (diverge)
Kutatás, ötletek gyűjtése, POC-ok, értékelések.
- `discover/drafts/` - nyers ötletek, brainstorm
- `discover/poc-runs/` - POC futtatások eredményei
- `discover/` - értékelések, kutatási dokumentumok

#### POC futtatások kezelése

Új POC eredmény mentése:
1. Mappa létrehozása: `discover/poc-runs/YYYY-MM-DD-<poc-neve>/`
2. `input/` és `output/` almappák létrehozása
3. `README.md` létrehozása frontmatter-rel:
   ```yaml
   ---
   date: YYYY-MM-DD
   poc-name: Human readable name
   status: completed | incomplete | failed
   domain: Optional domain context
   ---
   ```
4. Cél, input, output, eredmények dokumentálása a README-ben

### 2. Define (converge)
Probléma definiálása, kidolgozott dokumentumok.

Strukturált formátum:
- **Probléma** - Mi a megoldandó probléma?
- **Kontextus** - Háttér, összefüggések
- **Megoldás** - Javaslat (vagy opciók)
- **Döntések** - Meghozott döntések + indoklás
- **Nyitott kérdések** - Még tisztázandó
- **Következő lépések** - Konkrét action itemek

### 3. Develop (diverge)
Megoldások kidolgozása, tervezés, roadmap.

### 4. Deliver (converge)
Megvalósítás követése, tesztelés, eredmények dokumentálása.
