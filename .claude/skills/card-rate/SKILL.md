---
name: card-rate
description: Return earning rates, caps, exclusions, activation requirements, and merchant-coding caveats for one major-US credit card. Use when the user wants the earn-side details only.
argument-hint: [card name]
disable-model-invocation: true
allowed-tools: Read, WebSearch, WebFetch, Bash(curl -sS *)
---

# Card Rate

## Goal

Return the earning-structure view of one exact card variant in compact format.

## Search Strategy: knowledge-first

Use training knowledge plus one issuer page fetch. Do NOT search secondary sources. Do NOT spawn a subagent — answer directly.

## Workflow

1. Resolve the card using [../card-identity/SKILL.md](../card-identity/SKILL.md). If ambiguous, return a numbered choice list and stop.
2. Fetch the issuer's official product page for the card (one WebFetch or WebSearch call). Use the issuer domains in [../card-shared/source-policy.yaml](../card-shared/source-policy.yaml).
3. Combine the issuer page data with training knowledge to fill all required sections. Do not search secondary sources.
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
