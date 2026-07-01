---
name: echo-shield-card-only
description: Use when a Bankr or agent user gives a Base token contract and wants only the Echo Shield card. Return exactly one plain Card URL line; never analyze, explain, fetch PNG bytes, use Markdown image syntax, run curl, or leave Card blank.
version: 1
author: BuiltByEcho
license: MIT
tags: [echo-shield, base, launch-card, bankr, card-only]
visibility: public
metadata:
  clawdbot:
    emoji: "🛡️"
    homepage: "https://shield.builtbyecho.xyz"
---

# Echo Shield Card Only

## Output Contract

For a valid Base contract address, output exactly one line:

```txt
Card: https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

No summary. No explanation. No Markdown image. No curl command. No transaction. No scan.

## Address Rule

A valid address matches:

```txt
^0x[a-fA-F0-9]{40}$
```

If there is no valid address in the current message or conversation context, output exactly:

```txt
Send the Base token contract address.
```

## Workflow

1. Find the most recent valid `0x` address.
2. If no address exists, return only: `Send the Base token contract address.`
3. If an address exists, build the deterministic URL locally. Do not call any API.
4. Return only: `Card: <URL>`.

## Forbidden Outputs

Never output:

- `Card:` with nothing after it.
- Any terminal-preview explanation.
- Any download command with no URL.
- Markdown image syntax.
- Any scan summary, risk score, liquidity/holder explanation, or transaction step.
