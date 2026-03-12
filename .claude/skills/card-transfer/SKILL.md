---
name: card-transfer
description: Return transfer partners, transfer ratios, timing notes, and restrictions for one major-US credit card. Use when the user wants redemption-transfer details only.
argument-hint: [card name]
disable-model-invocation: true
allowed-tools: Read, WebSearch, WebFetch, Bash(curl -sS *)
---

# Card Transfer

## Goal

Return the transfer-program view of one exact card variant in compact format.

## Search Strategy: knowledge-first

Use training knowledge plus one issuer page fetch. Do NOT search secondary sources. Do NOT spawn a subagent — answer directly.

## Workflow

1. Resolve the card using [../card-identity/SKILL.md](../card-identity/SKILL.md). If ambiguous, return a numbered choice list and stop.
2. Fetch the issuer's official transfer partners / rewards program page for the card (one WebFetch or WebSearch call). Use the issuer domains in [../card-shared/source-policy.yaml](../card-shared/source-policy.yaml).
3. Combine the issuer page data with training knowledge to fill all required sections. Do not search secondary sources.
4. Follow [../card-shared/confidence-rules.md](../card-shared/confidence-rules.md). Flag any detail that may have changed since training data.
5. Return compact markdown using the `card-transfer` contract in [../card-shared/command-contracts.yaml](../card-shared/command-contracts.yaml).
6. YAML is internal only — do not include it in user-facing output.
7. Do not show inline links, a sources footer, or a "Why It Matters" section.

## Required Sections

- `## 🔄 Transfer Program`
- `## 🤝 Transfer Partners`
- `## ⚠️ Transfer Caveats`
- `## 📋 Confidence Notes`

Omit `## Card Identity` when the match is confident. Use a numbered list for transfer partners with ratio and one short note per item.
