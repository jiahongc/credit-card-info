---
name: card-credits
description: Return statement credits and cash-like credits for one major-US credit card — amount, cadence, trigger rules, enrollment requirements, and restrictions. Covers Amex, Chase, Capital One, Citi, Bank of America, Discover, and Wells Fargo.
metadata:
  openclaw:
    requires:
      env:
        - BRAVE_API_KEY
      bins:
        - curl
    primaryEnv: BRAVE_API_KEY
---

# Card Credits

Return the credits view of one exact card variant in compact format.

## When To Use

When the user asks about a card's statement credits, annual credits, or cash-back credits. Trigger phrases: "card-credits", "statement credits", "what credits", "annual credits", "perks".

## Workflow

1. **Resolve card identity** — normalize the input and match to one exact card variant.
2. **Search issuer only** — run one Brave Search API call scoped to the issuer domain. Do NOT search secondary sources.
3. **Compile** — combine search snippets with knowledge to fill all required sections.
4. **Confidence** — flag uncertain or conflicting claims.

## Step 1: Card Identity Resolution

Normalize the card name and resolve to an exact issuer + family + variant.

### Common Abbreviations

| Input | Resolved |
|---|---|
| CSP | Chase Sapphire Preferred |
| CSR | Chase Sapphire Reserve |
| CFU | Chase Freedom Unlimited |
| CFF | Chase Freedom Flex |
| Amex Gold | American Express Gold Card |
| Amex Plat | American Express Platinum Card |
| Venture X | Capital One Venture X Rewards Credit Card |
| Savor | Capital One SavorOne / Savor (ambiguous — ask) |
| Double Cash | Citi Double Cash Card |
| Custom Cash | Citi Custom Cash Card |

### Ambiguity Rules

- If the input maps to 2+ plausible variants, return a **numbered choice list** and stop.
- If no match exists, return: "Could not match a card. Try including the full card name with issuer."

### Supported Issuers

American Express, Bank of America, Capital One, Chase, Citi, Discover, Wells Fargo.

## Step 2: Search (Issuer Only)

```bash
curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+credits+benefits+site:ISSUER_DOMAIN&count=5" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

Use up to 1 secondary source (prefer Bankrate) for credit trigger details if needed. Combine the issuer search snippets with training knowledge.

## Required Output Sections

### `## 💳 Credits Overview`
Total annual credit value, number of distinct credits, general enrollment requirements.

### `## 🏷️ Credit Details`
Numbered list of each credit with amount, cadence (monthly/annual/semiannual), trigger (what purchase activates it), and restrictions.

### `## 📏 Usage Rules`
Enrollment requirements, expiration, stacking rules, clawback conditions.

### `## 📋 Confidence Notes`
Flag any detail that may have changed since training data.

## Output Rules

- Use one emoji per section heading and numbered lists for credits.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when the match is confident.
- Do not show inline links, sources footer, or YAML blocks in output.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources
- **unconfirmed**: plausible but not fully resolved
- **conflicting**: sources disagree on a material fact
