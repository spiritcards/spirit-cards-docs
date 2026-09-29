# Contracts & addresses

The canonical source of truth for addresses is the project's `ops/ADDRESSES.md`. This page mirrors the
testnet (**t7**) deployment and explains each contract's role.

> [!WARNING]
> Always verify an address against the official source before sending funds. Addresses below are the
> **Robinhood Chain testnet (chainId `46630`)** rehearsal deployment. Mainnet addresses are **not yet
> published**.

---

## The seven contracts

| Contract | Role | Standard |
|---|---|---|
| **`Config`** | Central tunable parameters + ownership + treasury + pause | — |
| **`ProofOfCard`** (`POC`) | Core NFT: PoW mining, merge, price, revenue split, royalty | ERC-721 + ERC-2981 |
| **`ChipToken`** | The chip consumable (mint discount) | ERC-1155 |
| **`StakeVault`** | Staking tiers, reward pool, staked cards, battle locks | — |
| **`Battle`** | PvP duels: escrow, resolution, lives, rake | — |
| **`Points`** | Activity points + leaderboard | — |
| **`Packs`** | Pack purchases (5/10/25/50/100) | — |

`[proven]`

---

## Roles and wiring

| Relationship | Value / check |
|---|---|
| `ProofOfCard.config` | → `Config` |
| `ProofOfCard.chip` | → `ChipToken` |
| `ProofOfCard.vault` | → `StakeVault` (destination of `pumpPool`) |
| `ProofOfCard.packMinter` | → `Packs` |
| `ProofOfCard.masterRef` | deployer (testnet rehearsal) |
| `StakeVault.nft` | → `ProofOfCard` |
| `StakeVault.battle` | → `Battle` (authorized to lock staked cards) |
| `Battle.vault` | → `StakeVault` |
| `Battle.poc` | → `ProofOfCard` |
| `ChipToken.minter` | → `ProofOfCard` |
| `ChipToken.owner` | → Safe |
| `Points.owner()` | → `Config.owner()` (dynamic) |

`[proven]` — these links are verified on the testnet deployment.

---

## Testnet deployment — Robinhood Chain (chainId 46630)

Deployed **2026‑09‑29** (redeploy **t7** — Battle v2: elements + skills + variance; master‑ref 3% +
referrer 7%; PvP rake 70% treasury / 30% pool; difficulty +0.33 bit/era; `eraSize = 1111`;
`masterRef` = deployer). `[proven]`

| Contract / role | Address |
|---|---|
| **Safe 2-of-3** | `0x7AC43F96021C50dC057F2F5f6fc5bAa02D4aD912` |
| **Config** | `0x4ac4f6850D7b25688636d1A2a903721cDE8Ef72d` |
| **ChipToken** (ERC-1155) | `0x7b8cbA0FA17b8d5982B6e33EB4d31fb5115527c7` |
| **ProofOfCard** | `0xeD628d2c353cB2071DD243E067C94516F38572d9` |
| **StakeVault** | `0xf41AcCFcC3A1A3225133F9C261b76Ca4a7651B54` |
| **Battle** | `0xAe8bDD77F59d3684A25Eb21ab2dAe980aa1710Bb` |
| **Points** | `0x6327Ed72071B223f786Cc5BF053516F667F97119` |
| **Packs** | `0x521eE1759F1842Cd81962c068Ec5e9eB708FDf2A` |

**Verified on-chain `[proven]`:** `ProofOfCard.vault()` = StakeVault · `Battle.vault()` = StakeVault ·
`StakeVault.battle()` = Battle · `ChipToken.owner()` = Safe · `ChipToken.minter()` = ProofOfCard ·
`Points.owner()` = `Config.owner()` · `Config.treasury()` = Safe · `Packs.poc()` = ProofOfCard ·
`POC.packMinter()` = Packs · `POC.masterRef()` = deployer.

### t7 parameters

| Parameter | Value |
|---|---|
| `eraSize` | 1111 |
| `baseBits` | 20 |
| `bitsStepX100` | 33 |
| `eraPrice` | 0.0003 ETH |
| `priceStepBps` | 2500 |
| `maxSupply` | 8888 |
| Split (`pool/referral/house/reserve`) | 6000 / 1000 / 3000 / 0 |
| `pvpRakeBps` | 1000 |
| `rakeToPoolBps` | 3000 |
| `masterRefBps` | 300 |
| `typeAdvBps` | 2000 |
| `atkVarianceBps` | 4000 |
| `lives` | 3 |

`[proven]`

---

## Chain parameters

| Property | Value |
|---|---|
| Network | Robinhood Chain (Arbitrum Orbit L2) |
| Mainnet chainId | `4663` |
| Testnet chainId | `46630` |
| Gas token | ETH |
| Block time | ~100–250 ms |
| RPC (mainnet) | `https://rpc.mainnet.chain.robinhood.com` |

`[proven]`

---

## Project wallets

Separate from team/personal wallets. Private keys live in **KeePass**, never in the repo. `[proven]`

| Name | Purpose |
|---|---|
| `deployer` | Contract deployment (ownership → Safe later) |
| `treasury` | Team + marketing (from the split) |
| `pool` | Staker dividends (later → the vault) |
| `referral` | Referral fund |
| `house-arena` | House arena (bots, liquidity) |
| `ops-hot` | Operational gas |
| `market` | Marketplace / secondary revenue (if needed) |

Mainnet wallets and testnet wallets are **distinct sets**. `[proven]`

---

## Reading it on-chain

```solidity
Config.owner()            Config.treasury()        Config.paused()
ProofOfCard.name()        ProofOfCard.symbol()     ProofOfCard.totalMinted()
ProofOfCard.currentPrice() ProofOfCard.requiredBits()
Battle.duelCount()        StakeVault.totalWeight() Packs.priceFor(i)
```

`[proven]`

---

## Next

- **[Security & ownership](security-and-ownership.md)**
- **[Official links](official-links.md)**
