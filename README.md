# 💳 Credit Card Skills

![Credit Card Skills](social-preview.png)

> Research any major US credit card in ~2 minutes — earning rates, transfer partners, credits, news, comparisons, and value estimates. Works with Claude Code, OpenAI Codex, Cursor, Gemini CLI, and any [Agent Skills](https://agentskills.io)-compatible tool.

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-4f46e5?style=flat-square)](https://agentskills.io)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-D97706?style=flat-square)](https://code.claude.com)
[![OpenAI Codex](https://img.shields.io/badge/OpenAI%20Codex-ready-10a37f?style=flat-square)](https://developers.openai.com/codex)
[![License: MIT](https://img.shields.io/badge/License-MIT-gray?style=flat-square)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0.0-blue?style=flat-square)](.claude-plugin/plugin.json)

---

## Install

Run from your project root — auto-detects your agent:

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/jiahongc/credit-card-skills/main/install.sh)
```

<details>
<summary>Target a specific agent</summary>

```bash
# Claude Code only  → .claude/skills/
bash <(curl -fsSL https://raw.githubusercontent.com/jiahongc/credit-card-skills/main/install.sh) --claude

# OpenAI Codex only → .agents/skills/
bash <(curl -fsSL https://raw.githubusercontent.com/jiahongc/credit-card-skills/main/install.sh) --codex

# Both
bash <(curl -fsSL https://raw.githubusercontent.com/jiahongc/credit-card-skills/main/install.sh) --all
```

</details>

<details>
<summary>Claude Code plugin (no install needed)</summary>

```bash
claude --plugin-dir /path/to/credit-card-skills
```

Skills load with a namespace: `/credit-card-skills:card-full`, etc.

</details>

---

## Setup (Optional)

For faster search results, set a [Brave Search API](https://brave.com/search/api/) key:

```bash
cp .env.example .env
# Edit .env and add your BRAVE_API_KEY
```

Free tier gives 2,000 queries/month. Without it, commands fall back to built-in web search (slower).

---

## Commands

### Single-card

| Command | Description |
|---------|-------------|
| `/card-full [card]` | Full report — fees, offer, earnings, credits, benefits, protections, eligibility, strategy |
| `/card-rate [card]` | Earning rates by category with caps and exclusions |
| `/card-transfer [card]` | Transfer partners with ratios, timing, and restrictions |
| `/card-credits [card]` | Credits with amount, cadence, trigger, and restrictions |
| `/card-news [card]` | Material changes from the last 3 months |

### Multi-card

| Command | Description |
|---------|-------------|
| `/card-compare [card A] vs [card B]` | Side-by-side: fees, earning rates, credits, transfer partners, benefits |
| `/card-value [card] [optional spend]` | First-year value: welcome bonus + earn + credits − annual fee |
| `/card-wallet [card1], [card2], ...` | Wallet audit: earning map, credit stack, overlaps, gaps |

---

## Example Output

<details>
<summary><code>/card-rate Citi Double Cash</code></summary>

```
## 📊 Rate Summary

- Base rate: 2% cash back on all purchases (1% when you buy + 1% as you pay)
- Point currency: Citi ThankYou Points (1 point = 1 cent at standard redemption)
- Key mechanic: second 1% earned only when you pay at least the minimum due on time

## 📈 Earning Categories

1. All purchases — 2% cash back (1% at purchase + 1% at payment)
2. Hotels, car rentals, and attractions via Citi Travel portal — 5% total cash back
3. Balance transfers, cash advances — 0%

## 🚫 Caps And Exclusions

- No annual or monthly cap on the base 2% earning rate
- The second 1% requires paying at least the minimum due on time
- The 5% travel rate applies only to the Citi Travel portal, not direct bookings

## 📋 Confidence Notes

- 5% Citi Travel portal rate confirmed via citi.com
- ThankYou Points redemption value varies — as low as 0.8 cpp for Amazon
```

</details>

<details>
<summary><code>/card-compare Amex Gold vs Chase Sapphire Preferred</code></summary>

```
## 💰 Fees

|                        | Amex Gold | Chase Sapphire Preferred |
|------------------------|-----------|--------------------------|
| Annual fee             | $325      | $95                      |
| Foreign transaction    | None      | None                     |
| Net after credits      | ~$1/yr    | ~$45/yr                  |

## 📈 Earning Rates

| Category            | Amex Gold              | Chase Sapphire Preferred      |
|---------------------|------------------------|-------------------------------|
| Dining              | 4x MR                  | 3x UR                         |
| U.S. supermarkets   | 4x MR (up to $25k/yr)  | 1x UR                         |
| Travel (portal)     | 3x MR                  | 5x UR (Chase Travel)          |
| Streaming           | 1x MR                  | 3x UR                         |

## 🏆 Bottom Line

| Dimension              | Winner                                        |
|------------------------|-----------------------------------------------|
| Lower annual fee       | Chase Sapphire Preferred ($95 vs $325)        |
| Dining & groceries     | Amex Gold (4x vs 3x / 4x vs 1x)              |
| Travel earn rate       | Chase Sapphire Preferred (5x portal)          |
| Statement credits      | Amex Gold ($424 vs ~$50)                      |
| Transfer partner reach | Amex Gold (20 partners vs 13)                 |
| Hotel transfer quality | Chase Sapphire Preferred (World of Hyatt)     |
| Travel protections     | Chase Sapphire Preferred                      |
```

</details>

<details>
<summary><code>/card-news Capital One Venture X</code></summary>

```
## 📅 News Window

- 2025-12-12 to 2026-03-12 (90 days)

## 📰 Recent Updates

1. Elevated 100K welcome bonus expired (Jan 5, 2026) — reverted to 75K after $4K spend
2. Lounge guest access removed (Feb 1, 2026) — guests now $45 each; free only after $75K/yr spend
3. Authorized user lounge access now paid (Feb 1, 2026) — $125/yr per AU
4. Priority Pass guest privileges revoked (Feb 1, 2026) — guests now $35 each
5. Discover network migration begins for other Capital One cards — Venture X stays on Visa

## 📝 Summary

February 2026 brought significant lounge access downgrades. Core earning, annual fee,
and travel credits remain unchanged.
```

</details>

---

## How It Works

- **Issuer-first**: always checks the card's official product page before secondary sources
- **Fast**: uses Brave Search API for ~1s lookups instead of slow page fetches; knowledge-first commands answer in seconds
- **Deep**: search-required commands pull up to 5 secondary sources; knowledge-first commands use 1 secondary for cross-checks
- **Compact**: emoji section headings, numbered lists, no prose padding
- **Honest**: unresolved fields are flagged in confidence notes, never invented
- **6 curated sources**: NerdWallet, The Points Guy, Doctor of Credit, Bankrate, One Mile at a Time, Upgraded Points (see [`source-policy.yaml`](.claude/skills/card-shared/source-policy.yaml))

---

## Compatibility

Follows the [Agent Skills](https://agentskills.io) open standard — one `SKILL.md` format, works across agents.

| Agent | Skill path |
|-------|-----------|
| Claude Code | `.claude/skills/` |
| OpenAI Codex | `.agents/skills/` |
| Cursor | `.cursor/skills/` |
| Gemini CLI | `.gemini/skills/` |
| ClawHub | [`clawhub/`](clawhub/) (self-contained, ready to publish) |
| Others | see [agentskills.io](https://agentskills.io) |

---

## Shared Layer

All commands share a single research policy and output contracts:

| File | Purpose |
|------|---------|
| [`source-policy.yaml`](.claude/skills/card-shared/source-policy.yaml) | Approved sources, issuer domains, fast-search budget |
| [`command-contracts.yaml`](.claude/skills/card-shared/command-contracts.yaml) | Required sections and YAML schema per command |
| [`card-identity-rules.md`](.claude/skills/card-shared/card-identity-rules.md) | Card-name resolution and variant disambiguation |
| [`confidence-rules.md`](.claude/skills/card-shared/confidence-rules.md) | `confirmed` / `unconfirmed` / `conflicting` definitions |
| [`recency-rules.md`](.claude/skills/card-shared/recency-rules.md) | 3-month news window rules |
| [`normalization-rules.md`](.claude/skills/card-shared/normalization-rules.md) | Shared formatting conventions |

---

## Validate Fixtures

```bash
./tests/run_all.sh
```

Runs 12 fixture validations + composition check + cross-fixture consistency check. See [`tests/`](tests/) for golden, ambiguous, conflict, missing, and recency fixture sets.

---

## Notes

- Major US card brands (Amex, Chase, Capital One, Citi, Bank of America, Bilt, Discover, Robinhood, Wells Fargo)
- `/card-news` is intentionally independent from `/card-full` — news is time-windowed and conceptually separate
- YAML output contracts are internal only and never appear in user-facing output
- See [`sample-outputs.md`](sample-outputs.md) for 10 real example outputs across all 8 commands
