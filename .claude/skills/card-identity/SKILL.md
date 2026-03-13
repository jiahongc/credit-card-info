---
name: card-identity
description: Resolve a card name to an exact issuer, family, and variant. This is a hidden shared skill used internally by all card commands as their first step. Not user-facing.
user-invocable: false
allowed-tools: Read, Bash(curl -sS *)
---

# Card Identity

## Purpose

Centralized card-name resolution used by all commands in the card suite. Every user-facing command should call this logic as step 1 before doing any research.

## Resolution Steps

1. **Normalize input** — strip extra whitespace, fix common abbreviations (e.g., "CSP" → "Chase Sapphire Preferred", "Amex" → "American Express").
2. **Parse components** — extract issuer, card family, and card variant separately when possible.
3. **Check issuer scope** — verify the issuer is in the approved list from [../card-shared/source-policy.yaml](../card-shared/source-policy.yaml). If not, return a short failure: "This card is not from a supported issuer."
4. **Resolve variant** — if the input maps to exactly one variant, proceed with confident match. If multiple variants are plausible, follow the ambiguity rules below.

## Business vs Personal

- Both personal and business credit cards are supported.
- If the user specifies "business" or "biz", resolve to the business variant.
- If a card name exists in both personal and business versions and the user doesn't specify, treat it as **ambiguous** — return a numbered choice list with both variants.
- Common business indicators: "business", "biz", "Business Platinum", "Ink", "Spark", etc.

## Ambiguity Handling

- If the input maps to 2+ plausible variants (e.g., "Chase Sapphire" → Preferred or Reserve, or "Amex Gold" → personal or business):
  - Return a **numbered choice list** and stop. Do not guess.
  - Format: one card per line with issuer, family, variant.
- If no plausible match exists:
  - Return a short failure: "Could not match a card. Try including the full card name with issuer."

## Confident Match Output

When the match is confident, the calling command should:
- **Omit** the `## Card Identity` section from user-facing output.
- Populate the card identity YAML keys (`card_name`, `issuer`, `network`, `card_family`, `card_variant`) in the internal YAML block.

## Weak / Unconfirmed Match Output

When the match is the best-supported variant but not certain:
- **Show** the `## ⚠️ Card Identity` section in user-facing output with the matched variant and a note about competing matches.
- Set `status: unconfirmed` on the relevant summary field.
- List competing variants in `confidence_notes`.

## Common Abbreviations

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

## Rules Reference

Full rules at [../card-shared/card-identity-rules.md](../card-shared/card-identity-rules.md).
