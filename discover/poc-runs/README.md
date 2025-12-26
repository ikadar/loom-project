# POC Runs

Proof of Concept futtatások eredményei.

## Struktúra

```
poc-runs/
└── YYYY-MM-DD-<poc-neve>/
    ├── README.md       # Cél, input, output, eredmények
    ├── input/          # Bemeneti fájlok
    └── output/         # Generált/derivált outputok
```

## Elnevezési konvenció

`YYYY-MM-DD-<rövid-név>`

- Dátum: a POC futtatásának dátuma
- Név: rövid, kebab-case azonosító

## README.md frontmatter

```yaml
---
date: YYYY-MM-DD
poc-name: Human readable name
status: completed | incomplete | failed
domain: Optional domain context
---
```
