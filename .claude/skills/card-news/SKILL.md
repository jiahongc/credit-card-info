---
name: card-news
description: Return material news about one major-US credit card from the last 3 months, including direct card changes, relevant issuer updates, and major approved-site coverage. Use when the user wants the latest card-specific developments.
argument-hint: [card name]
disable-model-invocation: true
allowed-tools: Read, Bash(curl -sS *)
---

# Card News

## Goal

Return the last-3-month news view of one exact card variant in compact format.

## Search Strategy: search-required

Fetch issuer newsroom + up to 5 secondary sources for recent coverage. Prefer Doctor of Credit, The Points Guy, One Mile at a Time, NerdWallet, and Bankrate. Run all searches in parallel.

## Workflow

1. Resolve the card using [../card-identity/SKILL.md](../card-identity/SKILL.md). If ambiguous, return a numbered choice list and stop.
2. Apply the 3-month lookback and inclusion rules from [../card-shared/recency-rules.md](../card-shared/recency-rules.md).
3. Run one Brave Search API call for recent news (see `search_method` in [../card-shared/source-policy.yaml](../card-shared/source-policy.yaml)):
   ```
   curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+news+CURRENT_YEAR&count=20&freshness=pm" -H "X-Subscription-Token: $BRAVE_API_KEY"
   ```
   The `freshness=pm` parameter limits results to the past month. If `$BRAVE_API_KEY` is not set, fall back to WebSearch.
4. Follow [../card-shared/confidence-rules.md](../card-shared/confidence-rules.md).
5. Return compact markdown using the `card-news` contract in [../card-shared/command-contracts.yaml](../card-shared/command-contracts.yaml).
6. YAML is internal only — do not include it in user-facing output.
7. Do not show inline links, a sources footer, or a "Why It Matters" section.

## Required Sections

- `## 📅 News Window`
- `## 📰 Recent Updates`
- `## 📝 Summary`
- `## 📋 Confidence Notes`

Omit `## Card Identity` when the match is confident. Use a numbered list for recent updates with a short summary line per item.
