# 合约与地址

> [!NOTE]
> **地址的唯一权威来源**是仓库中的 `ops/ADDRESSES.md`。本页是测试网 **t7** 的**快照**，
> 发布前请与 `ops/ADDRESSES.md` 核对。测试网地址会随重部署变化。

---

## 网络

| 参数 | 值 |
|---|---|
| 类型 | Arbitrum Orbit L2 |
| **主网 chainId** | **4663** |
| **测试网 chainId** | **46630** |
| 原生 Gas | **ETH** |
| RPC（主网） | `https://rpc.mainnet.chain.robinhood.com` |
| 出块时间 | 约 100–250 ms `[assumed — 来自实测笔记]` |

---

## 合约清单

核心合约（Foundry 工程）： `[proven]`

| 合约 | 角色 |
|---|---|
| `Config` | 参数中心（价格、难度、费用、分配、抽水、积分等旋钮） |
| `ProofOfCard` | 主 NFT：ERC-721（精简）+ ERC-2981 版税；PoW 挖矿与种子推导 |
| `ChipToken` | 筹码：ERC-1155（−30% 折扣券） |
| `StakeVault` | 质押金库（6 档权重、分红、质押卡参战） |
| `Battle` | 决斗托管与结算（createDuel / acceptDuel / cancelDuel） |
| `Points` | 链上活跃积分与排行榜数据源 |
| `Packs` | 卡包（N 张折扣铸造、保底计数） |

---

## 测试网部署（Robinhood Chain 46630）

部署于 **2026-09-29**（**redeploy t7**：Battle v2 = 元素 + 技能 + 结果浮动；**上级推荐 3% +
邀请人 7%**；**PvP 抽水 70% 国库 / 30% 分红池**；**难度 +0.33 位/纪元**；`eraSize = 1111`；
`masterRef` = deployer（测试期）），由 deployer 部署。**`treasury` = Safe**；`Config` 所有者 =
deployer（测试期，移交 Safe 为**单独步骤**）。 `[proven]`

| 合约 / 角色 | 地址 |
|---|---|
| **Safe 2-of-3** | `0x7AC43F96021C50dC057F2F5f6fc5bAa02D4aD912` |
| Config | `0x4ac4f6850D7b25688636d1A2a903721cDE8Ef72d` |
| ChipToken（ERC-1155） | `0x7b8cbA0FA17b8d5982B6e33EB4d31fb5115527c7` |
| ProofOfCard | `0xeD628d2c353cB2071DD243E067C94516F38572d9` |
| StakeVault | `0xf41AcCFcC3A1A3225133F9C261b76Ca4a7651B54` |
| Battle | `0xAe8bDD77F59d3684A25Eb21ab2dAe980aa1710Bb` |
| Points | `0x6327Ed72071B223f786Cc5BF053516F667F97119` |
| Packs | `0x521eE1759F1842Cd81962c068Ec5e9eB708FDf2A` |

---

## 已核对的关联 [proven]

| 关联 | 值 |
|---|---|
| `ProofOfCard.vault()` | = StakeVault |
| `Battle.vault()` | = StakeVault |
| `StakeVault.battle()` | = Battle |
| `ChipToken.owner()` | = Safe |
| `ChipToken.minter()` | = ProofOfCard |
| `Points.owner()` | = `Config.owner()` |
| `Config.treasury()` | = Safe |
| `Packs.poc()` | = ProofOfCard |
| `ProofOfCard.packMinter()` | = Packs |
| `ProofOfCard.masterRef()` | = deployer（测试期） |

---

## 测试网参数（t7）[proven]

| 键 | 值 |
|---|---|
| `eraSize` | 1111 |
| `baseBits` | 20 |
| `bitsStepX100` | 33 |
| `eraPrice` | 0.0003 ETH |
| `priceStepBps` | 2500 |
| `maxSupply` | 8888 |
| 分配 | 6000 / 1000 / 3000 / 0 |
| `pvpRakeBps` | 1000 |
| `rakeToPoolBps` | 3000 |
| `masterRefBps` | 300 |
| `typeAdvBps` | 2000 |
| `atkVarianceBps` | 4000 |
| `lives` | 3 |

> [!NOTE]
> 更早的测试网部署（t6 / t5 / t4 / t3 / t2）地址存放在仓库的 git 历史中。本页只列出最新 t7。

---

## 测试网钱包 [proven]

| 角色 | 地址 |
|---|---|
| testnet-deployer | `0x280d1C2B20728F43d440A7449113283aD224207A` |
| testnet-signer-1 | `0xfe8FEDA23C5cB174f4F2D7DA09053d14eE963032` |
| testnet-signer-2 | `0x82a7b043eaaDB46df247869987f1A0E1D6dC752C` |
| testnet-signer-3 | `0xbc99Ead2eC29ef7bbb0098c63278fF06c6242515` |

> [!WARNING]
> **官方地址 ≠ 团队 / 个人地址。** 永远不要把两者混淆。主网钱包与 Safe 将由同一脚本
> （`--chain mainnet`）单独生成，地址**不同**。

---

## 相关工具

| 工具 | 用途 |
|---|---|
| **OpenSea** | Robinhood Chain 原生市场——二级交易、5% 版税 `[proven]` |
| **RobinScan / Blockscout** | 区块浏览器（地址、交易、事件） `[proven]` |
| **推荐基金钱包** | 项目独立钱包 `[proven]` |

**不使用 Uniswap / LP**——没有 ERC-20 代币，交易只在二级市场进行。 `[proven]`

---

*下一步：[安全与所有权](security-and-ownership.md) · [官方链接](official-links.md) · [给 AI 智能体](for-ai-agents.md)*
