---
name: echo-shield-token-triage
description: Use when a Bankr or agent user gives a Base token contract and wants the Echo Shield card only. Return just the direct launch-card PNG URL; do not summarize, deep scan, trade, fetch binary image bytes, or execute transactions unless explicitly asked.
version: 1.2.0
author: BuiltByEcho
license: MIT
tags: [echo-shield, base, token-safety, launch-card, bankr, card-only]
visibility: public
metadata:
  hermes:
    tags: [echo-shield, base, token-safety, agents, launch-card, card-only]
    related_skills: []
---

# Echo Shield Card Only

## Overview

This skill exists for one narrow job: given a Base ERC-20 contract address, return the Echo Shield launch-card PNG URL and nothing else.

Default behavior is **card only**. Do not perform a full token triage, do not explain the score, do not call deep-analysis, do not fetch or paste PNG bytes, and do not execute any transaction. The launch-card endpoint itself is the image URL.

## Output Contract

For a valid address, respond with exactly one visible card line:

```txt
Card: https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

If the host supports Markdown image previews and the user specifically asks to embed/display the image, use exactly:

```md
![Echo Shield card](https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png)
```

Do not add score summaries, bullets, caveats, liquidity notes, holder notes, or buy/sell commentary unless the user explicitly asks for analysis after receiving the card.

## Address Rule

Only accept Base token contract addresses matching:

```txt
^0x[a-fA-F0-9]{40}$
```

If the input is not a valid address, ask for the Base token contract address in one sentence.

## Workflow

1. **Validate address.** Confirm the user provided one `0x` address with 40 hex characters. Completion: invalid input is rejected before any URL is returned.
2. **Build URL locally.** Construct the deterministic URL: `https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png`. Completion: no scan/deep-analysis/API call is required for the default card-only answer.
3. **Return only the card line.** Output exactly `Card: <URL>` and stop. Completion: no extra explanation or risk summary appears after the card line.

## Optional Scan-Specific URL

Only if the user explicitly asks for a fresh scan-specific/cache-busted card URL, call:

```txt
GET https://shield.builtbyecho.xyz/api/scan?address=<ADDRESS>
```

Then return the first available field, in this order:

1. `launchCardPng`
2. `launchCardPngUrl`
3. `cardUrl`
4. `cardPngUrl`
5. `card.cardUrl`

If no field exists, fall back to:

```txt
https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

Even in this optional mode, return only:

```txt
Card: <URL>
```

## Raw PNG Handling

The PNG endpoint returns binary image data. That is correct. Do not fetch the PNG bytes in text-only chat. Do not paste byte output. Do not say the card URL is blank. The URL itself is the displayable artifact.

If a user wants to download the file, provide only this command:

```bash
curl -L "https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png" --output echo-shield-card.png
```

## Do Not Do These

- Do not call `/api/deep-analysis` for card-only requests.
- Do not execute wallet transactions.
- Do not look up token holder/liquidity/admin details unless explicitly asked after the card.
- Do not return a long report.
- Do not leave a blank `Card:` line.
- Do not fetch the image bytes and describe terminal limitations unless the user asks why a terminal cannot render it.

## Verification Checklist

- [ ] Address is valid before returning a URL.
- [ ] Default answer requires zero API/tool calls by the Bankr agent.
- [ ] Output is exactly one `Card: <URL>` line for normal requests.
- [ ] URL contains `format=png`.
- [ ] No deep analysis, transaction, summary, or binary image bytes are returned.
