# Price & eras

The mint price is not fixed. It starts low and **rises by 25% every price-era**, where an era is a
block of **1111 cards**. There are **8 eras**, covering the whole 8888 base supply.

---

## The formula

```solidity
// ProofOfCard.sol
function currentPrice() public view returns (uint256) {
    uint256 era = paidMinted / config.eraSize();
    uint256 p = config.eraPrice();
    for (uint256 i = 0; i < era; i++) {
        p = (p * (10000 + config.priceStepBps())) / 10000;  // ×1.25 each era
    }
    return p;
}
```

| Parameter | Value | Meaning |
|---|---|---|
| `eraPrice` | **0.0003 ETH** | Starting price (≈ $1 at ETH ≈ $3000) |
| `priceStepBps` | **2500** | +25% per era |
| `eraSize` | **1111** | Cards per era |

`[proven]`

Price in era `e` = `eraPrice × 1.25^e`.

---

## The price ladder

The era is `floor(paidMinted / 1111)`. Below, USD is shown at an illustrative **ETH ≈ $3000**.

| Era | Card range (paid index) | Price (ETH) | Price (≈ USD) | Δ vs. previous |
|---|---|---|---|---|
| **0** | 1 – 1111 | `0.00030000` | ~$0.90 | — |
| **1** | 1112 – 2222 | `0.00037500` | ~$1.13 | +25% |
| **2** | 2223 – 3333 | `0.00046875` | ~$1.41 | +25% |
| **3** | 3334 – 4444 | `0.00058594` | ~$1.76 | +25% |
| **4** | 4445 – 5555 | `0.00073242` | ~$2.20 | +25% |
| **5** | 5556 – 6666 | `0.00091553` | ~$2.75 | +25% |
| **6** | 6667 – 7777 | `0.00114441` | ~$3.43 | +25% |
| **7** | 7778 – 8888 | `0.00143051` | ~$4.29 | +25% |

Price is **monotonic, non-decreasing** and independent of demand — a fixed schedule, not an auction.
`[proven]` for the ETH numbers; `[assumed]` for the USD column (it moves with the ETH price).

> [!NOTE]
> The contract caps the era loop at 64 iterations for safety, so the price formula stays bounded even
> if `eraSize` were retuned. With the defaults, all 8 eras behave as in the table. `[proven]`

---

## Total cost of a full mint

Summing `1111 × price_e` across eras 0–7:

| Era | Cards | Era total (ETH) |
|---|---|---|
| 0 | 1111 | 0.33330 |
| 1 | 1111 | 0.41663 |
| 2 | 1111 | 0.52078 |
| 3 | 1111 | 0.65098 |
| 4 | 1111 | 0.81372 |
| 5 | 1111 | 1.01715 |
| 6 | 1111 | 1.27144 |
| 7 | 1111 | 1.58929 |
| **Total** | **8888** | **≈ 6.61 ETH** |

`[proven]` for the arithmetic (it follows from the formula); the total assumes all cards are mined
at list price, ignoring chip discounts and packs.

In USD: **≈ $19,800** at ETH ≈ $3000. `[assumed]` — purely ETH-price dependent.

---

## Why +25% per era and not ×2

The step is intentionally **soft**. A steep curve rewards hoarding and punishes latecomers; a gentle
curve keeps mining worthwhile across the whole sale while still giving early miners a price edge.

- Early cards are cheaper **by design** — early miners take more risk on an unproven game.
- The price rises regardless of demand, so there is no "get in before the pump" dynamic.
- `[assumed]` — rationale wording; the mechanism (`+25%/era`) is `[proven]`.

---

## Difficulty moves with price

The same era index drives **both** the price and the mining **difficulty**. As price steps up, so does
`requiredBits`, by `+0.33` bit per era:

```
era = paidMinted / eraSize
price(era) = 0.0003 × 1.25^era
requiredBits(era) = 20 + floor(era × 0.33)
```

| Era | Price (ETH) | Difficulty (bits) |
|---|---|---|
| 0–3 | 0.00030 → 0.00059 | 20 |
| 4–6 | 0.00073 → 0.00114 | 21 |
| 7 | 0.00143 | 22 |

`[proven]`

So a card gets **both** more expensive **and** marginally harder to mine as the sale progresses.

---

## Chips reduce the price

A chip applies **−30%** to a mint (`chipDiscountBps = 3000`). A chip is granted only on a **full-price**
mint and is consumed when used. So the effective price of a mint can be as low as `price × 0.70` if you
spend a chip. See [How to mint](../mining/how-to-mint.md).

---

## Reading it on-chain

```solidity
poc.currentPrice()   // price for the current era
poc.paidMinted()     // cards minted (price-indexed)
config.eraPrice()    // start price
config.priceStepBps()// step in bps
config.eraSize()     // cards per era
```

`[proven]`

---

## Next

- **[Packs](packs.md)** — bundled buys at a discount
- **[Staking & rewards](staking-and-rewards.md)**
- **[Difficulty & network pace](../mining/difficulty-and-network-pace.md)**
