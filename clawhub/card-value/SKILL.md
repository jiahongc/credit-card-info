---
name: card-value
description: Estimate first-year value for one major-US credit card — welcome bonus + annual earn + credits minus annual fee. Accepts an optional spending breakdown. Covers Amex, Chase, Capital One, Citi, Bank of America, Discover, and Wells Fargo.
metadata:
  openclaw:
    requires:
      env:
        - BRAVE_API_KEY
      bins:
        - curl
    primaryEnv: BRAVE_API_KEY
---

# Card Value

Return a compact first-year value estimate for one exact card variant.

## When To Use

When the user asks about a card's value, worth, or first-year return. Trigger phrases: "card-value", "is it worth it", "first year value", "value estimate", "how much is the card worth".

## Input Format

The user provides a card name followed by an optional spending breakdown:
- `card-value Chase Sapphire Preferred` — uses default spend profile
- `card-value Chase Sapphire Preferred $500/mo dining, $200/mo travel, $3000/mo other` — uses provided breakdown

Default moderate-spender profile (if none given): $500/mo dining, $200/mo travel, $100/mo streaming, $200/mo groceries, $2000/mo other.

## Workflow

1. **Resolve card identity** — normalize and match to one exact variant. If ambiguous, return a numbered choice list and stop.
2. **Search** — run one Brave Search API call for current welcome offer and fee details.
3. **Research** — collect annual fee, welcome offer (bonus + spend requirement), earning rates by category, and statement credits.
4. **Compute** — `first_year_value = welcome_bonus_value + annual_earn_value + total_credits - annual_fee`.
5. **Confidence** — flag uncertain claims. Use 1 cpp baseline unless a known portal/transfer premium exists.

## Step 1: Card Identity Resolution

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

### Supported Issuers

American Express, Bank of America, Capital One, Chase, Citi, Discover, Wells Fargo.

## Step 2: Search

```bash
curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+welcome+offer+annual+fee&count=20" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

### Source Policy

- **Issuer-first**: check the card's official product page first.
- **Max 5 secondary sources** from: NerdWallet (preferred), The Points Guy (preferred), Bankrate (preferred), One Mile at a Time (preferred), Doctor of Credit (preferred), Upgraded Points.
- **Disallowed**: Reddit, Facebook, Instagram, TikTok, X, YouTube, referral links, user forums.

## Required Output Sections

### `## 💳 Spend Profile`
The spending breakdown being used (default or user-provided), formatted as a compact table.

### `## 🎁 Welcome Bonus`
Bonus amount, spend requirement, qualification window, point valuation used.

### `## 📈 Annual Earn`
Earn by category based on the spend profile, with point values.

### `## 🏷️ Credits`
Applicable statement credits with annual total.

### `## 💰 Net First-Year Value`
Show the math clearly: `welcome_bonus + annual_earn + credits - annual_fee = net_value`.

### `## 📋 Confidence Notes`
Flag any uncertain claims. Note the cpp valuation assumed.

## Output Rules

- Use one emoji per section heading.
- Show the math clearly in the Net First-Year Value section.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when the match is confident.
- Do not show inline links, sources footer, or YAML blocks in output.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources
- **unconfirmed**: plausible but not fully resolved
- **conflicting**: sources disagree on a material fact
