---
name: echo-shield-token-triage
description: Use when an agent needs to triage a Base ERC-20 token with Echo Shield. Run the quick scan first, put the direct PNG card URL near the top, summarize score/card/liquidity/holders, and only run deep analysis after explicit user intent.
version: 1.1.0
author: BuiltByEcho
license: MIT
tags: [echo-shield, base, token-safety, launch-card, bankr]
visibility: public
metadata:
  hermes:
    tags: [echo-shield, base, token-safety, agents, quick-scan, launch-card]
    related_skills: [echo-shield-launch-card]
---

# Echo Shield Token Triage

## Overview

Echo Shield is a public-data scanner for Base ERC-20 contracts. It produces an Echo risk score, source status, token metadata, holder/liquidity signals, and share-card metadata. It does not trade, connect wallets, custody funds, sign transactions, or make buy/sell calls.

The core discipline is **quick first, deep on intent**. A quick scan is suitable for first responses and social/card previews. A deep scan is slower and should only run when the user explicitly asks for deeper review.

For Bankr, Discord, Telegram, X, and other truncating clients, the **direct PNG card URL must appear near the top**. If the PNG endpoint returns raw binary image bytes, do not try to paste or render the bytes in text. Give the displayable URL instead.

## When to Use

Use this skill when:

- A user gives a Base token contract and asks “is this safe?”, “scan this,” “make a card,” or “what does Echo think?”
- A bot needs a concise token-safety reply in Discord, Telegram, Bankr, X, or another agent surface.
- A workflow needs a public launch-hygiene card plus structured JSON.

Do not use this skill for:

- Non-Base contracts unless Echo Shield adds chain support.
- Wallet balance checks, trading, swaps, approvals, or custody.
- Legal, audit, or buy/sell advice.

## Endpoints

Base URL:

```txt
https://shield.builtbyecho.xyz
```

| Intent | Endpoint | Rule |
| --- | --- | --- |
| Quick triage | `GET /api/scan?address=0x...` | Default first call. Returns JSON and card URL metadata. |
| Full review | `GET /api/deep-analysis?address=0x...` | Only after explicit user intent. |
| PNG card | `GET /api/launch-card?address=0x...&format=png` | Direct image endpoint. Returns raw PNG bytes, not text/base64. Use as URL for previews. |
| Cohorts | `GET /api/launch-cohorts` | Use for aggregate launch-surface context. |

## Card URL Rule

Always resolve a displayable card URL before writing the final response.

Preferred order:

1. Use the top-level `reply` from `/api/scan` verbatim if it already includes a visible card URL near the top.
2. Otherwise use the first available URL from:
   - `launchCardPng`
   - `launchCardPngUrl`
   - `cardUrl`
   - `cardPngUrl`
   - `card.cardUrl`
3. If every URL field is missing, build the deterministic fallback:

```txt
https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

If `/api/scan` returns a cache-busted URL such as:

```txt
https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png&scan=<REPORT_SCANNED_AT>
```

preserve the full URL, including `scan=`, so clients do not show stale cards.

### Raw PNG Handling

The PNG endpoint returns binary image data. In terminal/text-only contexts, this may appear as “raw binary PNG data” or a large byte count. That is expected.

Do **not** respond with:

```txt
Here is the direct card URL:
```

followed by a blank line.

Do respond with an actual URL, for example:

```txt
Card: https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

If the user needs a file, tell them to save it:

```bash
curl -L "https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png" --output echo-shield-card.png
```

## Workflow

1. **Validate address.** Confirm the input matches `^0x[a-fA-F0-9]{40}$`. Completion: invalid input is rejected before any network call.
2. **Run quick scan.** Call `/api/scan?address=<address>` once. Completion: response has `ok: true`, `depth: "quick"`, `report.analysisDepth: "quick"`, and either a top-level `reply` or at least one card URL field.
3. **Put the card URL near the top.** Prefer the response `reply` if it already contains the direct card URL. Otherwise put the card URL on its own line immediately after the score/risk line, before detailed findings. Completion: the card URL remains visible even when downstream clients truncate long scan details.
4. **Summarize compactly.** Include score/risk, liquidity, holders, launch surface, key red flags, and source availability. Completion: the user can understand the risk posture without opening the full JSON.
5. **Gate deep analysis.** Do not call `/api/deep-analysis` unless the user asks for deeper review or presses a dedicated button. Completion: expensive provider calls are not made for passive previews.
6. **Run deep analysis when requested.** Call `/api/deep-analysis?address=<address>`. Completion: response has `depth: "deep"` and `report.analysisDepth: "deep"`.
7. **Explain evidence.** Summarize bucket scores, Honeypot/tax simulation, holder concentration/Echo Map, website checks, source status, and scan-memory changes. Completion: deep summary names both findings and source availability.
8. **Close with caveat.** State that the result is public-data triage, not an audit or trading advice. Completion: no message implies guaranteed safety or a buy/sell call.

## Bankr-Ready Response Template

```txt
Echo Shield quick scan for SYMBOL — SCORE/100, LEVEL risk.
Card: CARD_URL

Summary: ...
Liquidity: ... · Holders: ... · Launch surface: ...
Key flags: ...

Public-data triage only — not an audit or buy/sell recommendation.
```

If terminal output cannot render the image:

```txt
Echo Shield generated the launch card as a PNG image.
Card: CARD_URL

This endpoint returns raw PNG bytes in a terminal. Open the URL in a browser or embed it in a client that supports image previews.
```

For deep analysis:

```txt
Deeper Echo Analysis:
Card: CARD_URL

- Contract: ...
- Honeypot/taxes: ...
- Holders/Echo Map: ...
- Website/source status: ...
- Scan memory: ...

Public-data triage only — not an audit or buy/sell recommendation.
```

## Common Pitfalls

1. **Calling deep automatically.** This wastes provider/API budget. Quick scan first; deep only on intent.
2. **Double-calling card endpoints.** `/api/scan` already returns top-level card URL aliases and `card.cardUrl`. Use those before making a separate `/api/launch-card` call.
3. **Treating raw PNG bytes as text.** The PNG endpoint returns binary image bytes. Never paste raw bytes or claim there is a URL without printing it.
4. **Burying or blanking the card URL.** Prefer `reply`; otherwise use `launchCardPng`, `launchCardPngUrl`, `cardUrl`, `cardPngUrl`, then `card.cardUrl`. If all are missing, construct `https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png` rather than leaving the line blank.
5. **Dropping the cache-buster.** Preserve `scan=` in card URLs so previews do not show stale score/holder images.
6. **Overstating safety.** Say “no major automated red flags surfaced,” not “safe.”
7. **Ignoring partial sources.** If a source says `partial` or `unavailable`, surface that honestly.
8. **Retry spam.** On `502 provider_failure`, report provider failure and retry later instead of looping.

## Verification Checklist

- [ ] Address validated before request.
- [ ] First request is `/api/scan`, not `/api/deep-analysis`.
- [ ] Quick response has `depth: "quick"` and either `reply` or a card URL alias (`launchCardPng`, `launchCardPngUrl`, `cardUrl`, `cardPngUrl`, or `card.cardUrl`).
- [ ] Card URL appears immediately after the score/risk line, or the top-level `reply` field is used verbatim and includes a card URL near the top.
- [ ] If the PNG endpoint returns raw bytes, the final text gives the URL and does not try to render/paste binary.
- [ ] Deep analysis only runs after explicit intent.
- [ ] Summary includes score/risk, liquidity, holders, launch surface, key flags, and caveat.
- [ ] Source status and unavailable providers are not hidden.
- [ ] Card URLs keep `format=png` and preserve `scan=` when provided.
