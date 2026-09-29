# For AI agents

A **machine-readable** summary of Spirit Cards mechanics, intended for agents, indexers, and
bot developers. Values are the **testnet (t7) defaults** unless noted. All formulas are as implemented
in the contracts.

> [!IMPORTANT]
> This is a **game**. No yield is promised. Points have **no monetary value**. Do not present any of
> these numbers as an investment, an APY, or a guaranteed return.

---

## Network

```json
{
  "chain": "Robinhood Chain (Arbitrum Orbit L2)",
  "mainnet_chainId": 4663,
  "testnet_chainId": 46630,
  "gas_token": "ETH",
  "block_time_ms": [100, 250],
  "rpc_mainnet": "https://rpc.mainnet.chain.robinhood.com"
}
```

`[proven]`

---

## Contracts (testnet t7)

```json
{
  "safe_2of3":   "0x7AC43F96021C50dC057F2F5f6fc5bAa02D4aD912",
  "Config":      "0x4ac4f6850D7b25688636d1A2a903721cDE8Ef72d",
  "ChipToken":   "0x7b8cbA0FA17b8d5982B6e33EB4d31fb5115527c7",
  "ProofOfCard": "0xeD628d2c353cB2071DD243E067C94516F38572d9",
  "StakeVault":  "0xf41AcCFcC3A1A3225133F9C261b76Ca4a7651B54",
  "Battle":      "0xAe8bDD77F59d3684A25Eb21ab2dAe980aa1710Bb",
  "Points":      "0x6327Ed72071B223f786Cc5BF053516F667F97119",
  "Packs":       "0x521eE1759F1842Cd81962c068Ec5e9eB708FDf2A"
}
```

`[proven]`

---

## Core constants (Config defaults)

```json
{
  "baseBits": 20,
  "bitsStepX100": 33,
  "mineCooldown": 45,
  "epochCap": 1000,
  "epochLength": 3600,
  "eraPrice_wei": "300000000000000",
  "priceStepBps": 2500,
  "eraSize": 1111,
  "chipDiscountBps": 3000,
  "mergeFee_wei": "20000000000000",
  "mergeFailBps": 700,
  "splitBps": { "pool": 6000, "referral": 1000, "house": 3000, "reserve": 0 },
  "maxSupply": 8888,
  "lives": 3,
  "pvpRakeBps": 1000,
  "rakeToPoolBps": 3000,
  "typeAdvBps": 2000,
  "atkVarianceBps": 4000,
  "masterRefBps": 300,
  "royaltyBps": 500,
  "tierLock_seconds": [0, 604800, 2592000, 7776000, 15552000, 31536000],
  "tierWeightX1000": [1000, 5000, 10000, 20000, 30000, 40000],
  "points": { "mine": 1, "merge": 2, "stake": 2, "pvpWin": 3 }
}
```

`[proven]`

---

## Formulas

### Mining

```
work = keccak256(abi.encodePacked(chainId, ProofOfCard, miner, nonce))
valid ⇔ leadingZeroBits(work) >= requiredBits()
requiredBits = baseBits + floor(era * bitsStepX100 / 100); era = paidMinted / eraSize
seed = keccak256(abi.encodePacked(block.prevrandao, miner, nonce, id))
nonceUsed[miner][nonce] = true   // one-time
```

`[proven]`

### Pricing

```
era = paidMinted / eraSize
price(era) = eraPrice * (1 + priceStepBps/10000)^era   // eraPrice * 1.25^era
```

Ladder (ETH): `0.0003, 0.000375, 0.00046875, 0.0005859375, 0.000732421875, 0.00091552734375, 0.0011444091796875, 0.001430511474609375`. Total ≈ **6.61 ETH**. `[proven]`

### Card profile (from seed)

```
rarity  = (s >> 248) % 6                                  // 0..5
atk     = 8  + ((s >> 240) & 0xFF) % 20 + rarity*4        // 8..27  + rarity*4
def     = 4  + ((s >> 232) & 0xFF) % 16 + rarity*3        // 4..19  + rarity*3
hp      = 50 + ((s >> 224) & 0xFF) % 100 + rarity*15      // 50..149 + rarity*15
element = (s >> 216) & 0x03                               // 0..3
skill   = (s >> 208) % 6                                  // 0..5
```

`[proven]`

### Merge

```
s = keccak256(seedOf[a], seedOf[b], block.prevrandao, msg.sender)
burn a, b; burned += 2
roll = keccak256(s, "DUD") % 10000
child = (roll < mergeFailBps) ? DUD(id 0) : mint(id = 10_000_000 + ++forgeCounter)
forged child seed = s
```

`[proven]`

### Battle resolution

