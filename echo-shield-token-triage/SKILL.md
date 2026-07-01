---
name: echo-shield-token-triage
description: Use when a Bankr or agent user gives a Base token contract and wants only the Echo Shield launch-card URL. Build and return exactly one plain Card URL line; never explain PNG rendering, never use Markdown images, never output curl commands, and never leave Card blank.
version: 2
author: BuiltByEcho
license: MIT
tags: [echo-shield, base, token-safety, launch-card, bankr, card-only]
visibility: public
metadata:
  clawdbot:
    emoji: "🛡️"
    homepage: "https://shield.builtbyecho.xyz"
  hermes:
    tags: [echo-shield, base, token-safety, agents, launch-card, card-only]
    related_skills: []
---

# Echo Shield Card Only

## Purpose

Return the Echo Shield card URL only.

This skill is intentionally tiny because Bankr can hit step limits or drop URLs when the agent tries to analyze, download, or explain the image. The card endpoint is already the artifact.

## Hard Output Contract

For any valid Base contract address, output exactly one line:

```txt
Card: https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

No other text.

## If the User Asks “Why won’t the card show?”

Do not explain terminal image rendering. Do not output `curl`. Do not output a blank `Card:` line.

If a valid address is available in the current message or conversation context, return exactly:

```txt
Card: https://shield.builtbyecho.xyz/api/launch-card?address=<ADDRESS>&format=png
```

If no valid address is available, ask exactly:

```txt
Send the Base token contract address.
```

## Address Rule

A valid address matches:

```txt
^0x[a-fA-F0-9]{40}$
```

If multiple addresses are present, use the most recent one from the user.

## Workflow

1. Find the most recent valid `0x` address.
2. If no address exists, return only: `Send the Base token contract address.`
3. If an address exists, build the URL locally. Do not call `/api/scan`; do not call `/api/deep-analysis`; do not fetch the PNG bytes.
4. Return only the `Card: <URL>` line.

## Forbidden Outputs

Never output any of these patterns:

```txt
Card:
```

- any terminal/image-preview explanation

- an empty download command with no URL
- Markdown image syntax

## Verification Checklist

- [ ] Output is one line for a valid address.
- [ ] Output starts with `Card: https://shield.builtbyecho.xyz/api/launch-card?address=`.
- [ ] Output includes `&format=png`.
- [ ] Output does not include explanations, Markdown image syntax, curl commands, scan summaries, or transactions.
- [ ] Output never contains a blank `Card:` line.
