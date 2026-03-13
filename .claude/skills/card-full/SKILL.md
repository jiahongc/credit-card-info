---
name: card-full
description: Return a compact full report for one major-US credit card, covering fees, offer, earnings, redemption, credits, travel benefits, protections, mechanics, eligibility, and strategy. Use when the user wants the whole card picture.
argument-hint: [card name]
disable-model-invocation: true
allowed-tools: Read, Bash(curl -sS *)
---

# Card Full

## Goal

Return a compact, complete card framework for one exact card variant.

## Search Strategy: search-required

Fetch the issuer page first, then up to 5 secondary sources for current SUB/offer details. Prefer NerdWallet, The Points Guy, Doctor of Credit, Bankrate, and One Mile at a Time. Run searches in parallel.

## Workflow

1. Resolve the card using [../card-identity/SKILL.md](../card-identity/SKILL.md). If ambiguous, return a numbered choice list and stop.
2. Run one Brave Search API call covering the issuer domain + preferred secondary sources (see `search_method` in [../card-shared/source-policy.yaml](../card-shared/source-policy.yaml)):
   ```
   curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+review+welcome+offer&count=20" -H "X-Subscription-Token: $BRAVE_API_KEY"
   ```
   If `$BRAVE_API_KEY` is not set, fall back to WebSearch. Stop early if the contract is satisfied.
3. Follow section composition rules from [../card-shared/section-definitions.md](../card-shared/section-definitions.md).
4. Apply confidence handling from [../card-shared/confidence-rules.md](../card-shared/confidence-rules.md).
5. Return compact markdown using the `card-full` contract in [../card-shared/command-contracts.yaml](../card-shared/command-contracts.yaml).
6. YAML is internal only — do not include it in user-facing output.
7. Do not show inline links, a sources footer, or a "Why It Matters" section.

## Required Sections

- `## 💰 Fees`
- `## 🎁 Welcome Offer`
- `## 📈 Earning Rates`
- `## 🔄 Redemption`
- `## 🏷️ Credits`
- `## ✈️ Travel Benefits`
- `## 🛡️ Protections`
- `## ⚙️ Account Mechanics`
- `## ✅ Eligibility`
- `## 🧭 Strategy`
- `## 📋 Confidence Notes`

Omit `## Card Identity` when the match is confident. Each section should contain condensed facts in numbered lists where appropriate.
