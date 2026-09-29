# Security & ownership

This page describes how Spirit Cards is governed, the ownership model, the safeguards in the contracts,
and the current audit status.

> [!IMPORTANT]
> Security is a **process**, not a badge. The contracts are deployed and function; a **formal external
> audit** and the **mainnet ownership handover to a Safe multisig** are planned, separate steps.

---

## Ownership model

| Contract | Owner / admin |
|---|---|
| `Config` | `owner` (deployer on testnet; **target: Safe 2-of-3**) |
| `ChipToken` | **Safe** (testnet Safe exists) `[proven]` |
| `Points` | `Config.owner()` — **dynamic**, follows `Config` `[proven]` |
| `ProofOfCard`, `StakeVault`, `Battle`, `Packs` | gate admin calls on `config.owner()` `[proven]` |

The design principle: **one ownership knob** (`Config.owner`) governs the whole system. Most modules ask
`config.owner()` for authorization rather than storing their own owner. So a single handover of `Config`
transfers control of the set.

`Config` uses a **two-step ownership transfer**:

```solidity
function nominateOwner(address to) external onlyOwner;   // owner nominates
function acceptOwnership() external;                     // nominee accepts
```

`[proven]`

> [!NOTE]
> **Current testnet state:** `Config.owner` = deployer (a rehearsal simplification). `ChipToken.owner` =
> Safe. Handing `Config` to the Safe is a **planned** step (the Safe contract already exists on
> testnet). `[proven]`

---

## The Safe (2-of-3)

Testnet Safe: `0x7AC43F96021C50dC057F2F5f6fc5bAa02D4aD912`. `[proven]`

- **2-of-3 multisig:** any admin action needs two of three signers.
- `Config.treasury()` = Safe (treasury funds land in the multisig).
- `ChipToken.owner` = Safe.
- **Target:** `Config` ownership → Safe, so parameter changes require multisig approval.

The mainnet Safe is created later with the same procedure (`--chain mainnet`). `[planned]`

---

## Access control in the contracts

| Guard | Where | Effect |
|---|---|---|
| `onlyOwner` / `onlyConfigOwner` | `Config`, `ProofOfCard`, `Packs` | Parameter & fund administration |
| `msg.sender == config.owner()` | `Battle.setPointsContract`, `setVault`, `setMasterRef`, `StakeVault.setBattle` | Wiring modules |
| `onlyMinter` | `ChipToken` | Only `ProofOfCard` mints/burns chips |
| `onlyBattle` | `StakeVault` | Only `Battle` locks/unlocks/kills staked cards |
| `authorized[module]` | `Points` | Only wired modules award points |

`[proven]`

---

## Safeguards & invariants

The contracts include several deliberate protections: `[proven]`

- **Reentrancy guards** (`nonReentrant`) on all state-changing entry points that move ETH or NFTs.
- **Checks-effects-interactions:** external ETH sends use `.call{value:...}("")` with a success check,
  after state updates.
- **Chip asymmetry:** a full-price mint grants a chip; a discounted mint only consumes one — so the
  −30% can never renew itself in a loop.
- **Merge "dud" (anti-arbitrage):** the 7% `mergeFailBps` chance closes the "N+1 > 2×N" loop.
- **Reward-debt rounding up (`_mulDivCeil`):** a staker can never claim more than their exact share —
  closed a 1-wei solvency edge found by invariant fuzzing.
- **Referral solvency protection:** `referralOutstanding` shields owed referral balances from the
  owner's sweep (`withdrawReferralLeftover` only takes the surplus).
- **`pumpPool()` is permissionless and safe:** it can only send pool funds to the vault, never to the
  caller.
- **Nonce is one-time per (miner, nonce):** prevents replaying a found nonce.
- **`maxSupply` gate** on both `mine` and `mintPack`.

---

## Audit status

- **Internal audit `[proven]`:** found **no critical or high** "steal funds" or "free-mint" issues.
- **Test suite `[proven]`:** a Foundry suite (hundreds of tests, including invariant/fuzz tests for the
  vault solvency) passes.
- **Formal external audit `[planned]`:** a separate step before mainnet.

> [!NOTE]
> An internal audit is **not** a guarantee. Smart contracts can still contain bugs. Only deposit what
> you can afford to lose, and prefer testnet for experimentation.

---

## Pause switch

`Config.paused` gates the main entry points:

| Function | Pausable? |
|---|---|
| `mine`, `mergeBurn` | Yes |
| `createDuel`, `acceptDuel` | Yes |
| `StakeVault.stake`, `stakeBatch` | Yes |
| `Packs.buyPack` | Yes |
| `cancelDuel`, `unlockFromBattle`, `killInBattle` | No (safety exits stay open) |

`[proven]`

Pausing **stops new actions** but leaves **exit paths open** so users can cancel/reclaim.

---

## Tunability (and its trade-off)

Virtually every parameter is adjustable by the owner via `Config` (`setMining`, `setPricing`,
`setSplit`, `setBattleParams`, `setRakeSplit`, `setStakingTier`, `setPoints`, `setRoyalty`, etc.).
Each change emits `ConfigChanged(key, old, new)`. `[proven]`

- **Upside:** the game can be rebalanced in the field without a migration.
- **Trade-off:** an owner can change parameters — including splitting the revenue. This is exactly why
  ownership is **targeted at a 2-of-3 Safe** rather than a single key: changes require multisig
  agreement and are visible on-chain. `[assumed]` rationale; mechanism `[proven]`.

---

## Key management

- All project private keys live in **KeePass** (`Crypto/pow_cards/...`), never in the repo.
- Mainnet and testnet keys are **separate sets**. `[proven]`
- No secrets in git; no `set -x` in scripts.

---

## Reporting a vulnerability

If you find a security issue, **do not** exploit it. Report it privately to the team via the channels on
the [Official links](official-links.md) page. For a production protocol, responsible disclosure matters.

---

## Honest summary

| Claim | Status |
|---|---|
| Reentrancy guards present | `[proven]` |
| Internal audit: no critical/high steal/free-mint | `[proven]` |
| Test suite passing | `[proven]` |
| Chip/referral/merge anti-abuse logic | `[proven]` |
| Ownership → Safe 2-of-3 | `[planned]` (testnet Safe exists) |
| External audit | `[planned]` |
| Mainnet | `[planned]` |

---

## Next

- **[Contracts & addresses](contracts-and-addresses.md)**
- **[Legal disclaimer](legal-disclaimer.md)**
