---
name: echo-shield-x-preview
description: Use when a Bankr or agent user wants to test an Echo Shield card on X/Twitter privately. Return the HTML social-card wrapper URL for X previews, with a cache-busting x_test value when requested.
version: 1
author: BuiltByEcho
license: MIT
tags: [echo-shield, x, twitter, social-card, bankr, private-test]
visibility: public
metadata:
  clawdbot:
    emoji: "🛡️"
    homepage: "https://shield.builtbyecho.xyz"
---

# Echo Shield X Preview

## Output Contract

For a valid Base contract address, output exactly this line:

```txt
X preview: https://shield.builtbyecho.xyz/api/card?address=<ADDRESS>&x_test=<CACHE_BUSTER>
```

Use the current timestamp or a short random suffix as `<CACHE_BUSTER>` unless the user provides one.

## Why This URL

X/Twitter should receive the HTML wrapper at `/api/card`, not the raw PNG at `/api/launch-card`.

The wrapper includes:

- `twitter:card=summary_large_image`
- `twitter:image` pointing to the PNG card
- `og:image`
- image width/height metadata

The `x_test` parameter busts X card cache. The live wrapper passes that value through to the image URL as `scan=<CACHE_BUSTER>`.

## Private Test Steps

If the user asks how to test privately, output only:

```txt
Paste this into an X draft or protected test-account post, but do not publish from your main account: https://shield.builtbyecho.xyz/api/card?address=<ADDRESS>&x_test=<CACHE_BUSTER>
```

## Address Rule

A valid address matches:

```txt
^0x[a-fA-F0-9]{40}$
```

If there is no valid address, output exactly:

```txt
Send the Base token contract address.
```

## Do Not Do These

- Do not return the raw PNG URL for X testing.
- Do not use Markdown image syntax.
- Do not fetch the PNG bytes.
- Do not summarize risk, liquidity, holders, or trading advice.
