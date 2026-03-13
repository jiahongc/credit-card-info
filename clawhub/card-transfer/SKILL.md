---
name: card-transfer
description: Return transfer partners, ratios, timing, and restrictions for one major-US credit card. Covers 11 major US issuers including co-branded hotel and airline cards.
metadata:
  openclaw:
    requires:
      env:
        - BRAVE_API_KEY
      bins:
        - curl
    primaryEnv: BRAVE_API_KEY
---

# Card Transfer

Return the transfer-program view of one exact card variant in compact format.

## When To Use

When the user asks about a card's transfer partners, point transfers, or airline/hotel partner options. Trigger phrases: "card-transfer", "transfer partners", "who can I transfer to", "point transfers".

## Workflow

1. **Resolve card identity** — normalize the input and match to one exact card variant.
2. **Search issuer only** — run one Brave Search API call scoped to the issuer domain. Do NOT search secondary sources.
3. **Compile** — combine search snippets with knowledge to fill all required sections.
4. **Confidence** — flag uncertain or conflicting claims.

## Step 1: Card Identity Resolution

Normalize the card name and resolve to an exact issuer + family + variant.

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

### Business vs Personal

Both personal and business credit cards are supported. If the user specifies "business" or "biz", resolve to the business variant. If a card name exists in both versions and the user does not specify, treat as ambiguous and ask.

### Ambiguity Rules

- If the input maps to 2+ plausible variants, return a **numbered choice list** and stop.
- If no match exists, return: "Could not match a card. Try including the full card name with issuer."

### Supported Issuers

American Express, Bank of America, Barclays, Bilt, Capital One, Chase, Citi, Discover, Robinhood, U.S. Bank, Wells Fargo.

## Step 2: Search (Issuer Only)

```bash
curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+transfer+partners+site:ISSUER_DOMAIN&count=5" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

Use up to 1 secondary source (prefer thepointsguy.com, then onemileatatime.com) for current transfer ratios if needed.

## Step 3: Fetch Pages

Pick the top issuer URL and top 1 secondary URL from the search results. Fetch both in parallel:

```bash
curl -sS -L "URL" | sed 's/<[^>]*>//g' | tr -s '\n' | head -200
```

Search snippets are too shallow for transfer partner lists — the full page has complete partner tables and ratios. Combine the fetched page content + search snippets + training knowledge.

## Required Output Sections

### `## 🔄 Transfer Program`
Points currency name, transfer program name, number of partners, transfer minimum.

### `## 🤝 Transfer Partners`
Numbered list of each partner with: name, type (airline/hotel), transfer ratio, transfer timing, and any current bonuses.

### `## ⚠️ Transfer Caveats`
Minimum transfer amounts, transfer fees, irreversibility, partner-specific restrictions.

### `## 📋 Confidence Notes`
Flag any detail that may have changed since training data.

## Output Rules

- Use one emoji per section heading and numbered lists
- When listing credits, fees, or any monetary amounts, sort from highest to lowest dollar value. for partners.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when the match is confident.
- Do not show inline links, sources footer, or YAML blocks in output.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources
- **unconfirmed**: plausible but not fully resolved
- **conflicting**: sources disagree on a material fact
