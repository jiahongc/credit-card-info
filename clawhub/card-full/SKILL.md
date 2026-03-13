---
name: card-full
description: Return a compact full report for one major-US credit card — fees, welcome offer, earning rates, redemption, credits, travel benefits, protections, mechanics, eligibility, and strategy. Covers Amex, Chase, Capital One, Citi, Bank of America, Discover, and Wells Fargo.
metadata:
  openclaw:
    requires:
      env:
        - BRAVE_API_KEY
      bins:
        - curl
    primaryEnv: BRAVE_API_KEY
---

# Card Full

Research any major US credit card and return a compact, complete report.

## When To Use

When the user asks for a full credit card review, breakdown, or "tell me about [card]". Trigger phrases: "card-full", "full report", "tell me about the [card name]", "review [card name]".

## Workflow

1. **Resolve card identity** — normalize the input, fix abbreviations, and match to one exact card variant.
2. **Search** — run one Brave Search API call for the card's issuer page + secondary sources.
3. **Compile** — assemble the report using the required sections below.
4. **Confidence** — flag uncertain or conflicting claims in the Confidence Notes section.

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
| Bilt | Bilt Blue / Obsidian / Palladium (ambiguous — ask) |
| Bilt Blue | Bilt Blue Card |
| Bilt Obsidian | Bilt Obsidian Card |
| Bilt Palladium | Bilt Palladium Card |
| Robinhood | Robinhood Gold Card / Cash Card (ambiguous — ask) |
| Robinhood Gold | Robinhood Gold Card |
| Robinhood Cash | Robinhood Cash Card |
| Amex Biz Gold | American Express Business Gold Card |
| Amex Biz Plat | American Express Business Platinum Card |
| Amex Blue Biz Plus | American Express Blue Business Plus Card |
| Amex Blue Biz Cash | American Express Blue Business Cash Card |
| Ink Preferred | Chase Ink Business Preferred |
| Ink Cash | Chase Ink Business Cash |
| Ink Unlimited | Chase Ink Business Unlimited |
| CIU | Chase Ink Business Unlimited |
| CIC | Chase Ink Business Cash |
| CIP | Chase Ink Business Preferred |
| Spark Cash Plus | Capital One Spark Cash Plus |
| Spark Miles | Capital One Spark Miles |
| Venture X Business | Capital One Venture X Business Card |

### Business vs Personal

Both personal and business credit cards are supported. If the user specifies "business" or "biz", resolve to the business variant. If a card name exists in both versions and the user does not specify, treat as ambiguous and ask.

### Ambiguity Rules

- If the input maps to 2+ plausible variants (e.g., "Chase Sapphire" could be Preferred or Reserve), return a **numbered choice list** and stop. Do not guess.
- If no match exists, return: "Could not match a card. Try including the full card name with issuer."

### Supported Issuers

American Express, Bank of America, Bilt, Capital One, Chase, Citi, Discover, Robinhood, Wells Fargo. If the card is from an unsupported issuer, return: "This card is not from a supported issuer."

## Step 2: Search

Run one Brave Search API call:

```bash
curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+review+welcome+offer&count=20" \
  -H "X-Subscription-Token: $BRAVE_API_KEY"
```

Parse the JSON response — results are in `.web.results[]` with `.title`, `.url`, `.description` fields.

### Source Policy

- **Issuer-first**: always check the card's official product page before secondary sources.
- **Max 5 secondary sources** from this approved list:
  1. NerdWallet (nerdwallet.com) — preferred
  2. The Points Guy (thepointsguy.com) — preferred
  3. Doctor of Credit (doctorofcredit.com)
  4. Bankrate (bankrate.com)
  5. One Mile at a Time (onemileatatime.com)
  6. Upgraded Points (upgradedpoints.com)
- **Stop early** once all required sections are covered.
- **Disallowed**: Reddit, Facebook, Instagram, TikTok, X, YouTube, referral links, user forums.

### Issuer Domains

| Issuer | Domains |
|---|---|
| American Express | americanexpress.com, aboutamex.com |
| Bank of America | bankofamerica.com |
| Bilt | bfrrewards.com |
| Capital One | capitalone.com |
| Chase | chase.com, media.chase.com |
| Citi | citi.com, citicards.com |
| Discover | discover.com |
| Robinhood | robinhood.com |
| Wells Fargo | wellsfargo.com |

## Step 3: Required Output Sections

Return compact markdown with these sections in order:

### `## 💰 Fees`
Annual fee, authorized user fee, foreign transaction fee, balance transfer fee, cash advance fee, late fee.

### `## 🎁 Welcome Offer`
Public bonus, spend requirement, qualification window, eligibility restrictions, lifetime/family language.

### `## 📈 Earning Rates`
Base rate, bonus categories with multipliers, caps, point currency.

### `## 🔄 Redemption`
Transfer partners summary, portal options, cash-out rates, minimum redemption.

### `## 🏷️ Credits`
Each credit with amount, cadence, trigger, and restrictions.

### `## ✈️ Travel Benefits`
Lounge access, hotel status, rental car benefits, travel credits, companion fares.

### `## 🛡️ Protections`
Purchase protection, extended warranty, return protection, cell phone protection, fraud protections.

### `## ⚙️ Account Mechanics`
Virtual cards, authorized user handling, app capabilities, autopay notes.

### `## ✅ Eligibility`
Issuer family rules, known restriction language (e.g., Chase 5/24, Amex lifetime language).

### `## 🧭 Strategy`
Downgrade paths, no-fee fallback, ecosystem role, keeper value after year one.

### `## 📋 Confidence Notes`
Flag any uncertain, unconfirmed, or conflicting claims.

## Output Rules

- Use one emoji per section heading.
- Use numbered lists for list-heavy sections.
- Keep content to condensed facts — no prose padding.
- Omit the Card Identity section when the match is confident.
- Do not show inline links or a sources footer.
- Do not include YAML blocks in user-facing output.
- Do not show a "Why It Matters" section.

## Confidence Definitions

- **confirmed**: supported by issuer terms or multiple approved sources without disagreement
- **unconfirmed**: plausible but not fully resolved from approved sources
- **conflicting**: approved sources disagree on a material fact

Every report must include a `## 📋 Confidence Notes` section. Keep notes short and tied to concrete uncertainties.
