---
name: card-compare
description: Side-by-side comparison of two major-US credit cards across fees, earning rates, credits, transfer partners, and key benefits. Covers Amex, Chase, Capital One, Citi, Bank of America, Discover, and Wells Fargo.
metadata:
  openclaw:
    requires:
      env:
        - BRAVE_API_KEY
      bins:
        - curl
    primaryEnv: BRAVE_API_KEY
---

# Card Compare

Return a compact side-by-side comparison of two exact card variants.

## When To Use

When the user asks to compare two credit cards. Trigger phrases: "card-compare", "compare", "vs", "versus", "which is better", "[card A] or [card B]".

## Input Format

The user provides two card names separated by "vs", "versus", "or", or a comma:
- `card-compare Amex Gold vs Chase Sapphire Preferred`
- `card-compare CSR, Venture X`

## Workflow

1. **Parse two card names** from the input.
2. **Resolve each card** — normalize and match to exact variants. If either is ambiguous, return a numbered choice list for that card and stop.
3. **Search** — run one Brave Search API call for comparison data.
4. **Compile** — assemble side-by-side report.
5. **Confidence** — flag uncertain or conflicting claims.

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
| Bilt | Bilt Blue / Obsidian / Palladium (ambiguous — ask) |
| Bilt Blue | Bilt Blue Card |
| Bilt Obsidian | Bilt Obsidian Card |
| Bilt Palladium | Bilt Palladium Card |
| Robinhood | Robinhood Gold Card / Cash Card (ambiguous — ask) |
| Robinhood Gold | Robinhood Gold Card |
| Robinhood Cash | Robinhood Cash Card |

### Supported Issuers

American Express, Bank of America, Bilt, Capital One, Chase, Citi, Discover, Robinhood, Wells Fargo.

## Step 2: Search

Run one Brave Search API call:

```bash
curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD_A+vs+CARD_B+compare&count=20" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

### Source Policy

- **Issuer-first**: check both cards' official product pages before secondary sources.
- **Max 5 secondary sources** from: NerdWallet (preferred), The Points Guy (preferred), Doctor of Credit (preferred), One Mile at a Time (preferred), Bankrate (preferred), Upgraded Points.
- **Disallowed**: Reddit, Facebook, Instagram, TikTok, X, YouTube, referral links, user forums.

## Required Output Sections

### `## 💰 Fees`
Two-column table: annual fee, foreign transaction fee, net after credits.

### `## 📈 Earning Rates`
Two-column table: key categories with multipliers for each card.

### `## 🏷️ Credits`
Two-column comparison of statement credits.

### `## 🔄 Transfer Partners`
Two-column comparison of transfer programs and partner count.

### `## ✈️ Key Benefits`
Two-column comparison of top travel/lifestyle benefits.

### `## 🏆 Bottom Line`
Concise factual summary of which card wins in each major dimension — not a recommendation.

### `## 📋 Confidence Notes`
Flag any uncertain, unconfirmed, or conflicting claims.

## Output Rules

- Use two-column format (Card A | Card B) for comparison sections.
- Use one emoji per section heading.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when both matches are confident.
- Do not show inline links, sources footer, or YAML blocks in output.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources
- **unconfirmed**: plausible but not fully resolved
- **conflicting**: sources disagree on a material fact
