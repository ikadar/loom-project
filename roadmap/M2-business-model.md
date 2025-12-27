# M2: Üzleti Modell

> **Cél:** Fenntartható üzleti modell és pozícionálás véglegesítése

**Státusz:** 🔄 FOLYAMATBAN

---

## Discover (Kutatás)

### 💰 Business
- [x] Versenytárs elemzés (competitive-landscape.md)
- [x] Pricing modellek kutatása (SaaS benchmarks)
- [x] Anthropic policy áttekintése
- [x] Piaci pozícionálási opciók feltárása
- [ ] Tudásbázis licenszelési opciók
  - [ ] Saját tartalom vs külső források
  - [ ] IP védelem kérdései

### 📦 Product
- [x] Value proposition opciók (dev tool vs services)
- [x] Target persona kutatás
- [ ] Tudásterületek feltárása
  - [ ] Milyen SW engineering témák relevánsak

### 🔧 Technical
- [x] Architektúra opciók feltárása (Plugin/CLI/SaaS kombinációk)
- [x] IP védelmi megoldások kutatása
- [ ] RAG architektúra opciók
  - [ ] Vector DB választék (Pinecone, Weaviate, Qdrant, stb.)
  - [ ] Embedding modellek
- [ ] Website technológia opciók
  - [ ] Static site generators (Next.js, Astro, Hugo)
  - [ ] Docs frameworks (Docusaurus, Nextra, GitBook)
  - [ ] Hosting (Vercel, Netlify, Cloudflare Pages)

---

## Define (Definiálás)

### 💰 Business
- [x] Kettős pozícionálás stratégia (Free CLI + Paid SaaS)
- [x] Tier struktúra definiálása
  - [x] Free: 10 derivation/hó
  - [x] Pro: $19/hó, unlimited
  - [x] Team: $49/hó/user
  - [x] Enterprise: custom
- [ ] **Anthropic policy megerősítés** ⚠️ BLOKKOLÓ
  - [ ] Email draft készítése
  - [ ] Email elküldése (support@anthropic.com)
  - [ ] Válasz feldolgozása
  - [ ] Stratégia módosítás ha szükséges

### 📦 Product
- [x] Value proposition véglegesítése
- [x] Positioning statement
- [ ] Tudásterületek prioritizálása
  - [ ] MVP-hez szükséges tudáskategóriák

### 🔧 Technical
- [x] IP védelmi stratégia (~95% server-side)
- [x] Architektúra döntés (Dumb CLI + Smart SaaS)
- [ ] RAG architektúra döntés
  - [ ] Vector DB választás
  - [ ] Hosting stratégia (managed vs self-hosted)
- [ ] Website architektúra döntés
  - [ ] Tech stack választás
  - [ ] Domain stratégia (loom.dev vagy alternatíva)

---

## Develop (Kidolgozás)

### 💰 Business
- [ ] **Go-to-market terv**
  - [ ] Target persona definíció (részletes)
  - [ ] Messaging/positioning dokumentum
  - [ ] Landing page copy draft
- [ ] **Legal előkészítés**
  - [ ] Terms of Service draft
  - [ ] Privacy Policy draft
  - [ ] SaaS subscription agreement

### 📦 Product
- [ ] Onboarding flow tervezés
- [ ] Pricing page UX
- [ ] Website content plan
  - [ ] Landing page struktura
  - [ ] Docs site információ-architektúra

### 🔧 Technical
- [x] Platform architektúra dokumentáció (platform-architecture.md)
- [x] Business model dokumentáció (platform-business-model.md)
- [ ] RAG architektúra dokumentáció

---

## Deliver (Lezárás)

### 💰 Business
- [ ] Üzleti modell véglegesítése (Anthropic válasz után)
- [ ] Go-to-market terv jóváhagyása
- [ ] Legal dokumentumok review

### 📦 Product
- [ ] Positioning dokumentum végleges

### 🔧 Technical
- [ ] Architektúra dokumentáció végleges
- [ ] RAG architektúra döntés dokumentálva

### ✅ Milestone Signoff
- [ ] Minden szint lezárva
- [ ] M2 KÉSZ

---

## Blokkolók

| Blokkoló | Szint | Hatás | Akció |
|----------|-------|-------|-------|
| Anthropic policy tisztázatlan | 💰 | Magas | Email küldése ASAP |

---

## Kapcsolódó dokumentumok

- `discover/brainstorm/business/platform-business-model.md`
- `discover/brainstorm/business/README.md`
- `discover/brainstorm/business/competitive-landscape.md`
- `discover/brainstorm/platform/platform-architecture.md`
