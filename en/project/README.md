# Project

This section covers the **engineering and operational** side of Spirit Cards: the contracts and their
addresses, the security and ownership model, official links, answers to common questions, the roadmap,
a page specifically for AI agents, a glossary, and the legal disclaimer.

> [!NOTE]
> Spirit Cards is deployed on **Robinhood Chain** (Arbitrum Orbit L2). Testnet is live; mainnet,
> a formal audit, and the handover of ownership to a Safe multisig are **separate, planned steps**.

---

## Contents

| Page | What it covers |
|---|---|
| **[Contracts & addresses](contracts-and-addresses.md)** | The 7 contracts, their roles, the testnet deployment, chain params |
| **[Security & ownership](security-and-ownership.md)** | Access control, ownership handover, audit status, safeguards |
| **[Official links](official-links.md)** | Website, brand domain, X, Telegram, GitHub |
| **[FAQ](faq.md)** | Common questions, answered plainly |
| **[Roadmap](roadmap.md)** | Phases, what is done, what is planned |
| **[For AI agents](for-ai-agents.md)** | A machine-readable summary of the mechanics |
| **[Glossary](glossary.md)** | Terms in EN / RU / ZH |
| **[Legal disclaimer](legal-disclaimer.md)** | Risks, no-advice, China compliance notes |

---

## Project shape

- **Contracts** (Foundry): `Config`, `ProofOfCard`, `ChipToken` (ERC-1155), `StakeVault`, `Battle`,
  `Points`, `Packs`. `[proven]`
- **Web app** (Next.js 15): Collection · Mine · Merge · Stake · Battle · Points · Docs · Pool · Profile.
  `[proven]`
- **Miner**: an in-browser PoW engine (auto-mining in the background) and a CLI miner for farms.
  `[proven]`
- **Art pipeline**: 16 species (8 universal + 8 dragons), deterministic render from the seed. `[proven]`

---

## Repository layout (high level)

```
contracts/        Foundry project: src/{Config,ProofOfCard,ChipToken,
                  StakeVault,Battle,Points,Packs,ReentrancyGuard}.sol
                  + test/ (suite) + script/{Deploy,PumpPool}.s.sol
webapp/           Next.js 15 site (production on Vercel), RH testnet
miner/            CLI PoW miner + phone hash-rate benchmark
ops/              ADDRESSES.md (source of truth), chain notes
gitbook/          documentation (this book, EN under gitbook/en/)
```

`[proven]` for the structure; exact file listings can change.

---

## Status at a glance

| Area | Status |
|---|---|
| Contracts (Foundry) | Deployed on testnet; test suite green `[proven]` |
| Web app | Production on Vercel (testnet) `[proven]` |
| CLI miner | Working `[proven]` |
| Testnet stack (t7) | Live `[proven]` |
| Mainnet | Planned `[planned]` |
| Formal audit | Planned; internal audit found no critical/high steal/free-mint issues `[proven]` |
| Ownership handover to Safe 2-of-3 | Planned (testnet Safe exists) `[proven]` |

---

## Next

Start with **[Contracts & addresses](contracts-and-addresses.md)**.
