# API-alapú AI-DOP: Fizetési Modellek

> ⚠️ **SUPERSEDED** - Ez a dokumentum részben felülírva. Lásd: [platform-strategy.md](./platform-strategy.md)
>
> A fő irány most: **Claude Code Plugin** (user subscription fedezi a költséget).
> Az API-alapú modellek továbbra is relevánsak **Enterprise tier** esetén.

> Dokumentum az API-alapú architektúra fizetési modelljeiről.

**Létrehozva:** 2025-12-23
**Frissítve:** 2025-12-23
**Státusz:** RÉSZBEN FELÜLÍRVA (lásd platform-strategy.md)
**Döntés:** ~~BYOK~~ → Claude Code Plugin (fő irány) | BYOK/SaaS (enterprise)

---

## Új Irány (2025-12-23)

| Tier | Fizetési modell | Ki fizeti az LLM-et? |
|------|-----------------|---------------------|
| **Free (Plugin)** | $0 | User (Claude subscription) |
| **Enterprise** | Subscription | BYOK vagy SaaS (TBD) |

---

## Korábbi Kontextus (archív)

Ha az AI-DOP a felhasználó lokális Claude Code-ját használja (command-alapú), akkor nincs külön költség - a user már fizeti a Claude subscription-t.

API-alapú megvalósításnál viszont kérdés: **ki fizeti az API költséget?**

---

## Fizetési Modellek

### 1. BYOK (Bring Your Own Key) ✅ VÁLASZTOTT

```
User → AI-DOP (lokális vagy hosted) → User's Anthropic API Key
```

```python
# User config
config = {
    "anthropic_api_key": "sk-ant-...",  # User's key
    "input_dir": "./l0",
    "output_dir": "./l1"
}

loom.derive(config)
```

**Előnyök:**
- AI-DOP-nak nincs API költsége
- User teljes kontroll a költségek felett
- Nincs pricing/billing komplexitás
- Open source friendly

**Hátrányok:**
- User-nek kell Anthropic account
- Nem consumer-friendly (technikai setup)

---

### 2. SaaS Subscription

```
User → AI-DOP SaaS → AI-DOP's API Key → Anthropic
         ↓
    Monthly fee / Per-derivation fee
```

| Plan | Price | Includes |
|------|-------|----------|
| Free | $0 | 10 derivations/month |
| Pro | $29/mo | 200 derivations/month |
| Team | $99/mo | Unlimited + collaboration |

**Előnyök:**
- Consumer-friendly (no API key needed)
- Predictable cost for user
- Revenue stream

**Hátrányok:**
- AI-DOP viseli az API költséget
- Pricing kockázat (heavy users)
- Infrastructure költség

---

### 3. Pass-through + Markup

```
User → AI-DOP SaaS → AI-DOP's API Key → Anthropic
         ↓
    Actual API cost + 20% markup
```

**Előnyök:**
- Fair: user fizeti amit használ
- Nincs pricing kockázat
- Skálázódik

**Hátrányok:**
- Változó költség (user nem szereti)
- Billing komplexitás
- Versenyhátrány vs subscription

---

### 4. Hybrid: Lokális + Cloud Option

```
                    ┌─→ Lokális (BYOK) - Free
User → AI-DOP CLI ──┤
                    └─→ Cloud API (SaaS) - Paid
```

```bash
# Option A: User's key (free)
loom derive --api-key $ANTHROPIC_API_KEY

# Option B: AI-DOP cloud (paid, no key needed)
loom derive --cloud
```

---

## Összehasonlítás

| Model | User pays | AI-DOP pays | Complexity |
|-------|-----------|-------------|------------|
| **BYOK** | API directly | $0 | Low |
| **SaaS Subscription** | Fixed monthly | API + infra | High |
| **Pass-through** | Usage-based | Infra only | Medium |
| **Hybrid** | Choice | Depends | Medium |

---

## Döntés

**Választott modell: Local CLI + Remote API + BYOK**

```
┌─────────────────────────────────────────────────────┐
│                  User's Machine                      │
│  ┌─────────────────────────────────────────────────┐│
│  │         Loom CLI (open source)                  ││
│  │  - File access (local)                          ││
│  │  - Context building                             ││
│  │  - RAG (local Chroma)                           ││
│  └──────────────────┬──────────────────────────────┘│
└─────────────────────┼───────────────────────────────┘
                      │ {context, user_api_key}
                      ▼
         ┌────────────────────────────────────────────┐
         │         AI-DOP API (protected)             │
         │  🔒 Secret prompts (IP védett)             │
         │  → Claude API (user's key = BYOK)          │
         └────────────────────────────────────────────┘
```

**Indoklás:**

| Követelmény | Megoldás |
|-------------|----------|
| File access | ✅ Local CLI |
| IP védelem | ✅ Promptok szerveren |
| Nincs billing | ✅ BYOK (user fizeti Claude-ot) |
| Privacy | ✅ Csak context megy, nem teljes repo |

**Költségek:**

| Ki fizet | Mit |
|----------|-----|
| **User** | Claude API (BYOK) |
| **AI-DOP** | Szerver infra only (nincs LLM cost) |

---

## BYOK vs SaaS: Nulla Architektúra Különbség

A két modell között **csak az API key forrása** különbözik:

```python
def get_api_key(user, payment_model):
    if payment_model == "byok":
        return user.api_key          # User fizeti a Claude API-t
    else:  # saas
        track_usage(user)
        return os.environ["AIDOP_KEY"]  # AI-DOP fizeti
```

**Minden más ugyanaz:**
- Server kód
- Promptok
- Agentic loop
- Context handling

**Következmény:** BYOK → SaaS váltás = csak config + billing logic, nem architektúra változás.

**Későbbi SaaS opció:** Subscription tier non-technical users-nek

---

## Kapcsolódó

- [API-alapú Derivation Architektúra](./api-based-derivation-architecture.md)
