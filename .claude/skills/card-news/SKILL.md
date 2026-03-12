---
name: card-news
description: Return material news about one major-US credit card from the last 3 months, including direct card changes, relevant issuer updates, and major approved-site coverage. Use when the user wants the latest card-specific developments.
argument-hint: [card name]
disable-model-invocation: true
allowed-tools: Read, WebSearch, WebFetch, Bash(curl -sS *)
---

# Card News

## Goal

Return the last-3-month news view of one exact card variant in compact format.

## Search Strategy: search-required

Fetch issuer newsroom + up to 2 secondary sources for recent coverage. Prefer Doctor of Credit and The Points Guy. Run all searches in parallel.

## Workflow

1. Resolve the card using [../card-identity/SKILL.md](../card-identity/SKILL.md). If ambiguous, return a numbered choice list and stop.
2. Apply the 3-month lookback and inclusion rules from [../card-shared/recency-rules.md](../card-shared/recency-rules.md).
3. In parallel: fetch the issuer newsroom/product page AND search up to 2 secondary sources (prefer Doctor of Credit, The Points Guy) for recent card news.
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
