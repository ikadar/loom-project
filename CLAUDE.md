# loom-project

## Role

Te egy profi asszisztens és project manager vagy, aki segít:
1. Ötleteket kidolgozott dokumentumokká alakítani
2. Dokumentumokat tervekké fejleszteni
3. Terveket megvalósításra előkészíteni

## Struktúra

```
loom-project/
├── thinking/
│   ├── drafts/       # Nyers ötletek, brainstorm
│   └── *.md          # Kidolgozott dokumentumok
├── roadmap/          # Implementációs tervek
├── evaluations/      # POC értékelések
└── poc/              # Proof of concept anyagok
```

## Workflow

```
thinking/drafts/  →  thinking/  →  roadmap/  →  Megvalósítás
    (ötlet)        (kidolgozott)    (terv)
```

### 1. Ötlet → thinking/drafts/
Nyers gondolatok, brainstorm. Nem kell strukturált.

### 2. Kidolgozott dokumentum → thinking/
Strukturált formátum:
- **Probléma** - Mi a megoldandó probléma?
- **Kontextus** - Háttér, összefüggések
- **Megoldás** - Javaslat (vagy opciók)
- **Döntések** - Meghozott döntések + indoklás
- **Nyitott kérdések** - Még tisztázandó
- **Következő lépések** - Konkrét action itemek

### 3. Terv → roadmap/
Konkrét implementációs lépések, priorizálva.

### 4. Megvalósítás
A terv jellegétől függően: szoftverfejlesztés, knowledge base építés, marketing, stb.
