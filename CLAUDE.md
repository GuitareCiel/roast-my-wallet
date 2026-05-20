# Roast My Wallet (rmw)

A Claude Code skill that turns an AI coding agent into a wallet diagnostician. The agent reads a Ledger-discovered crypto portfolio via `@ledgerhq/wallet-cli`, runs a seven-sins structural diagnostic, delivers a brutal-but-fair 3-5 sentence roast with a 1-5 score, prints a target allocation table + long/spot/DeFi verdict, and gates the first concrete rebalance trade behind two explicit user confirmations plus a device approval.

## Current state

Single artifact: `skills/roast-my-wallet/SKILL.md`. That's the whole product.

Distribution is **manual / via curl** from the GitHub repo — no npm package, no CLI, no installer. The strategic point is exactly this minimalism: wallet-cli is so well-shaped that an AI agent can drive it directly with a single markdown file. The launch pitch is "two installs, one skill file, no SDK."

## Repo layout

```
.
├── CLAUDE.md
├── LICENSE
├── README.md                          # the showcase + install instructions
├── .gitignore
└── skills/
    └── roast-my-wallet/
        └── SKILL.md                   # the canonical product
```

## How it actually runs

- User has `@ledgerhq/wallet-cli` installed globally (separate step).
- User has dropped `skills/roast-my-wallet/SKILL.md` into `~/.claude/skills/roast-my-wallet/SKILL.md`.
- In Claude Code, "roast my wallet" triggers the skill via the frontmatter `triggers:` list.
- Claude Code reads the SKILL.md body and orchestrates: pre-flight checks → discovery (if session empty) → balances + operations → CoinGecko prices → assemble portfolio → write roast → render Verdict block → quote → execute (two-yes gated).
- The Verdict block is drawn by Claude itself; the SKILL.md pins the dimensions (62 cols total, 56 content, continuation indent under first wrapped char).

## Source of truth

- **Voice + orchestration:** `skills/roast-my-wallet/SKILL.md` — frontmatter (name, triggers, allowed-tools) + body (pre-flight, data gathering, voice prompt, output sequencing, safety contract). Touch this file to tune anything.

## wallet-cli reality (probed against 1.0.1)

Pinned in the SKILL.md. If the installed version is newer, re-probe `wallet-cli <cmd> --help` before trusting any command/flag:

- `account discover --network <name>` (no `account list`)
- `session view` to enumerate; subsequent calls use `--account <label>`
- `balances` / `operations` need no device (use cached public addresses)
- `swap quote` is the no-signing preview (no `--dry-run` flag on swap)
- `swap execute` requires the device
- Three response shapes: `{ok:true,data}` for meta, flat `{status:"success",...}` for data, `{ok:false,error}` for errors
- Currency IDs not tickers: `bitcoin`, `ethereum`, `solana`, `ethereum/erc20/usd__coin` (USDC, double underscore)
- swap `--amount` takes a bare number; send `--amount` is ticker'd
- Provider list: `changelly_v2` (default), `changelly`, `cic_v2`, `cic`, `exodus`, `nearintents`, `swapsxyz`

## What lives outside this repo

- `~/.claude/skills/roast-my-wallet/SKILL.md` — user's installed copy. Stays in sync manually (or via curl).
- wallet-cli's session store (e.g. `~/.ledger/`) — persists across runs; xpubs cached after one-time discovery.
- `npm i -g @ledgerhq/wallet-cli` — global wallet-cli install. Required.
