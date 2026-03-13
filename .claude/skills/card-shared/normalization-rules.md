# Normalization Rules

## Goal

Convert raw research notes into consistent, compact outputs across the full command suite.

## Shared Field Conventions

### Card Identity

Every command should carry these keys in internal YAML:

- `card_name`
- `issuer`
- `network`
- `card_family`
- `card_variant`

### Sources And Confidence

Every command should carry:

- `sources`: list of `{name, url, type, used_for}`
- `confidence_notes`: list of short strings explaining ambiguity, freshness concerns, or conflicts

### Status Values

Use only:

- `confirmed`
- `unconfirmed`
- `conflicting`

## Collection Item Shape

When a command returns lists of benefits, partners, rates, updates, or credits, use objects rather than prose blobs. Each object should include:

- `title`
- `summary`
- `status`
- `source_name`
- `source_url`

Add optional fields only when the command needs them, such as:

- `value`
- `cadence`
- `cap`
- `conditions`
- `ratio`
- `timing`
- `category`
- `enrollment_required`
- `time_window`

## Formatting Rules

- YAML is an internal schema and test artifact only. Never include YAML in user-facing output.
- Do not show inline links or a sources footer in user-facing output.
- Use one emoji per section heading (e.g., `## 💰 Fees`).
- Use numbered lists for list-heavy sections (credits, transfer partners, earning categories, news items).
- When listing credits, fees, or any monetary amounts, sort from highest to lowest dollar value.
- Keep section content to condensed facts. No prose padding.
- Omit the Card Identity section when the card match is confident.
- Show Card Identity only when the match is ambiguous or unconfirmed.
- Empty lists are acceptable when a category truly does not apply.
- Do not invent missing values.

## Disambiguation Rules

- If the card name is ambiguous, return a numbered card-choice list and stop. Wait for the user to pick one.
- If no confident match exists, return a short "be more specific" failure message.

## Conflict Rules

- Prefer issuer pages for benefits, rates, fees, transfer definitions, and official terms.
- For public offers and news context, use approved secondary sources when they are more current or add necessary material context.
- If sources disagree, preserve the disagreement and explain it in `confidence_notes`.
- Do not silently merge conflicting values into one answer.
