---
name: card-compare
description: Side-by-side comparison of two major-US credit cards across fees, earning rates, credits, transfer partners, and key benefits. Covers 11 major US issuers including co-branded hotel and airline cards.
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

## Step 4: Fetch Pages

Pick the issuer URL for each card and up to 2 secondary URLs (prefer nerdwallet.com and thepointsguy.com) from the search results. Fetch in parallel:

```bash
curl -sS -L "URL" | sed 's/<[^>]*>//g' | tr -s '\n' | head -200
```

Search snippets are too shallow for accurate comparisons — the actual pages have complete rate tables, credit lists, and benefit details.

## Required Output Sections

### `## 💰 Fees`
Two-column table: annual fee, foreign transaction fee, net after credits.

### `## 📈 Earning Rates`
Two-column table: key categories with multipliers for each card.

### `## 🏷️ Credits`
Two-column comparison of statement credits, cash-back rebates, and complimentary subscriptions only — not enhanced earn rates or point multipliers.

### `## 🔄 Transfer Partners`
Two-column comparison of transfer programs and partner count.

### `## ✈️ Key Benefits`
Two-column comparison of top travel/lifestyle benefits.

### `## 🏆 Bottom Line`
Concise factual summary of which card wins in each major dimension — not a recommendation.

### `## 📋 Confidence Notes`
Flag any uncertain, unconfirmed, or conflicting claims.

### `## 🔗 Sources`
Numbered list of URLs fetched, as markdown hyperlinks with short "Site - Topic" labels.

## Output Rules

- Use two-column format (Card A | Card B) for comparison sections.
- Use one emoji per section heading.
- When listing credits, fees, or any monetary amounts, sort from highest to lowest dollar value.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when both matches are confident.
- Do not show YAML blocks in output.
- End every report with a `## 🔗 Sources` section listing each URL fetched during research as a markdown hyperlink with a short "Site - Topic" label, e.g. `[NerdWallet - CSR vs Amex Gold](https://...)`.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources
- **unconfirmed**: plausible but not fully resolved
- **conflicting**: sources disagree on a material fact
