# simulateSellStopLoss 函数说明 / simulateSellStopLoss Function Documentation

## 功能概述 / Function Overview

`simulateSellStopLoss` 函数用于模拟计算做空时的止损位。该函数会分析当前市场价格、已存在的订单，并计算出一个合理的止损价格，确保不会与现有订单产生冲突。

The `simulateSellStopLoss` function is used to simulate stop loss calculation for short positions. It analyzes current market prices, existing orders, and calculates a reasonable stop loss price to ensure no conflicts with existing orders.

## 函数签名 / Function Signature

```javascript
async function simulateSellStopLoss(
    mint, 
    sellTokenAmount, 
    stopLossPrice, 
    mintInfo = null, 
    ordersData = null
)
```

## 参数说明 / Parameters

| 参数名 / Parameter | 类型 / Type | 必填 / Required | 说明 / Description |
|-------------------|-------------|-----------------|-------------------|
| `mint` | `string` | ✅ | 代币地址 / Token address |
| `sellTokenAmount` | `bigint\|string\|number` | ✅ | 准备开空卖出的token数量 (精度: 10^6) / Token amount to sell for short position (precision: 10^6) |
| `stopLossPrice` | `bigint\|string\|number` | ✅ | 用户希望设置的止损位 (精度: 10^28) / User desired stop loss price (precision: 10^28) |
| `mintInfo` | `Object\|null` | ❌ | 代币信息，默认null会自动获取 / Token info, default null will auto-fetch |
| `ordersData` | `Object\|null` | ❌ | 订单数据，默认null会自动获取 / Orders data, default null will auto-fetch |

## 返回值 / Return Value

```javascript
{
    executableStopLossPrice: bigint,    // 计算后给出合理的止损值 / Calculated reasonable stop loss value
    stopLossPercentage: number,         // 相对目前价格的止损百分比 / Stop loss percentage relative to current price
    leverage: number,                   // 杠杆比例 / Leverage ratio
    currentPrice: bigint,               // 当前价格 / Current price
    iterations: number,                 // 调整次数 / Number of adjustments
    originalStopLossPrice: bigint       // 原始止损价格 / Original stop loss price
}
```

## 工作原理 / How It Works

### 1. 数据获取 / Data Retrieval
- 如果没有提供 `mintInfo`，会自动调用 `this.sdk.fast.mint_info(mint)` 获取代币信息
- 如果没有提供 `ordersData`，会自动调用 `this.sdk.fast.orders(mint, { type: 'up_orders' })` 获取做空订单数据

### 2. 价格计算 / Price Calculation
- 获取当前价格：`currentPrice = BigInt(mintInfo.data.details[0].latest_price)`
- 如果当前价格为空，使用 `CurveAMM.getInitialPrice()` 作为初始价格

### 3. 订单处理 / Order Processing
- 转换订单数据：`upOrders = transformOrdersData(ordersData)`
- 获取已存在的做空订单列表

### 4. 止损价格调整 / Stop Loss Price Adjustment
- 初始化：`stopLossStartPrice = stopLossPrice`
- 计算结束价格：`stopLossEndPrice = CurveAMM.buyFromPriceWithTokenOutput(stopLossStartPrice, sellTokenAmount)`
- 检查价格区间重叠：`checkPriceRangeOverlap('up_orders', upOrders, stopLossStartPrice, stopLossEndPrice)`
- 如果有重叠，每次增加 0.5% 的价格，重新计算直到无重叠

### 5. 结果计算 / Result Calculation
- **可执行止损价格**：`executableStopLossPrice = stopLossStartPrice`
- **止损百分比**：`stopLossPercentage = (executableStopLossPrice - currentPrice) / currentPrice * 100`
- **杠杆比例**：`leverage = currentPrice / (executableStopLossPrice - currentPrice)`

## 使用示例 / Usage Example

```javascript
const { SpinPetSdk } = require('spin-sdk');

// 初始化SDK
const sdk = new SpinPetSdk(connection, wallet, programId, options);

// 调用函数
const result = await sdk.simulator.simulateSellStopLoss(
    'CHqBH3UXXnqFhq2oSuFmYMyK6x7qYEB5gjuKtwY4zSj3', // mint
    1000000, // 1 Token (10^6 precision)
    32598759341402128276409n // 止损价格
);

console.log('可执行止损价格:', result.executableStopLossPrice.toString());
console.log('止损百分比:', result.stopLossPercentage + '%');
console.log('杠杆比例:', result.leverage + 'x');
```

## 测试 / Testing

运行测试文件：
```bash
node tests/test-simulate-sell-stop-loss.js
```

测试包括：
- 参数验证测试 / Parameter validation test
- 基本功能测试 / Basic functionality test
- 做空止损逻辑测试 / Short position stop loss logic test

## 注意事项 / Notes

1. **做空逻辑**：做空时，止损价格应该高于当前价格
2. **价格精度**：所有价格计算都使用 BigInt 以确保精度
3. **重叠检测**：函数会自动调整价格以避免与现有订单重叠
4. **最大迭代**：为防止无限循环，最多迭代 1000 次
5. **价格范围**：止损价格不能超过 `CurveAMM.MAX_U128_PRICE`

## 错误处理 / Error Handling

函数会抛出以下错误：
- `缺少必要参数`：当必要参数为空时
- `获取代币信息失败`：当无法获取代币信息时
- `获取订单数据失败`：当无法获取订单数据时
- `计算止损结束价格失败`：当无法计算交易结果时
- `止损价格调整后超过最大值`：当价格调整超出范围时
- `达到最大迭代次数`：当无法找到合适的止损价格时
