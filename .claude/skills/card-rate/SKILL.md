---
name: card-rate
description: Return earning rates, caps, exclusions, activation requirements, and merchant-coding caveats for one major-US credit card. Use when the user wants the earn-side details only.
argument-hint: [card name]
disable-model-invocation: true
allowed-tools: Read, Bash(curl -sS *)
---

# Card Rate

## Goal

Return the earning-structure view of one exact card variant in compact format.

## Search Strategy: knowledge-first

Use training knowledge plus one issuer page fetch + one secondary source for merchant-coding caveats. Do NOT spawn a subagent — answer directly.

## Workflow

1. Resolve the card using [../card-identity/SKILL.md](../card-identity/SKILL.md). If ambiguous, return a numbered choice list and stop.
2. Run one Brave Search API call scoped to the issuer domain (see `search_method` in [../card-shared/source-policy.yaml](../card-shared/source-policy.yaml)):
   ```
   curl -sS "https://api.search.brave.com/res/v1/web/search?q=CARD+NAME+site:ISSUER_DOMAIN&count=5" -H "X-Subscription-Token: $BRAVE_API_KEY"
   ```
   If `$BRAVE_API_KEY` is not set, fall back to WebSearch.
3. Combine the search snippets with training knowledge to fill all required sections. Use up to 1 secondary source (prefer Bankrate) for merchant-coding caveats if needed.
4. Follow [../card-shared/confidence-rules.md](../card-shared/confidence-rules.md). Flag any detail that may have changed since training data.
5. Return compact markdown using the `card-rate` contract in [../card-shared/command-contracts.yaml](../card-shared/command-contracts.yaml).
6. YAML is internal only — do not include it in user-facing output.
7. Do not show inline links, a sources footer, or a "Why It Matters" section.

## Required Sections

- `## 📊 Rate Summary`
- `## 📈 Earning Categories`
- `## 🚫 Caps And Exclusions`
- `## 📋 Confidence Notes`

Omit `## Card Identity` when the match is confident. Use short category lines and a compact exclusions/caps block.
