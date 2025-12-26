---
title: "Loom Platform Business Model"
status: draft
created: 2025-12-23
updated: 2025-12-26
split-from: platform-strategy.md
see-also:
  - ../platform/platform-architecture.md  # Technikai architektúra
  - README.md                              # Üzleti stratégia összefoglaló
---

# Loom Platform Business Model

> Monetizáció, IP védelem, jogi kérdések, Go-to-Market

**Státusz:** AKTUÁLIS IRÁNY (policy tisztázás szükséges!)

---

## Összefoglaló

| Döntés | Választás |
|--------|-----------|
| **Architektúra** | Plugin + Dumb CLI + Smart SaaS |
| **CLI** | Go, ingyenes, NO embedded prompts (thin client) |
| **CLI pozícionálás** | "Ingyenes prompt delivery tool + convenience execution" |
| **SaaS pozícionálás** | "AI Development Orchestration Platform" |
| **IP védelem** | ~95% (prompts server-side) |
| **Monetizáció** | SaaS subscription (CLI ingyenes!) |
| **LLM költség** | User Claude Code subscription |
| **Tiers** | Free (limited) → Pro ($19) → Team ($49) → Enterprise |
| **Policy státusz** | Kettős pozícionálással erősebb, de Anthropic megerősítés ajánlott |

---

## IP Védelem

### Védelmi szintek

```
Remote API (szerveren)    ██████████  ~95%  (nem másolható) ← VÁLASZTOTT
Compiled + Encrypted      ████████░░  ~70%  (effort kell)
Compiled (plain)          ██████░░░░  ~50%  (strings, strace)
Plugin (plain text)       ██░░░░░░░░  ~10%  (bárki látja)
```

### Dumb CLI + Smart SaaS = ~95% védelem

| Aspektus | Embedded Prompts (régi) | Fetched from SaaS (új) |
|----------|-------------------------|------------------------|
| **IP védelem** | ~70% (binary) | ~95% (server-side) |
| **Frissítés** | CLI update kell | Instant, server-side |
| **Lock-in** | Gyenge | Erős (CLI haszontalan SaaS nélkül) |
| **Érték lokáció** | CLI-ben (kérdéses) | Egyértelműen SaaS-ban |
| **Legitimáció** | Szürke zóna | Tiszta modell |
| **Offline mód** | Működik | Nem működik |

### Az igazi érték a SaaS-ban

A fizetős termék értéke nem a technikai funkciók, hanem:

| Érték | Leírás |
|-------|--------|
| **Tudományos háttér** | Software engineering research, akadémiai alapok |
| **Karbantartott promptok** | Folyamatosan javított, A/B tesztelt, validált |
| **Checklists** | Domain-specifikus, iparági best practices alapján |
| **Templates** | Kipróbált sablonok különböző domainekhez |
| **Knowledge base** | Szoftverfejlesztési tudás strukturáltan |

**Ez a "secret sauce" ami a SaaS-ban él, nem a CLI-ben.**

---

## Monetizáció: SaaS Subscription

### Tier Struktúra

| Tier | Ár | Limit | Features |
|------|-----|-------|----------|
| **Free** | $0 | 10 derivation/hó | Basic prompts, standard templates |
| **Pro** | $19/hó | Unlimited | Latest prompts, priority updates, advanced templates |
| **Team** | $49/hó/user | Unlimited | Pro + shared config, team dashboard |
| **Enterprise** | Custom | Unlimited | Team + SSO, audit, on-prem, dedicated support |

### License Validation Flow

```
┌─────────────────────────────────────────────────────────────┐
│  $ loom derive ./l0 ./l1                                    │
│                                                              │
│  loom-cli startup:                                          │
│  ├── Load cached license                                    │
│  ├── If expired/missing:                                    │
│  │   └── POST loom.dev/api/validate {token}                │
│  │       └── Response: {valid, tier, features}             │
│  ├── Cache for 24h (offline support)                       │
│  ├── Feature gate based on tier                            │
│  └── Run command                                            │
│                                                              │
│  First time:                                                 │
│  $ loom login                                                │
│  → Browser: loom.dev/login                                  │
│  → Subscribe / enter license                                │
│  → Token saved: ~/.loom/license                             │
└─────────────────────────────────────────────────────────────┘
```

### Miért SaaS Subscription?

| Szempont | Előny |
|----------|-------|
| **IP védelem** | Promptok sosem hagyják el a szervert |
| **Fair** | Fizetsz = használod |
| **Kipróbálható** | Free tier 10 derivation/hó |
| **Continuous improvement** | Prompt update = instant, no CLI release |
| **A/B testing** | Különböző prompt verziók tesztelése |

---

## Költségek

### Loom Költségei

| Komponens | Költség |
|-----------|---------|
| SaaS hosting | ~$50-100/hó |
| Domain, CDN | ~$20/hó |
| Stripe fees | 2.9% + $0.30/tranzakció |
| **LLM költség** | **$0** (user subscription) |

