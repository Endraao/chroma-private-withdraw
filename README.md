# Chroma — Meteora DBC launchpad + private creator withdrawals (Cloak)

Excerpts from [Chroma Launchpad](https://chromalaunch.fun), a Solana memecoin launchpad where
**coin creators earn 40% of every trade fee** — built on **Meteora's Dynamic Bonding Curve** —
plus a cash bonus at volume goals and **private withdrawals through Cloak's shielded pool**.

Live: https://chromalaunch.fun · Rules: https://chromalaunch.fun/creator-bonus

---

## 1. The Chroma Curve (Meteora Dynamic Bonding Curve)

`src/lib/meteora-dbc.ts`

- **Our own DBC config** (`BbbaZGSkhjFMfFQNfUrgbDFX9pWATSZvWoVF9jkaFQXw`), built with
  `buildCurveWithMarketCap`: 30 → 420 SOL market cap, SPL token, immutable authority.
- **Anti-sniper fee**: `FeeSchedulerExponential` 25% → 1% over 120 s, with
  `enableFirstSwapWithMinFee` so the **creator's first buy pays the minimum fee**.
- **One-signature launch**: `creator.createPoolWithFirstBuy` — coin + pool + creator first buy
  in a single transaction (simulated before the wallet signs).
- **Fee split**: `creatorTradingFeePercentage: 50` → after Meteora's protocol share, the
  creator and Chroma each get ~0.4% of all volume, **on any app** (Jupiter, Phantom, bots),
  because the fee is charged by the curve itself.
- **Graduation** to a **DAMM v2** pool (`FixedBps100`), liquidity permanently locked 50/50
  creator/partner — creators keep earning after graduation.
- **Creator claims** (`transacoesDeSaqueDoCriador`): `claimCreatorTradingFee` on the curve +
  `claimPositionFee` on the creator's DAMM v2 position, packed into as few transactions as fit
  1232 bytes and simulated before signing.
- **Partner claims** (`transacoesDeResgate`, `transacoesDeResgateNaMeteora`): pool-creation fee,
  partner trading fee and DAMM v2 position fees for the platform wallet.

## 2. Creator bonus

`src/lib/bonus-criador.ts`, `src/app/api/bonus-criador/route.ts`, `src/app/creator-bonus/page.tsx`

Progress is read **on-chain**: Chroma's partner fee on the curve + half of the DAMM v2 LP fees
after graduation, converted to volume. Goals: $25K / $100K / $250K / $500K → total bonus
$50 / $200 / $500 / $1,000. Funded from Chroma's own fee share (half of it), always paid to the
wallet that created the coin.

## 3. Private creator withdrawals (Cloak)

Built for the **Superteam Brasil Privacy Week** (Sep 30 → Oct 5, 2026).

The creator wallet is the most watched wallet on Solana: when a dev moves earnings, the
community panics and the dev's personal wallet gets exposed. **"Withdraw privately"**
(`src/lib/saque-privado.ts`, `src/components/trading/SaquePrivado.tsx`):

1. claims the creator's fees (above);
2. deposits SOL into **Cloak's shielded pool** (`transact`, signed by the wallet);
3. withdraws to any wallet the creator chooses with a **zero-knowledge proof** generated in the
   browser (`fullWithdraw`, authenticated by a wallet message signature and submitted by the
   Cloak relay — the creator wallet does not appear in the outgoing transaction).

The deposit note is saved in the browser before anything is sent; a closed tab can be resumed,
and a note whose deposit never landed is discarded automatically.

**Proof on Solana mainnet**

- Deposit (creator wallet → Cloak pool):
  [`3PjVFmUc…gagHeeV`](https://solscan.io/tx/3PjVFmUcEJga5wZPYbfqhZHQiTgHR5qAmTsEM5UpSys1kqAx6Ky1UU7HQkDFJBwV6LaEXd5ouwzzfnWBZgagHeeV)
- Withdrawal (Cloak pool → destination, creator wallet not present):
  [`4paTT9ww…nuvinimK`](https://solscan.io/tx/4paTT9wwpwhvrtKGppKyzLtUB64GkmVdRLYKm92z646uELsk47kv6cPSch9AS51wxkyyQYnvLJ3NKv2knuvinimK)

---

## Files

| File | What it does |
| --- | --- |
| `src/lib/meteora-dbc.ts` | Chroma Curve: DBC config, one-signature launch, curve state, creator/partner claims on DBC and DAMM v2 |
| `src/lib/bonus-criador.ts` | Creator bonus goals and progress |
| `src/app/api/bonus-criador/route.ts` | Bonus progress API (on-chain) + bonus requests |
| `src/app/creator-bonus/page.tsx` | Public rules page (EN / PT / ZH) |
| `src/components/trading/GanhosDoCriador.tsx` | Creator earnings card: one "Claim" for fees + bonus |
| `src/lib/saque-privado.ts` | Cloak flow: deposit, ZK withdrawal, pending-note storage and recovery |
| `src/components/trading/SaquePrivado.tsx` | The "Withdraw privately" panel (EN / PT / ZH) |
| `src/app/api/rpc/solana/route.ts` | RPC proxy that absorbs 429 bursts from the proof flow |
| `src/lib/security-headers.mjs` | CSP: allows the Cloak relay, circuits and WebAssembly (no `unsafe-eval`) |

Stack: Next.js 16, `@meteora-ag/dynamic-bonding-curve-sdk`, `@meteora-ag/cp-amm-sdk`,
`@cloak.dev/sdk`, `@solana/web3.js`. The full app repository is private; access on request.
