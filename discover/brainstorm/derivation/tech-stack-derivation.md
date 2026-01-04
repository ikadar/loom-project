# Tech Stack Deriválás

## Összefoglaló

Ez a dokumentum a **tech stack deriválás** elveit tartalmazza - hogyan következik a technológiai választás a requirement-ekből.

**Státusz:** TODO - később kidolgozandó

---

## Alapelv

A tech stack **nem fix input**, hanem a requirement-ekből deriválható:

```
L0/L1 requirements
    ↓
Architecture Interview (deployment, CQRS, UoW, stb.)
    ↓
Tech stack ajánlás ← User jóváhagyás VAGY explicit megadás
    ↓
L2 tech-specs
```

**Tehát:**
- User megadhatja expliciten a tech stack-et
- Ha nem adja meg, Loom ajánl a requirement-ek alapján
- Az ajánlást user jóváhagyja vagy felülírja

---

## Kidolgozandó témák

- [ ] Nyelv választás (Go, TypeScript, Python, stb.)
- [ ] Database választás (PostgreSQL, MongoDB, stb.)
- [ ] Framework választás
- [ ] Infrastructure választás (cloud, on-prem)
- [ ] Interview kérdések tech stack-hez
- [ ] Deriválási szabályok (NFR → tech ajánlás)

---

## Kapcsolódó dokumentumok

- [system-level-patterns.md](system-level-patterns.md) - Rendszer szintű pattern döntések
- [design-patterns-analysis.md](design-patterns-analysis.md) - Pattern elemzés
