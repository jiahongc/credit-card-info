---
name: card-wallet
description: Audit a multi-card wallet — earning map, credit stack, overlaps, gaps, and total annual cost. Evaluates a user's full card lineup. Covers 11 major US issuers including co-branded hotel and airline cards.
metadata:
  openclaw:
    requires:
      env:
        - BRAVE_API_KEY
      bins:
        - curl
    primaryEnv: BRAVE_API_KEY
---

# Card Wallet

Return a compact wallet audit for a set of cards the user holds.

## When To Use

When the user asks to evaluate their card lineup, check for overlap, or optimize their wallet. Trigger phrases: "card-wallet", "wallet audit", "my cards", "do I have overlap", "which cards should I keep".

## Input Format

The user provides a comma-separated list of card names:
- `card-wallet Chase Sapphire Preferred, Amex Gold, Citi Double Cash`

## Workflow

1. **Parse card list** from comma-separated input.
2. **Resolve each card** — normalize and match to exact variants. If any card is ambiguous, return a numbered choice list for that card and stop.
3. **Search** — run one Brave Search API call per card. Classify results as issuer or secondary by domain. Run all calls in parallel.
4. **Collect** — for each card: annual fee, top earning categories, statement credits, key benefits.
5. **Analyze** — identify overlapping earn categories, uncovered categories, redundant benefits, total fee burden.
6. **Confidence** — flag uncertain claims.

## Step 1: Card Identity Resolution

### Common Abbreviations

Only shorthands and ambiguous names need entries here. Cards with full, unambiguous names (e.g., "Chase Marriott Bonvoy Boundless", "Chase United Explorer", "American Express Hilton Honors Aspire") are resolved via search — no table entry needed.

| Input | Resolved |
|---|---|
| CSP | Chase Sapphire Preferred |
| CSR | Chase Sapphire Reserve |
| CFU | Chase Freedom Unlimited |
| CFF | Chase Freedom Flex |
| CIP | Chase Ink Business Preferred |
| CIC | Chase Ink Business Cash |
| CIU | Chase Ink Business Unlimited |
| Amex Gold | American Express Gold Card |
| Amex Plat | American Express Platinum Card |
| Amex Biz Gold | American Express Business Gold Card |
| Amex Biz Plat | American Express Business Platinum Card |
| Amex Blue Biz Plus | American Express Blue Business Plus Card |
| Amex Blue Biz Cash | American Express Blue Business Cash Card |
| Venture X | Capital One Venture X Rewards Credit Card |
| Venture X Business | Capital One Venture X Business Card |
| Savor | Capital One SavorOne / Savor (ambiguous — ask) |
| Spark Cash Plus | Capital One Spark Cash Plus |
| Spark Miles | Capital One Spark Miles |
| Double Cash | Citi Double Cash Card |
| Custom Cash | Citi Custom Cash Card |
| Ink Preferred | Chase Ink Business Preferred |
| Ink Cash | Chase Ink Business Cash |
| Ink Unlimited | Chase Ink Business Unlimited |
| Bilt | Bilt Blue / Obsidian / Palladium (ambiguous — ask) |
| Robinhood | Robinhood Gold Card / Cash Card (ambiguous — ask) |
| Aviator Red | Barclays AAdvantage Aviator Red World Elite Mastercard |
| Wyndham Rewards | Barclays Wyndham Rewards Earner Card / Plus / Business (ambiguous — ask) |
| Altitude Reserve | U.S. Bank Altitude Reserve Visa Infinite Card |
| Altitude Connect | U.S. Bank Altitude Connect Visa Signature Card |
| Altitude Go | U.S. Bank Altitude Go Visa Signature Card |
| Delta Gold | American Express Delta SkyMiles Gold Card |
| Delta Platinum | American Express Delta SkyMiles Platinum Card |
| Delta Reserve | American Express Delta SkyMiles Reserve Card |
| Delta Biz Gold | American Express Delta SkyMiles Gold Business Card |
| Delta Biz Plat | American Express Delta SkyMiles Platinum Business Card |
| Delta Biz Reserve | American Express Delta SkyMiles Reserve Business Card |

### Supported Issuers

American Express, Bank of America, Barclays, Bilt, Capital One, Chase, Citi, Discover, Robinhood, U.S. Bank, Wells Fargo.

## Step 2: Search (Per Card)

For each card, run one Brave Search API call:

```bash
curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+benefits+credits&count=10" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

Run all calls in parallel. Classify results by domain: issuer pages (use Issuer Domains table below) vs approved secondary sources. Optionally use 1 secondary source (prefer thepointsguy.com) for cross-checking.

### Fetch Pages

For each card, fetch the top issuer URL from search results. Optionally fetch 1 secondary URL:

```bash
curl -sS -L "URL" | sed 's/<[^>]*>//g' | tr -s '\n' | head -200
```

Run fetches in parallel. Search snippets alone miss detailed credit and rate info. Combine fetched page content + search snippets + training knowledge.

### Issuer Domains (for classifying results, not constraining searches)

| Issuer | Domain |
|---|---|
| American Express | americanexpress.com |
| Bank of America | bankofamerica.com |
| Barclays | cards.barclaycardus.com |
| Bilt | bfrrewards.com |
| Capital One | capitalone.com |
| Chase | chase.com |
| Citi | citi.com |
| Discover | discover.com |
| Robinhood | robinhood.com |
| U.S. Bank | usbank.com |
| Wells Fargo | wellsfargo.com |

## Required Output Sections

### `## 💰 Annual Cost`
Total annual fees across all cards, listed per card.

### `## 📈 Earning Map`
Table showing the best card for each major spend category (dining, travel, groceries, gas, streaming, other).

### `## 🏷️ Credits Stack`
Statement credits, cash-back rebates, and complimentary subscriptions only — not enhanced earn rates or point multipliers. All credits across the wallet with total annual value.

### `## 🔁 Overlap`
Numbered list of redundant earn categories or duplicate benefits.

### `## 🕳️ Gaps`
Numbered list of common spend categories not covered at a bonus rate by any card.

### `## 📋 Confidence Notes`
Flag any uncertain, unconfirmed, or conflicting claims.

### `## 🔗 Sources`
Numbered list of URLs fetched, as markdown hyperlinks with short "Site - Topic" labels.

## Output Rules

- Use one emoji per section heading and numbered lists for overlap/gaps.
- When listing credits, fees, or any monetary amounts, sort from highest to lowest dollar value.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when all matches are confident.
- Do not show YAML blocks in output.
- End every report with a `## 🔗 Sources` section listing each URL fetched during research as a markdown hyperlink with a short "Site - Topic" label, e.g. `[Chase - Sapphire Preferred](https://...)`.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources
- **unconfirmed**: plausible but not fully resolved
- **conflicting**: sources disagree on a material fact
