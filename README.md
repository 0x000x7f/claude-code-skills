# claude-code-skills

📖 **English** | [日本語](README.ja.md)

Personal [Claude Code](https://claude.com/claude-code) skills I actually run.

## `grill` / `grill-deep` — stress-test a plan before you build

A two-tier "grilling" interview. It interrogates a plan **one question at a time** — each question carrying *why it matters*, *the risk it removes*, and *a recommended default* — until the design is precise enough to execute.

| Command | Weight | Use it for |
|---|---|---|
| `/grill` | light | small / medium tasks — the base interview plus the one non-negotiable guardrail |
| `/grill-deep` | full | high-stakes work (infra, security, irreversible changes, data handling, anything you'll keep) — adds artifact inspection, premise-challenge, a cross-cutting risk sweep, and a completeness pass |

Both are **manual-only** (`disable-model-invocation: true`), so they never fire on their own — you decide when a plan is worth grilling.

## Why two tiers — the verification gap

This started as a copy of Matt Pocock's [`grilling`](https://github.com/mattpocock/skills) skill. Before trusting it, I A/B-tested the stock skill against an enhanced version on a real plan and fact-checked every technical claim against official docs.

The stock skill was **strong — 15 / 15 technical claims correct.** But it had no *verification step*: it asserts mechanisms from memory with no way to tell a checked fact from a confident guess. In one run it stated, with full confidence, a **wrong** mechanism — "Looker Studio binds a sheet by `gid`" (it actually binds by **tab name**). Nothing flagged it. "Right this time" is luck, not method.

So the enhanced version adds five guardrails. The highest-leverage one:

> **Verify a decision-driving claim, or tag it `[UNVERIFIED]`** — never let a confident guess masquerade as a checked fact.

`grill` keeps just that core; `grill-deep` runs all five (also: inspect the artifact not its shape, challenge the premise, sweep cross-cutting risks like privacy / idempotency / quotas / secrets, finish with a "what did we miss?" pass).

Method and full results: [docs/grilling-ab-test.md](docs/grilling-ab-test.md).

## Install

Copy a skill folder into your personal skills directory:

```sh
cp -r skills/grill       ~/.claude/skills/grill
cp -r skills/grill-deep  ~/.claude/skills/grill-deep
```

Restart Claude Code if the command doesn't appear yet, then run `/grill` or `/grill-deep`.

## Credit

The `grilling` interview pattern is from [mattpocock/skills](https://github.com/mattpocock/skills). The verification + risk-sweep guardrails and the light / deep split are mine.