### User Költségei

| Tétel | Költség |
|-------|---------|
| Claude Code subscription | $20-100/hó (már van) |
| Loom Free | $0 |
| Loom Pro | $19/hó |

---

## Go-to-Market

### Phase 1: MVP

1. CLI wrapper (Go binary, thin client)
2. SaaS backend (prompts, license validation)
3. Free tier (10/hó limit)
4. Plugin (slash commands)
5. Documentation

### Phase 2: Growth

1. Pro tier launch ($19/hó)
2. Prompt improvements based on feedback
3. Advanced templates
4. Community building

### Phase 3: Team/Enterprise

1. Team tier ($49/hó/user)
2. Shared config, dashboard
3. Enterprise features (SSO, audit)
4. On-premise option

---

## Kockázatok

| Kockázat | Valószínűség | Hatás | Mitigáció |
|----------|--------------|-------|-----------|
| Anthropic beépíti | Közepes | Magas | Niche features, ecosystem |
| Claude Code pricing változás | Alacsony | Közepes | Adaptáció |
| **Policy szürke zóna** | **Közepes** | **Magas** | **Anthropic megerősítés kérése** |

---

## FONTOS: Agent SDK vs Claude Code Policy

### Anthropic Policy az Agent SDK-ról

Az Agent SDK dokumentációból (https://platform.claude.com/docs/en/agent-sdk/overview):

> "Unless previously approved, **we do not allow third party developers to offer Claude.ai login or rate limits** for their products, including agents built on the Claude Agent SDK. **Please use the API key authentication methods** described in this document instead."

### Mit jelent ez?

```
┌─────────────────────────────────────────────────────────────────┐
│  Claude Code CLI (claude -p)                                    │
│  └─→ Saját használatra, Claude Code subscription-nel            │
│      (Személyes/interaktív használat)                           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  Agent SDK (Python/TypeScript)                                  │
│  └─→ ANTHROPIC_API_KEY kötelező                                 │
│      Third-party devs NEM használhatják a subscription-t!       │
│      Pay-per-token billing                                      │
└─────────────────────────────────────────────────────────────────┘
```

### A Loom Szürke Zóna Kérdése

**Loom működése:**
- User fizet a Loom SaaS-ért (subscription)
- CLI hívja a `claude -p`-t a user gépén
- A user saját Claude Code subscription-jét használja

**A kérdés:** Ez "offering Claude through your product"?

#### Érvek, hogy OK:

| Érv | Magyarázat |
|-----|------------|
| User saját gépén fut | Nincs proxy, nincs SaaS server |
| User saját subscription-je | Mint bármely automation script |
| Nem szolgáltatás | Desktop tool, nem hosted service |
| User maga is megcsinálhatná | Csak kényelem/automatizálás |
| Hasonló létező tools | VS Code extensions, Cursor, Aider |

#### Érvek, hogy Szürke Zóna:

| Érv | Aggály |
|-----|--------|
| Pénzt kérünk érte | Claude képességeit monetizáljuk |
| A prompt a miénk | IP a Claude outputján alapul |
| Ezért veszik meg | Lényegében Claude-ra épülő termék |

### Javaslat: Anthropic Megerősítés

**TEENDŐ:** Mielőtt production-be megyünk, kérjünk írásos megerősítést az Anthropic-tól.

**Kapcsolat:**
- https://www.anthropic.com/contact-sales
- support@anthropic.com

**Email tartalma:**
1. CLI tool ami `claude -p`-t hív a user gépén
2. User saját Claude Code subscription-jét használja
3. Mi a SaaS-t monetizáljuk (prompts, knowledge), nem a Claude hozzáférést
4. Kérés: írásos megerősítés, hogy ez megfelel a policy-nek

**Státusz:** TISZTÁZANDÓ

### Alternatívák ha NEM engedélyezik

1. **API-based modell**: ANTHROPIC_API_KEY használata, pay-per-token
   - Drágább a user-nek
   - De egyértelműen compliant

2. **Hybrid**: Free tier = user subscription, Pro = API key
   - Komplexebb implementáció
   - De mindkét use case-t lefedi

3. **Enterprise focus**: On-prem API server
   - Teljes IP védelem
   - De magasabb belépési küszöb

---

## Legitimációs Stratégia

### Plugin + SaaS + Helper CLI

```
┌─────────────────────────────────────────────────────────────┐
│  1. Claude Code Plugin (official mechanism)                 │
│     └─→ /loom-derive slash command                          │
│         (Ez a user-facing interface)                        │
│                                                              │
│  2. Helper CLI Binary (implementation detail)                │
│     └─→ loom-cli                                             │
│         (Thin client, csak orchestration)                   │
│                                                              │
│  3. SaaS Backend (ez a fizetős termék)                       │
│     └─→ Prompts, checklists, knowledge base                 │
│         (Valódi értéket ad, nem csak license gate)          │
│                                                              │
│  4. User's Claude Code Subscription (user's own)             │
│     └─→ claude -p calls                                      │
│         (User saját subscription-je)                         │
└─────────────────────────────────────────────────────────────┘
```

### Iparági precedensek

| Product | CLI | SaaS |
|---------|-----|------|
| **GitHub CLI** | Ingyenes | GitHub.com (private repos, actions) |
| **Terraform CLI** | Ingyenes | Terraform Cloud (state, teams) |
| **Docker CLI** | Ingyenes | Docker Hub (private repos) |
| **Vercel CLI** | Ingyenes | Vercel (hosting, analytics) |

Ezek mind legitim modellek, mert a SaaS valódi értéket ad.

### Loom SaaS Értékajánlat

A SaaS-nak valódi értéket kell adnia:

1. **Spec Repository** - verziókövetés, history
2. **Decision Log** - minden döntés auditálható
3. **Team Workspace** - kollaboráció spec-eken
4. **Project Dashboard** - spec coverage, progress
5. **Template Marketplace** - domain-specifikus template-ek
6. **Integrations** - Jira, GitHub, Linear, Notion
7. **AI Review** - spec quality scoring, gap analysis

### Legitimációs rétegek

1. **Plugin** = Official Claude Code extension mechanism
2. **CLI** = "Dumb" helper, csak orchestration, nincs IP benne
3. **SaaS** = "Smart", itt él az IP (prompts, checklists, knowledge)
4. **Claude** = User saját subscription-je

**Ez a kombináció legitimebb**, de nem "megoldja" a policy kérdést. Anthropic megerősítés továbbra is ajánlott.

---

## Positioning Strategy: Kettős Pozícionálás

### A Lényeg

A policy compliance érdekében **eltérő pozícionálást** alkalmazunk a SaaS és a CLI komponensekre. A technikai megvalósítás nem változik, csak a narratíva/marketing/üzleti definíció.

### Komponensek Pozícionálása

| Komponens | Pozícionálás | Fizetős? | Cél |
|-----------|--------------|----------|-----|
| **Loom SaaS** | AI Development Orchestration Platform | ✅ Igen | Teljes értékajánlat megőrzése |
| **Loom CLI** | Ingyenes prompt delivery tool | ❌ Nem | Policy compliance, alázatos framing |

### Loom SaaS Narratíva

> **"AI Development Orchestration Platform"**
>
> Strukturált specifikáció deriválás, L0→L3 pipeline, traceability, TDAI, knowledge base.

A SaaS megtartja az eredeti, teljes értékű víziót. Ez a fizetős termék, ami valódi értéket ad:
- Prompt engineering tudás és templates
- Software engineering knowledge base
- Collaboration, verziókezelés
- Integrációk, dashboard

### Loom CLI Narratíva

> **"Ingyenes eszköz, ami letölti a Loom promptokat és kényelemből futtatja is a te Claude subscription-ödön"**

A CLI alázatos pozícionálása:
- **Nem** "AI automation tool"
- **Hanem** "prompt delivery + convenience execution"
- Az érték a **SaaS-ban van** (a promptokban), nem a CLI-ben
- A CLI csak **kényelmi funkció** - a user akár manuálisan is futtathatná a promptokat

### Miért Működik Ez?

```
Régi narratíva (problémás):
"Fizess a Loom-ért, ami Claude-ot hív helyetted"
→ "Monetizáljátok a Claude hozzáférést?" (szürke zóna)

Új narratíva (tisztább):
"Fizess a Loom prompt engineering tudásáért (SaaS).
 A CLI ingyenes - csak letölti és mellesleg futtatja is a promptokat."
→ "Prompt template-eket árultok, a CLI csak delivery mechanism" (legitim)
```

### Analógia

| Termék | Amit fizetőssé tesz | Ingyenes rész |
|--------|---------------------|---------------|
| **VS Code** | Extensions, Copilot | Editor + code execution |
| **GitHub** | Private repos, Actions | CLI + public repos |
| **Loom** | SaaS (prompts, knowledge) | CLI (prompt delivery + execution) |

A Loom CLI olyan mint egy text editor "Run" gombja - nem ezért veszi meg valaki a terméket, csak kényelmi funkció.

### Policy Érvelés

Ezzel a pozícionálással az Anthropic felé:

1. **Nem monetizáljuk a Claude hívást** - a CLI ingyenes
2. **A fizetős termék (SaaS) nem hív Claude-ot** - csak promptokat szolgáltat
3. **A CLI csak kényelmi eszköz** - a user maga is futtathatná a promptokat manuálisan
4. **Hasonló létező eszközök** - VS Code extensions, GitHub CLI, stb.

> **Megjegyzés:** Ez a pozícionálás erősíti a policy compliance érveket, de az Anthropic megerősítés továbbra is ajánlott a biztonság kedvéért.

---

## Kapcsolódó

- [Platform Architecture](../platform/platform-architecture.md) - Technikai architektúra
- [Business Strategy](./README.md) - Üzleti stratégia összefoglaló
