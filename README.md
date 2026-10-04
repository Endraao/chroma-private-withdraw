# Chroma — Private creator withdrawal (Cloak)

Built for the **Superteam Brasil Privacy Week** (Sep 30 → Oct 5, 2026), on top of
[Chroma Launchpad](https://chromalaunch.fun), a Solana memecoin launchpad where coin
creators earn 40% of every trade fee.

## The problem

The creator wallet is the most watched wallet on Solana. Holders, bots and wallet trackers
follow every move: when a dev moves their earnings, the community panics ("dev is selling"),
and the dev's personal wallet gets exposed.

## What we built

A **"Withdraw privately"** button inside the creator earnings card (shown only to the wallet
that created the coin). In one flow it:

1. claims the creator's trading fees (Meteora DBC / DAMM v2, signed by the creator);
2. deposits SOL into **Cloak's shielded pool** (`transact`, signed by the wallet);
3. withdraws to any wallet the creator chooses with a **zero-knowledge proof** generated in
   the browser (`fullWithdraw`, authenticated by a wallet message signature and submitted by
   the Cloak relay — so the creator wallet does not appear in the outgoing transaction).

The deposit note is saved in the browser before anything is sent, so a closed tab can be
resumed ("Finish send"); if the deposit never landed, the note is discarded automatically.

## Proof on Solana mainnet

- Deposit (creator wallet → Cloak pool):
  [`3PjVFmUc…gagHeeV`](https://solscan.io/tx/3PjVFmUcEJga5wZPYbfqhZHQiTgHR5qAmTsEM5UpSys1kqAx6Ky1UU7HQkDFJBwV6LaEXd5ouwzzfnWBZgagHeeV)
- Withdrawal (Cloak pool → destination wallet, creator wallet not present):
  [`4paTT9ww…nuvinimK`](https://solscan.io/tx/4paTT9wwpwhvrtKGppKyzLtUB64GkmVdRLYKm92z646uELsk47kv6cPSch9AS51wxkyyQYnvLJ3NKv2knuvinimK)

## Files

| File | What it does |
| --- | --- |
| `src/lib/saque-privado.ts` | Cloak flow: deposit, ZK withdrawal, pending-note storage and recovery |
| `src/components/trading/SaquePrivado.tsx` | The "Withdraw privately" panel (EN / PT / ZH) |
| `src/components/trading/GanhosDoCriador.tsx` | Creator earnings card that hosts it (fee claim + bonus) |
| `src/app/api/rpc/solana/route.ts` | RPC proxy that absorbs 429 bursts from the proof flow |
| `src/lib/security-headers.mjs` | CSP: allows the Cloak relay, circuits and WebAssembly (no `unsafe-eval`) |

These files are excerpted from the Chroma app (Next.js 16, `@cloak.dev/sdk@0.2.5`).
Live: https://chromalaunch.fun — open any Chroma Curve coin you created.
