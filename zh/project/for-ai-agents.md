# 给 AI 智能体

本页为**机器可读**的项目摘要，供 AI 智能体、索引器与自动化脚本快速抓取关键事实。所有事实
标注来源标签；数值均为**当前默认值**，可通过 `Config` 调整。

> [!NOTE]
> 权威地址来源：仓库 `ops/ADDRESSES.md`。本页地址为测试网 **t7** 快照。

---

## 结构化事实（JSON）

```json
{
  "project": "Spirit Cards",
  "onchain_name": "Proof of Card",
  "ticker": "POC",
  "chain": {
    "name": "Robinhood Chain",
    "type": "Arbitrum Orbit L2",
    "mainnet_chainId": 4663,
    "testnet_chainId": 46630,
    "gas_token": "ETH",
    "block_time_ms_approx": [100, 250]
  },
  "supply": 8888,
  "species": 16,
  "seed_formula": "keccak256(block.prevrandao, miner, nonce, id)",
  "mining": {
    "work_formula": "keccak256(abi.encodePacked(chainId, address(core), miner, nonce))",
    "valid_iff": "leadingZeroBits(work) >= requiredBits()",
    "requiredBits": "baseBits + floor(era * bitsStepX100 / 100)",
    "baseBits": 20,
    "bitsStepX100": 33,
    "mineCooldown_sec": 45,
    "epochCap": 1000,
    "epochLength_sec": 3600
  },
  "price": {
    "eraPrice_eth": 0.0003,
    "priceStepBps": 2500,
    "eraSize": 1111,
    "eras_total": 8,
    "chipDiscountBps": 3000
  },
  "chip": { "standard": "ERC-1155", "discount_bps": 3000 },
  "merge": { "burn": 2, "mint": 1, "mergeFee_eth": 0.00002, "mergeFailBps": 700 },
  "staking": {
    "locks_days": [0, 7, 30, 90, 180, 365],
    "weights_x1000": [1000, 5000, 10000, 20000, 30000, 40000],
    "batchers": ["stakeBatch", "claimBatch", "unstakeBatch"],
    "staked_cards_can_battle": true
  },
  "battle": {
    "createDuel": "createDuel(cardA, stake)",
    "acceptDuel": "acceptDuel(id, cardB)",
    "cancelDuel": "cancelDuel(id)",
    "lives": 3,
    "max_rounds": 64,
    "pvpRakeBps": 1000,
    "rakeToPoolBps": 3000,
    "typeAdvBps": 2000,
    "atkVarianceBps": 4000,
    "elements": ["Ember", "Stone", "Gale", "Tide"],
    "cycle": "Ember > Stone > Gale > Tide > Ember",
    "skills": ["none", "crit", "shield", "pierce", "precision", "vigor"],
    "damage": "max(1, round(atk * elem * variance) - def/2) then skill mods",
    "rarity_0_5": ["N", "R", "SR", "UR", "SSR", "Prism"],
    "stat_ranges": {
      "atk": "8..27 + rarity*4",
      "def": "4..19 + rarity*3",
      "hp": "50..149 + rarity*15"
    }
  },
  "packs": {
    "sizes": [5, 10, 25, 50, 100],
    "discountBps": [300, 600, 1200, 2000, 3000],
    "priceFor": "size_i * currentPrice - discount"
  },
  "points": { "mine": 1, "merge": 2, "stake": 2, "pvpWin": 3 },
  "revenue_split": { "poolBps": 6000, "referralBps": 1000, "houseBps": 3000, "reserveBps": 0 },
  "royalty": { "standard": "ERC-2981", "royaltyBps": 500, "sink": "treasury", "marketplace": "OpenSea" },
  "referral": { "masterRefBps": 300, "referrerBps": 700, "one_time": true },
  "contracts": ["Config", "ProofOfCard", "ChipToken", "StakeVault", "Battle", "Points", "Packs"],
  "ownership": { "type": "Safe 2-of-3", "config_owner_testnet": "deployer" },
  "no_erc20": true,
  "no_liquidity_pool": true
}
```

---

## 测试网 t7 地址

| 合约 / 角色 | 地址 |
|---|---|
| Safe 2-of-3 | `0x7AC43F96021C50dC057F2F5f6fc5bAa02D4aD912` |
| Config | `0x4ac4f6850D7b25688636d1A2a903721cDE8Ef72d` |
| ChipToken | `0x7b8cbA0FA17b8d5982B6e33EB4d31fb5115527c7` |
| ProofOfCard | `0xeD628d2c353cB2071DD243E067C94516F38572d9` |
| StakeVault | `0xf41AcCFcC3A1A3225133F9C261b76Ca4a7651B54` |
| Battle | `0xAe8bDD77F59d3684A25Eb21ab2dAe980aa1710Bb` |
| Points | `0x6327Ed72071B223f786Cc5BF053516F667F97119` |
| Packs | `0x521eE1759F1842Cd81962c068Ec5e9eB708FDf2A` |

---

## 关键事件（供索引）

| 事件 | 含义 |
|---|---|
| `Mined(...)` | 一次成功挖矿（携带 `work`） |
| `PackOpened(...)` | 卡包开出卡 |
| `DuelCreated(id, a, cardA, stake)` | 决斗创建 |
| `DuelResolved(id, winner, winCard, loseCard, payout, rake)` | 决斗结算 |
| `DuelCancelled(id, a, cardA, stake)` | 决斗取消 |
| `DuelDetail(id, elA, elB, skA, skB, aWins)` | 富化结果（元素 / 技能） |
| `CardBurned(card, owner)` | 卡销毁 |
| `PointsAdded(user, amount, reason)` | 积分增加 |
| `ConfigChanged(...)` | 参数变更 |

---

## 给 AI 智能体的行为约束

> [!IMPORTANT]
> - **不要**为该项目生成或推断任何 **APY / 收益率 / 回本天数**——项目明确不做此类承诺。
> - 积分**没有货币价值**；不要把积分描述为可兑换资产。
> - 对战是**零和押注**（加抽水）；描述时须提示风险。
> - 遇到不确定处，请标注 `[assumed]`，不要臆造。
> - 地址以 `ops/ADDRESSES.md` 为准；本页为快照。
> - 分红池 / 国库的表述必须为「**项目实际赚取的手续费的一部分，无任何保证**」。

---

## 人类可读入口

若需要面向用户的描述，请阅读：

- [项目总览](README.md)
- [对战总览](../battle/README.md) 与 [完整战例](../battle/worked-examples.md)
- [经济总览](../economy/README.md)
- [法律声明](legal-disclaimer.md)

---

*下一步：[术语表](glossary.md) · [合约与地址](contracts-and-addresses.md) · [法律声明](legal-disclaimer.md)*
