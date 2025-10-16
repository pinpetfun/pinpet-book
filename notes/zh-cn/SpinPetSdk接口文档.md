# SpinPetSdk 接口文档

这是 SpinPetSdk 类的详细接口说明文档，为开发者提供清晰明了的 API 使用指南。

## 目录

1. [SDK 初始化](#sdk-初始化)
2. [核心配置](#核心配置)
3. [Trading 模块 - 交易功能](#trading-模块---交易功能)
4. [Fast 模块 - 数据获取](#fast-模块---数据获取)
5. [Token 模块 - 代币管理](#token-模块---代币管理)
6. [Param 模块 - 参数管理](#param-模块---参数管理)
7. [Simulator 模块 - 交易模拟](#simulator-模块---交易模拟)
8. [Chain 模块 - 链上数据查询](#chain-模块---链上数据查询)
9. [工具方法](#工具方法)

---

## SDK 初始化

### 构造函数

```javascript
new SpinPetSdk(connection, wallet, programId, options)
```

**参数:**
- `connection` *(Connection)*: Solana 连接实例
- `wallet` *(Wallet|Keypair)*: 钱包实例或密钥对
- `programId` *(PublicKey|string)*: 程序 ID
- `options` *(Object)*: 可选配置参数

**options 配置项:**
```javascript
{
  fee_recipient: "4nffmKaNrex34LkJ99RLxMt2BbgXeopUi8kJnom3YWbv",           // 手续费接收账户
  base_fee_recipient: "8fJpd2nteqkTEnXf4tG6d1MnP9p71KMCV4puc9vaq6kv",      // 基础手续费接收账户
  params_account: "DVRnPDW1MvUhRhDfE1kU6aGHoQoufBCmQNbqUH4WFgUd",          // 参数账户
  spin_fast_api_url: "http://192.168.18.36:8080",                         // FastAPI 地址
  commitment: "confirmed",                                                 // 提交级别
  preflightCommitment: "processed",                                        // 预检查提交级别
  skipPreflight: false,                                                    // 是否跳过预检查
  maxRetries: 3                                                           // 最大重试次数
}
```

**示例:**
```javascript
const { Connection, PublicKey } = require('@solana/web3.js');
const anchor = require('@coral-xyz/anchor');
const SpinPetSdk = require('spinpet-sdk');

// 创建连接
const connection = new Connection('http://localhost:8899', 'confirmed');

// 创建钱包
const wallet = new anchor.Wallet(keypair);

// 初始化 SDK
const sdk = new SpinPetSdk(
  connection, 
  wallet, 
  "YourProgramId", 
  {
    fee_recipient: "4nffmKaNrex34LkJ99RLxMt2BbgXeopUi8kJnom3YWbv",
    base_fee_recipient: "8fJpd2nteqkTEnXf4tG6d1MnP9p71KMCV4puc9vaq6kv",
    params_account: "DVRnPDW1MvUhRhDfE1kU6aGHoQoufBCmQNbqUH4WFgUd",
    spin_fast_api_url: "http://localhost:8080"
  }
);
```

---

## 核心配置

### 常量配置

- `sdk.MAX_ORDERS_COUNT`: 20 - 每笔交易最大订单处理数量
- `sdk.FIND_MAX_ORDERS_COUNT`: 1000 - 查询订单时最大获取数量

---

## Trading 模块 - 交易功能

### sdk.trading.buy() - 买入代币

```javascript
await sdk.trading.buy(params, options)
```

**参数:**
- `params.mintAccount` *(string|PublicKey)*: 代币铸造账户地址
- `params.buyTokenAmount` *(anchor.BN)*: 要购买的代币数量
- `params.maxSolAmount` *(anchor.BN)*: 最大 SOL 花费
- `params.payer` *(PublicKey)*: 支付者公钥
- `options.computeUnits` *(number)*: 计算单元限制，默认 1400000

**返回值:**
```javascript
{
  transaction: Transaction,           // 交易对象
  signers: [],                       // 签名者数组（空数组，只需 payer 签名）
  accounts: {                        // 相关账户信息
    mint: PublicKey,
    curveAccount: PublicKey,
    poolTokenAccount: PublicKey,
    poolSolAccount: PublicKey,
    userTokenAccount: PublicKey,
    payer: PublicKey
  },
  orderData: {                       // 订单数据信息
    ordersUsed: number,              // 使用的订单数量
    lpPairsCount: number,            // LP 配对数量
    lpPairs: Array,                  // LP 配对数组
    orderAccounts: Array             // 订单账户数组
  }
}
```

**示例:**
```javascript
const result = await sdk.trading.buy({
  mintAccount: "HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7",
  buyTokenAmount: new anchor.BN("1000000000"),    // 1 token (假设精度为 9)
  maxSolAmount: new anchor.BN("2000000000"),      // 2 SOL
  payer: wallet.publicKey
});

// 签名并发送交易
const signature = await connection.sendTransaction(result.transaction, [wallet.payer]);
```

### sdk.trading.sell() - 卖出代币

```javascript
await sdk.trading.sell(params, options)
```

**参数:**
- `params.mintAccount` *(string|PublicKey)*: 代币铸造账户地址
- `params.sellTokenAmount` *(anchor.BN)*: 要卖出的代币数量
- `params.minSolOutput` *(anchor.BN)*: 最小 SOL 输出
- `params.payer` *(PublicKey)*: 支付者公钥
- `options.computeUnits` *(number)*: 计算单元限制，默认 1400000

**返回值:** 同 `buy()` 方法

**示例:**
```javascript
const result = await sdk.trading.sell({
  mintAccount: "HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7",
  sellTokenAmount: new anchor.BN("1000000000"),   // 1 token
  minSolOutput: new anchor.BN("1800000000"),      // 最少得到 1.8 SOL
  payer: wallet.publicKey
});
```

### sdk.trading.long() - 保证金做多

```javascript
await sdk.trading.long(params, options)
```

**参数:**
- `params.mintAccount` *(string|PublicKey)*: 代币铸造账户地址
- `params.buyTokenAmount` *(anchor.BN)*: 要购买的代币数量
- `params.maxSolAmount` *(anchor.BN)*: 最大 SOL 花费
- `params.marginSol` *(anchor.BN)*: 保证金数量
- `params.closePrice` *(anchor.BN)*: 平仓价格
- `params.prevOrder` *(PublicKey|null)*: 前一个订单
- `params.nextOrder` *(PublicKey|null)*: 下一个订单
- `params.payer` *(PublicKey)*: 支付者公钥
- `options.computeUnits` *(number)*: 计算单元限制，默认 1400000

**返回值:**
```javascript
{
  transaction: Transaction,
  signers: [],
  accounts: {
    mint: PublicKey,
    curveAccount: PublicKey,
    poolTokenAccount: PublicKey,
    poolSolAccount: PublicKey,
    payer: PublicKey,
    selfOrder: PublicKey               // 自己创建的订单地址
  },
  orderData: {
    ordersUsed: number,
    lpPairsCount: number,
    lpPairs: Array,
    orderAccounts: Array,
    uniqueSeed: anchor.BN,             // 唯一种子
    prevOrder: PublicKey|null,
    nextOrder: PublicKey|null
  }
}
```

**示例:**
```javascript
const result = await sdk.trading.long({
  mintAccount: "HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7",
  buyTokenAmount: new anchor.BN("10000000"),      // 0.01 token
  maxSolAmount: new anchor.BN("1100000000"),      // 最多花费 1.1 SOL
  marginSol: new anchor.BN("2200000000"),         // 保证金 2.2 SOL
  closePrice: new anchor.BN("1000000000000000"),  // 平仓价格
  prevOrder: null,
  nextOrder: null,
  payer: wallet.publicKey
});
```

### sdk.trading.short() - 保证金做空

```javascript
await sdk.trading.short(params, options)
```

**参数:**
- `params.mintAccount` *(string|PublicKey)*: 代币铸造账户地址
- `params.borrowSellTokenAmount` *(anchor.BN)*: 借出卖出的代币数量
- `params.minSolOutput` *(anchor.BN)*: 最小 SOL 输出
- `params.marginSol` *(anchor.BN)*: 保证金数量
- `params.closePrice` *(anchor.BN)*: 平仓价格
- `params.prevOrder` *(PublicKey|null)*: 前一个订单
- `params.nextOrder` *(PublicKey|null)*: 下一个订单
- `params.payer` *(PublicKey)*: 支付者公钥
- `options.computeUnits` *(number)*: 计算单元限制，默认 1400000

**返回值:** 类似 `long()` 方法

**示例:**
```javascript
const result = await sdk.trading.short({
  mintAccount: "HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7",
  borrowSellTokenAmount: new anchor.BN("1000000000"),  // 借出卖出 1 token
  minSolOutput: new anchor.BN("100"),                   // 最少得到 0.0000001 SOL
  marginSol: new anchor.BN("2200000000"),               // 保证金 2.2 SOL
  closePrice: new anchor.BN("1000000000000000"),        // 平仓价格
  prevOrder: null,
  nextOrder: null,
  payer: wallet.publicKey
});
```

### sdk.trading.closeLong() - 平仓做多

```javascript
await sdk.trading.closeLong(params, options)
```

**参数:**
- `params.mintAccount` *(string|PublicKey)*: 代币铸造账户地址
- `params.closeOrder` *(string|PublicKey)*: 需要关闭的订单地址
- `params.lpPairs` *(Array)*: LP配对数组
- `params.sellTokenAmount` *(anchor.BN)*: 希望卖出的token数量
- `params.minSolOutput` *(anchor.BN)*: 卖出后最少得到的sol数量
- `params.payer` *(PublicKey)*: 支付者公钥
- `options.computeUnits` *(number)*: 计算单元限制，默认 1400000

**示例:**
```javascript
// 先获取订单数据
const ordersData = await sdk.fast.orders(mint, { type: 'down_orders' });
const lpPairs = sdk.fast.buildLpPairs(ordersData.data.orders);

const result = await sdk.trading.closeLong({
  mintAccount: "HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7",
  closeOrder: "E2T72D4wZdxHRjELN5VnRdcCvS4FPcYBBT3UBEoaC5cA",
  lpPairs: lpPairs,
  sellTokenAmount: new anchor.BN("1000000000"),
  minSolOutput: new anchor.BN("100000000"),
  payer: wallet.publicKey
});
```

### sdk.trading.closeShort() - 平仓做空

```javascript
await sdk.trading.closeShort(params, options)
```

**参数:**
- `params.mintAccount` *(string|PublicKey)*: 代币铸造账户地址
- `params.closeOrder` *(string|PublicKey)*: 需要关闭的订单地址
- `params.lpPairs` *(Array)*: LP配对数组
- `params.buyTokenAmount` *(anchor.BN)*: 希望买入的token数量
- `params.maxSolAmount` *(anchor.BN)*: 愿意给出的最大sol数量
- `params.payer` *(PublicKey)*: 支付者公钥
- `options.computeUnits` *(number)*: 计算单元限制，默认 1400000

**示例:**
```javascript
// 先获取订单数据
const ordersData = await sdk.fast.orders(mint, { type: 'up_orders' });
const lpPairs = sdk.fast.buildLpPairs(ordersData.data.orders);

const result = await sdk.trading.closeShort({
  mintAccount: "HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7",
  closeOrder: "E2T72D4wZdxHRjELN5VnRdcCvS4FPcYBBT3UBEoaC5cA",
  lpPairs: lpPairs,
  buyTokenAmount: new anchor.BN("1000000000"),
  maxSolAmount: new anchor.BN("100000000"),
  payer: wallet.publicKey
});
```

---

## Fast 模块 - 数据获取

### sdk.fast.mints() - 获取代币列表

```javascript
await sdk.fast.mints(options)
```

**参数:**
- `options.page` *(number)*: 页码，默认 1
- `options.limit` *(number)*: 每页数量，默认 10
- `options.sort_by` *(string)*: 排序方式，默认 'slot_asc'

**返回值:**
```javascript
{
  "success": true,
  "data": {
    "mints": ["56hfrQYiyRSUZdRKDuUvsqRik8j2UDW9kCisy7BiRxmg", "4RGUZQ7PGF2JQWXKxr9hybPs1ZY3LWDb4QQ7FSHx92g9"],
    "total": null,
    "page": 1,
    "limit": 10,
    "has_next": false,
    "has_prev": false,
    "next_cursor": null,
    "sort_by": "slot_asc"
  },
  "message": "Operation successful"
}
```

**示例:**
```javascript
const result = await sdk.fast.mints({ page: 1, limit: 10 });
const mintAddresses = result.data.mints; // 代币地址数组
```

### sdk.fast.mint_info() - 获取代币详情

```javascript
await sdk.fast.mint_info(mint)
```

**参数:**
- `mint` *(string|Array)*: 代币地址或地址数组

**返回值:**
```javascript
{
  "success": true,
  "data": {
    "details": [
      {
        "mint_account": "56hfrQYiyRSUZdRKDuUvsqRik8j2UDW9kCisy7BiRxmg",
        "payer": "Hfi9FpHeqAz8qih87NccCqRQY7VWs3JH8ixqqBXRBLH5",
        "curve_account": "4xPzWMvbGT2AMmCMcvCw3h3PA3iG6kNKixfJyZ45r2BA",
        "pool_token_account": "7FsTN832wYsfa1fThy4sKZiMNavVRJ6gPk3SXbEwcAXH",
        "pool_sol_account": "3j4PfGm7YNhpBLijBHgNXggvVzsDMYTJW5fNJQ6cV89N",
        "name": "Hello",
        "symbol": "HLO",
        "uri": "https://example.com/hello-token.json",
        "swap_fee": 250,
        "borrow_fee": 300,
        "fee_discount_flag": 2,
        "create_timestamp": 1755667440,
        "latest_price": "13514066072452801812769",
        "latest_trade_time": 1755667451,
        "total_sol_amount": 214303125000,
        "total_margin_sol_amount": 5837102249,
        "total_force_liquidations": 4,
        "total_close_profit": 1605153562,
        "created_by": "Hfi9FpHeqAz8qih87NccCqRQY7VWs3JH8ixqqBXRBLH5",
        "last_updated_at": "2025-08-20T05:24:12.327211706Z"
      }
    ],
    "total": 1
  },
  "message": "Operation successful"
}
```

**示例:**
```javascript
// 获取单个代币详情
const info = await sdk.fast.mint_info('56hfrQYiyRSUZdRKDuUvsqRik8j2UDW9kCisy7BiRxmg');
const details = info.data.details[0];

// 获取多个代币详情
const info = await sdk.fast.mint_info(['address1', 'address2']);
```

### sdk.fast.orders() - 获取订单数据

```javascript
await sdk.fast.orders(mint, options)
```

**参数:**
- `mint` *(string)*: 代币地址
- `options.type` *(string)*: 订单类型，"up_orders"（做空）或 "down_orders"（做多）
- `options.page` *(number)*: 页码，默认 1
- `options.limit` *(number)*: 每页数量，默认 500

**返回值:**
```javascript
{
  "success": true,
  "data": {
    "orders": [
      {
        "order_type": 2,
        "mint": "代币地址",
        "user": "用户地址",
        "lock_lp_start_price": "753522984132656210522",
        "lock_lp_end_price": "833102733432007194898",
        "lock_lp_sol_amount": 2535405978,
        "lock_lp_token_amount": 32000000000000,
        "start_time": 1755964862,
        "end_time": 1756137662,
        "margin_sol_amount": 1909140052,
        "borrow_amount": 32000000000000,
        "position_asset_amount": 656690798,
        "borrow_fee": 1200,
        "order_pda": "59yP5tpDP6DBcyy4mge9wKKKdLmk45Th4sbd6Un9LxVN"
      }
    ],
    // ... 更多订单
  }
}
```

**示例:**
```javascript
// 获取做多订单
const ordersData = await sdk.fast.orders(mint, { type: 'down_orders' });
const orders = ordersData.data.orders;
```

### sdk.fast.price() - 获取代币价格

```javascript
await sdk.fast.price(mint)
```

**参数:**
- `mint` *(string)*: 代币地址

**返回值:**
- `string`: 最新价格字符串

**示例:**
```javascript
// 获取代币最新价格
const price = await sdk.fast.price('56hfrQYiyRSUZdRKDuUvsqRik8j2UDW9kCisy7BiRxmg');
console.log('最新价格:', price); // "13514066072452801812769"
```

### sdk.fast.user_orders() - 获取用户订单

```javascript
await sdk.fast.user_orders(user, mint, options)
```

**参数:**
- `user` *(string)*: 用户地址
- `mint` *(string)*: 代币地址
- `options.page` *(number)*: 页码，默认 1
- `options.limit` *(number)*: 每页数量，默认 200
- `options.order_by` *(string)*: 排序方式，默认 'start_time_desc'

**示例:**
```javascript
const userOrders = await sdk.fast.user_orders(
  '8iGFeUkRpyRx8w5uoUMbfZepUr6BfTdPuJmqGoNBntdb',
  '4Kq51Kt48FCwdo5CeKjRVPodH1ticHa7mZ5n5gqMEy1X',
  { page: 1, limit: 200 }
);
const orders = userOrders.data.orders;
```

### sdk.fast.findPrevNext() - 查找订单前后节点

```javascript
sdk.fast.findPrevNext(orders, findOrderPda)
```

**参数:**
- `orders` *(Array)*: 订单数组
- `findOrderPda` *(string)*: 要查找的订单PDA地址

**返回值:**
```javascript
{
  prevOrder: Object|null,    // 前一个订单
  nextOrder: Object|null     // 下一个订单
}
```

**示例:**
```javascript
const ordersData = await sdk.fast.orders(mint, { type: 'down_orders' });
const result = sdk.fast.findPrevNext(ordersData.data.orders, 'E2T72D4wZdxHRjELN5VnRdcCvS4FPcYBBT3UBEoaC5cA');

if (result.prevOrder) {
  console.log('前一个订单:', result.prevOrder.order_pda);
}
```

### sdk.fast.buildLpPairs() - 构建LP配对数组

```javascript
sdk.fast.buildLpPairs(orders)
```

**参数:**
- `orders` *(Array)*: 订单数组

**返回值:**
```javascript
[
  { solAmount: anchor.BN, tokenAmount: anchor.BN },
  { solAmount: anchor.BN, tokenAmount: anchor.BN },
  // ... 最多20个配对
]
```

**示例:**
```javascript
const ordersData = await sdk.fast.orders(mint, { type: 'down_orders' });
const lpPairs = sdk.fast.buildLpPairs(ordersData.data.orders);
```

### sdk.fast.buildOrderAccounts() - 构建订单账户数组

```javascript
sdk.fast.buildOrderAccounts(orders)
```

**参数:**
- `orders` *(Array)*: 订单数组

**返回值:**
```javascript
[
  "4fvsPDNoRRacSzE3PkEuNQeTNWMaeFqGwUxCnEbR1Dzb",  // 订单地址
  "G4nHBYX8EbrP8r35pk5TfpvJZfGNyLnd4qsfT7ru5vLd",  // 订单地址
  // ...
  null,  // 空位
  null   // 空位（总共20个位置）
]
```

---

## Token 模块 - 代币管理

### sdk.token.create() - 创建新代币

```javascript
await sdk.token.create(params)
```

**参数:**
- `params.mint` *(Keypair)*: 代币 mint 密钥对
- `params.name` *(string)*: 代币名称
- `params.symbol` *(string)*: 代币符号
- `params.uri` *(string)*: 元数据 URI
- `params.payer` *(PublicKey)*: 创建者公钥

**返回值:**
```javascript
{
  transaction: Transaction,
  signers: [Keypair],           // mint keypair 需要作为签名者
  accounts: {
    mint: PublicKey,
    curveAccount: PublicKey,
    poolTokenAccount: PublicKey,
    poolSolAccount: PublicKey,
    payer: PublicKey
  }
}
```

**示例:**
```javascript
const { Keypair } = require('@solana/web3.js');

const mintKeypair = Keypair.generate();
const result = await sdk.token.create({
  mint: mintKeypair,
  name: "My Token",
  symbol: "MTK",
  uri: "https://example.com/token-metadata.json",
  payer: wallet.publicKey
});

// 签名并发送（需要 payer 和 mint 两个签名）
const signature = await connection.sendTransaction(
  result.transaction, 
  [wallet.payer, mintKeypair]
);
```

---

## Param 模块 - 参数管理

### sdk.param.createParams() - 创建合作伙伴参数

```javascript
await sdk.param.createParams(params)
```

**参数:**
- `params.partner` *(PublicKey)*: 合作伙伴公钥

**返回值:**
```javascript
{
  transaction: Transaction,
  signers: [],                  // 不需要额外签名者
  accounts: {
    partner: PublicKey,
    adminAccount: PublicKey,
    paramsAccount: PublicKey
  }
}
```

**示例:**
```javascript
const result = await sdk.param.createParams({
  partner: partnerPublicKey
});
```

### sdk.param.getParams() - 获取合作伙伴参数

```javascript
await sdk.param.getParams(partner)
```

**参数:**
- `partner` *(PublicKey)*: 合作伙伴公钥

**返回值:**
```javascript
{
  address: PublicKey,           // 参数账户地址
  data: Object                  // 参数数据
}
```

### sdk.param.getAdmin() - 获取Admin账户

```javascript
await sdk.param.getAdmin()
```

**返回值:**
```javascript
{
  address: PublicKey,           // Admin账户地址
  data: Object                  // Admin数据
}
```

### sdk.param.getParamsAddress() - 计算参数账户地址

```javascript
sdk.param.getParamsAddress(partner)
```

**参数:**
- `partner` *(PublicKey)*: 合作伙伴公钥

**返回值:** *(PublicKey)* - 参数账户地址

### sdk.param.getAdminAddress() - 计算Admin账户地址

```javascript
sdk.param.getAdminAddress()
```

**返回值:** *(PublicKey)* - Admin账户地址

---

## Simulator 模块 - 交易模拟

### sdk.simulator.simulateBuy() - 模拟买入分析

```javascript
await sdk.simulator.simulateBuy(mint, buySolAmount)
```

**参数:**
- `mint` *(string)*: 代币地址
- `buySolAmount` *(bigint|string|number)*: 购买SOL数量（u64格式，精度10^9）

**返回值:**
```javascript
{
  success: boolean,
  estimatedTokenOutput: string,     // 预估获得的代币数量
  priceImpact: string,             // 价格影响百分比
  fees: {
    swapFee: string,               // 交换手续费
    totalFee: string               // 总手续费
  },
  // ... 更多分析数据
}
```

**示例:**
```javascript
const analysis = await sdk.simulator.simulateBuy(
  '56hfrQYiyRSUZdRKDuUvsqRik8j2UDW9kCisy7BiRxmg',
  '1000000000'  // 1 SOL
);
```

### sdk.simulator.simulateSell() - 模拟卖出分析

```javascript
await sdk.simulator.simulateSell(mint, sellTokenAmount)
```

**参数:**
- `mint` *(string)*: 代币地址
- `sellTokenAmount` *(bigint|string|number)*: 卖出代币数量（u64格式，精度10^6）

**返回值:** 类似 `simulateBuy()`

### sdk.simulator.simulateLongStopLoss() - 模拟做多止损

```javascript
await sdk.simulator.simulateLongStopLoss(mint, buyTokenAmount, stopLossPrice, mintInfo, ordersData)
```

**参数:**
- `mint` *(string)*: 代币地址
- `buyTokenAmount` *(bigint|string|number)*: 准备开多买入的token数量
- `stopLossPrice` *(bigint|string|number)*: 用户希望设置的止损位
- `mintInfo` *(Object|null)*: 代币信息，默认 null
- `ordersData` *(Object|null)*: 订单数据，默认 null

**返回值:**
```javascript
{
  success: boolean,
  stopLossAnalysis: {
    canSetStopLoss: boolean,        // 是否可以设置止损
    recommendedPrice: string,       // 推荐止损价格
    riskLevel: string,              // 风险级别
    // ... 更多分析数据
  }
}
```

### sdk.simulator.simulateSellStopLoss() - 模拟做空止损

```javascript
await sdk.simulator.simulateSellStopLoss(mint, sellTokenAmount, stopLossPrice, mintInfo, ordersData)
```

**参数:** 类似 `simulateLongStopLoss()`

---

## Chain 模块 - 链上数据查询

Chain 模块提供直接从 Solana 链上读取账户数据的功能。在没有辅助服务器的情况下，可以通过此模块获取实时的链上数据，包括流动池状态、账户余额等信息。

**特点:**
- 🔗 **直接链上查询**: 无需依赖第三方API，直接从区块链读取数据
- ⚡ **并发优化**: 使用Promise.all并发查询多个账户，提升性能
- 📊 **完整数据**: 返回包含所有相关账户地址和余额的完整信息
- 🛡️ **错误处理**: 提供详细的错误信息和异常处理
- 🔄 **实时同步**: 数据与链上状态实时同步，确保准确性

**注意事项:**
- 在交易高峰期，链上查询可能会有延迟
- 建议在关键交易场景下使用Fast模块的API接口
- Chain模块适合用于监控、分析和非实时交易场景

### sdk.chain.getCurveAccount() - 获取完整流动池数据

这是Chain模块的核心方法，用于获取指定代币的完整借贷流动池账户数据。

```javascript
await sdk.chain.getCurveAccount(mint)
```

**参数:**
- `mint` *(string|PublicKey)*: 代币铸造账户地址

**返回值:**
```javascript
{
  // 核心储备数据 - Core Reserve Data
  lpTokenReserve: bigint,              // LP Token 储备量，流动性提供者代币的总储备
  lpSolReserve: bigint,                // LP SOL 储备量，流动性池中的 SOL 储备
  price: bigint,                       // 当前代币价格，基于 AMM 算法计算
  borrowTokenReserve: bigint,          // 借贷 Token 储备量，可借贷的代币储备
  borrowSolReserve: bigint,            // 借贷 SOL 储备量，可借贷的 SOL 储备

  // 费用和参数配置 - Fee and Parameter Configuration
  swapFee: number,                     // 交换费率，以基点表示 (如 100 = 1%)
  borrowFee: number,                   // 借贷费率，以基点表示
  feeDiscountFlag: boolean,            // 费用折扣标志，是否启用费用优惠
  feeSplit: number,                    // 费用分配比例，决定费用如何在不同接收方间分配
  borrowDuration: number,              // 借贷期限，以秒为单位
  bump: number,                        // curve_account PDA 的 bump seed

  // 账户地址 - Account Addresses
  baseFeeRecipient: string,            // 基础费用接收地址，接收基础交易费用
  feeRecipient: string,                // 费用接收地址，接收额外费用收入
  mint: string,                        // 代币铸造账户地址
  upHead: string|null,                 // 上行订单链表头部账户地址，如果无则为 null
  downHead: string|null,               // 下行订单链表头部账户地址，如果无则为 null
  poolTokenAccount: string,            // 流动池代币账户地址，存储流动池中的代币
  poolSolAccount: string,              // 流动池 SOL 账户地址，存储流动池中的原生 SOL

  // 余额信息 - Balance Information
  baseFeeRecipientBalance: number,     // 基础费用接收地址的 SOL 余额 (lamports)
  feeRecipientBalance: number,         // 费用接收地址的 SOL 余额 (lamports)
  poolTokenBalance: bigint,            // 流动池代币账户的代币余额
  poolSolBalance: number,              // 流动池 SOL 账户的 SOL 余额 (lamports)

  // 元数据 - Metadata
  _metadata: {
    accountAddress: string,            // curve_account 的完整地址
    mintAddress: string                // 输入的代币铸造地址
  }
}
```

**完整示例:**
```javascript
try {
  // 获取完整的流动池数据
  const curveData = await sdk.chain.getCurveAccount('3YggGtxXEGBbjK1WLj2Z79doZC2gkCWXag1ag8BD4cYY');
  
  // === 显示核心储备信息 ===
  console.log('=== 核心储备数据 Core Reserve Data ===');
  console.log('LP Token 储备量:', curveData.lpTokenReserve.toString());
  console.log('LP SOL 储备量:', (Number(curveData.lpSolReserve) / 1e9).toFixed(4), 'SOL');
  console.log('当前价格:', curveData.price.toString());
  console.log('借贷 Token 储备量:', curveData.borrowTokenReserve.toString());
  console.log('借贷 SOL 储备量:', (Number(curveData.borrowSolReserve) / 1e9).toFixed(4), 'SOL');
  
  // === 显示费用配置 ===
  console.log('=== 费用配置 Fee Configuration ===');
  console.log('交换费率:', (curveData.swapFee / 100).toFixed(2), '%');
  console.log('借贷费率:', (curveData.borrowFee / 100).toFixed(2), '%');
  console.log('费用折扣:', curveData.feeDiscountFlag ? '启用' : '禁用');
  console.log('费用分配比例:', curveData.feeSplit);
  console.log('借贷期限:', curveData.borrowDuration, '秒');
  
  // === 显示账户地址 ===
  console.log('=== 账户地址 Account Addresses ===');
  console.log('基础费用接收地址:', curveData.baseFeeRecipient);
  console.log('费用接收地址:', curveData.feeRecipient);
  console.log('流动池代币账户:', curveData.poolTokenAccount);
  console.log('流动池 SOL 账户:', curveData.poolSolAccount);
  
  // === 显示余额信息 ===
  console.log('=== 余额信息 Balance Information ===');
  console.log('基础费用接收地址余额:', (curveData.baseFeeRecipientBalance / 1e9).toFixed(6), 'SOL');
  console.log('费用接收地址余额:', (curveData.feeRecipientBalance / 1e9).toFixed(6), 'SOL');
  console.log('流动池代币余额:', curveData.poolTokenBalance.toString());
  console.log('流动池 SOL 余额:', (curveData.poolSolBalance / 1e9).toFixed(6), 'SOL');
  
  // === 显示链表头部信息 ===
  console.log('=== 订单链表 Order Linked Lists ===');
  console.log('上行订单头部:', curveData.upHead || '空');
  console.log('下行订单头部:', curveData.downHead || '空');
  
} catch (error) {
  console.error('获取 curve account 失败:', error.message);
}
```

**流动池监控示例:**
```javascript
// 监控流动池健康状态的实用函数
async function monitorPoolHealth(mintAddress) {
  try {
    const data = await sdk.chain.getCurveAccount(mintAddress);
    
    // 计算流动池利用率
    const tokenUtilization = Number(data.lpTokenReserve - data.poolTokenBalance) / Number(data.lpTokenReserve);
    const solUtilization = Number(data.lpSolReserve - BigInt(data.poolSolBalance)) / Number(data.lpSolReserve);
    
    // 计算总费用收入
    const totalFeeBalance = data.baseFeeRecipientBalance + data.feeRecipientBalance;
    
    // 评估流动性状态
    const liquidityStatus = {
      tokenUtilization: (tokenUtilization * 100).toFixed(2) + '%',
      solUtilization: (solUtilization * 100).toFixed(2) + '%',
      totalFeeEarnings: (totalFeeBalance / 1e9).toFixed(6) + ' SOL',
      currentPrice: data.price.toString(),
      isHealthy: tokenUtilization < 0.9 && solUtilization < 0.9,  // 利用率低于90%为健康
      hasOrders: Boolean(data.upHead || data.downHead)
    };
    
    console.log('流动池健康状态:', liquidityStatus);
    
    return liquidityStatus;
    
  } catch (error) {
    console.error('监控失败:', error.message);
    return null;
  }
}

// 批量监控多个代币的流动池状态
async function batchMonitorPools(mintAddresses) {
  const results = await Promise.allSettled(
    mintAddresses.map(mint => monitorPoolHealth(mint))
  );
  
  results.forEach((result, index) => {
    if (result.status === 'fulfilled' && result.value) {
      console.log(`代币 ${mintAddresses[index]}:`, result.value);
    } else {
      console.error(`代币 ${mintAddresses[index]} 监控失败`);
    }
  });
}
```

### sdk.chain.getCurveAccountBatch() - 批量获取流动池数据

```javascript
await sdk.chain.getCurveAccountBatch(mints)
```

**参数:**
- `mints` *(Array<string|PublicKey>)*: 代币地址数组

**返回值:**
```javascript
{
  success: Array,                      // 成功获取的数据数组
  errors: Array,                       // 失败的错误信息数组
  total: number,                       // 总数量
  successCount: number,                // 成功数量
  errorCount: number                   // 失败数量
}
```

**示例:**
```javascript
const curveDataList = await sdk.chain.getCurveAccountBatch([
  '3YggGtxXEGBbjK1WLj2Z79doZC2gkCWXag1ag8BD4cYY',
  'HZBos3RNhExDcAtzmdKXhTd4sVcQFBiT3FDBgmBBMk7'
]);

console.log('成功获取:', curveDataList.successCount);
console.log('失败数量:', curveDataList.errorCount);
curveDataList.success.forEach(data => {
  console.log('代币:', data.mint, '价格:', data.price.toString());
});
```

### sdk.chain.getCurveAccountAddress() - 计算流动池地址

```javascript
sdk.chain.getCurveAccountAddress(mint)
```

**参数:**
- `mint` *(string|PublicKey)*: 代币铸造账户地址

**返回值:** *(PublicKey)* - curve_account 的 PDA 地址

**示例:**
```javascript
const curveAddress = sdk.chain.getCurveAccountAddress('3YggGtxXEGBbjK1WLj2Z79doZC2gkCWXag1ag8BD4cYY');
console.log('Curve Account 地址:', curveAddress.toString());
```

**Chain 模块使用场景:**

1. **实时监控**: 监控流动池状态、余额变化、费用收入
2. **数据分析**: 分析价格趋势、利用率、交易活跃度
3. **风险管理**: 检查流动性健康状态、识别异常情况
4. **开发调试**: 在开发过程中验证合约状态和数据正确性
5. **离线分析**: 获取历史数据进行后续分析和报告生成

**性能优化建议:**

- 使用批量查询方法减少网络请求
- 在高频查询场景下考虑使用Fast模块的API
- 合理设置查询间隔，避免过度频繁的链上查询
- 使用错误重试机制处理网络波动

---

## 工具方法

### 网络配置

```javascript
const { getDefaultOptions } = require('spinpet-sdk/src/utils/constants');

// 获取默认配置
const options = getDefaultOptions('MAINNET');  // 'MAINNET' | 'TESTNET' | 'LOCALNET'
```

**可用网络:**
- `MAINNET`: 主网配置
- `TESTNET`: 测试网配置  
- `LOCALNET`: 本地网络配置

---

## 完整使用示例

```javascript
const { Connection, PublicKey } = require('@solana/web3.js');
const anchor = require('@coral-xyz/anchor');
const SpinPetSdk = require('spinpet-sdk');
const { getDefaultOptions } = require('spinpet-sdk/src/utils/constants');

async function example() {
  // 1. 创建连接和钱包
  const connection = new Connection('http://localhost:8899', 'confirmed');
  const wallet = new anchor.Wallet(keypair);
  
  // 2. 获取默认配置
  const options = getDefaultOptions('LOCALNET');
  
  // 3. 初始化 SDK
  const sdk = new SpinPetSdk(connection, wallet, programId, options);
  
  // 4. 使用各种功能
  
  // 获取代币列表
  const mints = await sdk.fast.mints();
  console.log('代币列表:', mints.data.mints);
  
  // 获取代币详情
  const mintInfo = await sdk.fast.mint_info(mints.data.mints[0]);
  console.log('代币详情:', mintInfo.data.details[0]);
  
  // 模拟买入
  const buyAnalysis = await sdk.simulator.simulateBuy(
    mints.data.mints[0], 
    '1000000000'  // 1 SOL
  );
  console.log('买入分析:', buyAnalysis);
  
  // 执行买入交易
  const buyResult = await sdk.trading.buy({
    mintAccount: mints.data.mints[0],
    buyTokenAmount: new anchor.BN("1000000000"),
    maxSolAmount: new anchor.BN("2000000000"),
    payer: wallet.publicKey
  });
  
  // 签名并发送交易
  const signature = await connection.sendTransaction(buyResult.transaction, [wallet.payer]);
  console.log('交易签名:', signature);
}
```

---

## 注意事项

1. **数值精度**: 所有涉及金额的参数都需要使用 `anchor.BN` 类型，注意 SOL 精度为 10^9，代币精度通常为 10^6 或 10^9。

2. **交易签名**: SDK 返回的 `transaction` 对象需要用户自行签名并发送，SDK 不会自动执行交易。

3. **订单查询**: 在执行平仓操作前，需要先通过 `sdk.fast.orders()` 获取订单数据，并使用工具方法处理。

4. **网络配置**: 不同网络环境需要使用对应的配置参数，建议使用 `getDefaultOptions()` 获取。

5. **错误处理**: 所有异步方法都可能抛出异常，建议使用 try-catch 进行错误处理。