```
rng0 = keccak256(seedOf[a], seedOf[b], duelId, block.prevrandao, Battle)
atk_i = roll(atk_i)                       // factor ∈ [0.60, 1.40]
mltA, mltB = elementMult(elA, elB)        // {+/-20% | neutral}
if Vigor: hp *= 1.20
for round in 0..63:
    rng = keccak256(rng)
    ai = round % 2                         // 0 => A strikes (even rounds)
    d  = strike(atk[ai], def[1-ai], mlt[ai], sk[ai], sk[1-ai], rng)
    hp[1-ai] -= d; if 0 => winner = (ai == 0)
tiebreak: compare hp[0]*startHp[1] vs hp[1]*startHp[0]; final tie -> (rng&1)==0
```

`strike`:
```
effAtk = atk * elementMult / 10000
def'   = (attacker Pierce) ? 0 : def
base   = max(1, effAtk - def'/2)
if Precision: base = base*115/100
if Crit and rng%5==0: base *= 2
if defender Shield: base = base*70/100
damage = max(1, base)
```

`[proven]`

### Elements

```
beats(x,y) ⇔ y == (x+1) % 4
Ember(0) > Stone(1) > Gale(2) > Tide(3) > Ember(0)
```

`[proven]`

### Settlement

```
pot  = 2 * stake
rake = pot * pvpRakeBps / 10000            // 10%
toPool  = rake * rakeToPoolBps / 10000     // 30%
toHouse = rake - toPool                    // 70% -> treasury
winner payout = pot - rake
loser: lives -= 1; if 0 -> burn
wins[winCard] += 1; losses[loseCard] += 1
points.addPoints(winner, pointsPvpWin, "PVP_WIN")
```

`[proven]`

### Revenue split (mint/merge/pack)

```
toPool     = amount * poolBps / 10000      // 60%
toReferral = amount * referralBps / 10000  // 10%
toHouse    = amount * houseBps / 10000     // 30%
toReserve  = amount - others               // 0%
inside referral fund:
  toMaster = min(amount * masterRefBps / 10000, toReferral)   // 3%
  toReferrer = toReferral - toMaster                          // 7%
```

`[proven]`

### Packs

```
size        = [5, 10, 25, 50, 100]
discountBps = [300, 600, 1200, 2000, 3000]
priceFor(i) = currentPrice()*size[i] - base*discountBps[i]/10000
buyPack(i) -> poc.mintPack(buyer, size[i]); revenue via collectRevenue -> split
pack seeds: keccak256(block.prevrandao, buyer, id, index)
```

`[proven]`

---

## Events

```
Mined(miner, tokenId, nonce, work, bits, paid)
Merged(a, b, child, fee)                 // child == 0 -> dud
DuelCreated(id, a, cardA, stake)
DuelDetail(id, elA, elB, skA, skB, aWins)
DuelResolved(id, winner, winCard, loseCard, payout, rake)
DuelCancelled(id, a, cardA, stake)
CardBurned(card, owner)
Staked(user, tokenId, tier, weight)
RewardsNotified(amount)
PoolPumped(vault, amount)
PointsAdded(user, amount, reason)
ConfigChanged(key, oldValue, newValue)
PackOpened(buyer, size, firstId, paid, discountBps)
```

`[proven]`

---

## Do / Don't for agents

**Do:**

- Read `seedOf`, `profileOf`, `currentPrice`, `requiredBits` before acting.
- Reproduce `work` exactly as `keccak256(chainId ‖ contract ‖ miner ‖ nonce)`.
- Treat battle as **probabilistic** (elements + skills + variance).
- Respect `maxSupply`, cooldown, epoch cap, and the pause switch.

**Don't:**

- Present any figure as an APY, yield, or guaranteed return.
- Claim points have value.
- Assume a fixed battle outcome from stats alone.
- Share or request seed phrases / private keys.

---

## Full parameter disclosure

Every tunable is a public getter on `Config`. To enumerate:

```solidity
Config.poolBps() Config.referralBps() Config.houseBps() Config.reserveBps()
Config.masterRefBps() Config.royaltyBps() Config.maxSupply() Config.lives()
Config.pvpRakeBps() Config.rakeToPoolBps() Config.typeAdvBps() Config.atkVarianceBps()
Config.baseBits() Config.bitsStepX100() Config.mineCooldown() Config.epochCap() Config.epochLength()
Config.eraPrice() Config.priceStepBps() Config.eraSize() Config.chipDiscountBps()
Config.mergeFee() Config.mergeFailBps() Config.treasury()
```

`[proven]`

---

## Next

- **[Contracts & addresses](contracts-and-addresses.md)**
- **[Battle → Resolution & damage](../battle/resolution-and-damage.md)**
