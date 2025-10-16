# simulateLongStopLoss 函数使用说明

## 功能概述 / Function Overview

`simulateLongStopLoss` 函数用于模拟计算做多时的止损位。该函数会根据用户设定的止损价格，自动调整到一个合理的、不与现有订单冲突且满足流动性预留要求的止损价格。

## 函数签名 / Function Signature

```javascript
async simulateLongStopLoss(mint, buyTokenAmount, stopLossPrice, mintInfo = null, ordersData = null)
```

## 参数说明 / Parameters

| 参数名 / Parameter | 类型 / Type | 说明 / Description |
|-------------------|-------------|-------------------|
| `mint` | `string` | 代币地址 / Token address |
| `buyTokenAmount` | `bigint\|string\|number` | 准备开多买入的token数量 (u64格式，精度10^6) / Token amount to buy for long position |
| `stopLossPrice` | `bigint\|string\|number` | 用户希望设置的止损位 (u128格式) / User desired stop loss price |
| `mintInfo` | `Object\|null` | 代币信息，默认null。如果为null会自动获取 / Token info, auto-fetch if null |
| `ordersData` | `Object\|null` | 订单数据，默认null。如果为null会自动获取 / Orders data, auto-fetch if null |

## 返回值 / Return Value

函数返回一个包含以下结构的对象：

```javascript
{
    success: boolean,           // 是否成功 / Success status
    errorCode: string|null,     // 错误代码 / Error code
    errorMessage: string|null,  // 错误信息 / Error message
    data: {                     // 成功时的数据 / Data when successful
        executableStopLossPrice: bigint,    // 计算后的合理止损价格 / Calculated reasonable stop loss price
        stopLossPercentage: number,         // 相对当前价格的止损百分比 / Stop loss percentage relative to current price
        leverage: number,                   // 杠杆比例 / Leverage ratio
        currentPrice: bigint,               // 当前价格 / Current price
        originalStopLossPrice: bigint,      // 用户原始设定的止损价格 / User's original stop loss price
        adjustmentAttempts: number          // 调整尝试次数 / Number of adjustment attempts
    }
}
```

## 使用示例 / Usage Examples

### 基本用法 / Basic Usage

```javascript
const SpinSDK = require('./src/sdk');

const sdk = new SpinSDK({
    rpcUrl: 'https://api.mainnet-beta.solana.com',
    wsUrl: 'wss://api.mainnet-beta.solana.com',
    apiUrl: 'https://api.spinpet.xyz'
});

async function example() {
    const result = await sdk.simulator.simulateLongStopLoss(
        'So11111111111111111111111111111111111111112', // SOL mint
        1000000n,                                      // 1 Token (10^6 precision)
        100000000000000000000000000000n                // 止损价格 / Stop loss price
    );

    if (result.success) {
        console.log('执行的止损价格 / Executable stop loss price:', result.data.executableStopLossPrice.toString());
        console.log('止损百分比 / Stop loss percentage:', result.data.stopLossPercentage + '%');
        console.log('杠杆比例 / Leverage:', result.data.leverage + 'x');
    } else {
        console.error('错误 / Error:', result.errorMessage);
    }
}
```

### 使用预获取的数据 / Using Pre-fetched Data

```javascript
async function exampleWithPreFetchedData() {
    const mint = 'So11111111111111111111111111111111111111112';
    
    // 预先获取数据 / Pre-fetch data
    const mintInfo = await sdk.fast.mint_info(mint);
    const ordersData = await sdk.fast.orders(mint, { type: 'down_orders' });
    
    // 使用预获取的数据 / Use pre-fetched data
    const result = await sdk.simulator.simulateLongStopLoss(
        mint,
        1000000n,
        100000000000000000000000000000n,
        mintInfo,
        ordersData
    );
    
    console.log('结果 / Result:', result);
}
```

## 算法说明 / Algorithm Description

### 1. 参数验证 / Parameter Validation
- 验证所有必需参数的有效性
- 转换参数为正确的数据类型

### 2. 数据获取 / Data Acquisition
- 如果未提供，自动获取代币信息和订单数据
- 获取当前价格和做多订单列表

### 3. 止损价格调整 / Stop Loss Price Adjustment
- 使用双向搜索算法（向上和向下调整）
- 每次调整0.5%的价格
- 最大尝试200次

### 4. 冲突检测 / Conflict Detection
- 检查止损区间与现有订单区间是否有交集
- 使用区间交集算法进行判断

### 5. 流动性预留验证 / Liquidity Reservation Validation
- 根据流动性空间预留验证机制进行验证
- 确保满足预留比例要求（默认100%）

### 6. 结果计算 / Result Calculation
- 计算止损百分比：`(当前价格 - 止损价格) / 当前价格 * 100`
- 计算杠杆比例：`当前价格 / (当前价格 - 止损价格)`

## 错误处理 / Error Handling

| 错误代码 / Error Code | 说明 / Description |
|---------------------|-------------------|
| `PARAM_ERROR` | 参数错误 / Parameter error |
| `API_ERROR` | API调用错误 / API call error |
| `CALCULATION_ERROR` | 计算错误（无法找到有效止损价格）/ Calculation error |
| `DATA_ERROR` | 数据处理错误 / Data processing error |

## 注意事项 / Notes

1. **价格精度**：止损价格使用u128格式，token数量使用u64格式（精度10^6）
2. **网络调用**：如果不提供mintInfo和ordersData，函数会进行网络调用获取数据
3. **性能考虑**：为了避免重复网络调用，建议在批量操作时预先获取数据
4. **调整限制**：最大调整尝试次数为200次，如果仍无法找到有效价格会返回错误

## 相关文档 / Related Documentation

- [流动性空间预留验证机制](./流动性空间预留验证机制.md)
- [API文档](./API_DOC_CN.md)
- [工程说明](./工程说明.md)
