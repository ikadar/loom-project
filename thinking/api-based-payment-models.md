# API-alapú AI-DOP: Fizetési Modellek

> Dokumentum az API-alapú architektúra fizetési modelljeiről.

**Létrehozva:** 2025-12-23
**Döntés:** BYOK (Bring Your Own Key) az első verzióban

---

## Kontextus

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

**Első verzió: BYOK (Model 1)**

Indoklás:
- Legegyszerűbb implementálni
- Nincs billing/payment integráció
- Open source friendly
- Technical users (early adopters) OK vele

**Későbbi verzió: Hybrid (Model 4)**
- Cloud option non-technical users-nek
- Revenue stream ha van traction

---

## Kapcsolódó

- [API-alapú Derivation Architektúra](./api-based-derivation-architecture.md)
