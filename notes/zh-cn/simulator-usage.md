# SpinPet SDK Simulator 模块使用指南

## 概述

Simulator 模块提供交易模拟功能，让用户在真实交易之前预览市场信息和交易结果。

## simulateBuy 函数

### 功能描述
模拟买入交易，分析价格滑点、流动性深度、交易完成率等关键指标。

### 函数签名
```javascript
async simulateBuy(mint, buySolAmount)
```

### 参数说明
- `mint` (string): 代币地址
- `buySolAmount` (bigint|string|number): 购买SOL数量，单位为 lamports (1 SOL = 1,000,000,000 lamports)

### 返回值格式
```javascript
{
    success: boolean,           // 是否成功
    errorCode: string|null,     // 错误代码 ('API_ERROR', 'DATA_ERROR', 'PARAM_ERROR')
    errorMessage: string|null,  // 错误信息
    data: {                     // 分析结果数据
        inputType: 'sol',                          // 输入类型
        inputAmount: bigint,                       // 输入数量
        maxAllowedPrice: bigint,                   // 允许的最高起始价格
        totalPriceSpan: bigint,                    // 交易价格区间
        transactionCompletionRate: number,         // 交易完成率 (%)
        idealTokenAmount: bigint,                  // 理想Token数量
        idealSolAmount: bigint,                    // 理想SOL数量
        actualRequiredSolAmount: bigint,           // 实际需要SOL数量
        actualObtainableTokenAmount: bigint,       // 实际可得Token数量
        theoreticalSolAmount: bigint,              // 理论所需SOL数量
        minimumSlippagePercentage: number,         // 最小滑点百分比
        totalLiquiditySolAmount: bigint,           // 总流动性SOL数量
        totalLiquidityTokenAmount: bigint          // 总流动性Token数量
    }
}
```

### 使用示例

```javascript
const { SpinPetSdk, getDefaultOptions, SPINPET_PROGRAM_ID } = require('spin-sdk');
const { Connection, Keypair, PublicKey } = require('@solana/web3.js');

// 1. 初始化 SDK
const options = getDefaultOptions('LOCALNET');
const connection = new Connection(options.solanaEndpoint, 'confirmed');
const sdk = new SpinPetSdk(connection, wallet, new PublicKey(SPINPET_PROGRAM_ID), options);

// 2. 模拟买入
const result = await sdk.simulator.simulateBuy(
    '2e3ss97Ynp16rrCTBxKLv2f8s9tkNWQmkebgC6GR7xf9', // 代币地址
    1000000000n  // 1 SOL
);

// 3. 分析结果
if (result.success) {
    const data = result.data;
    console.log('交易完成率:', data.transactionCompletionRate.toFixed(2), '%');
    console.log('最小滑点:', data.minimumSlippagePercentage.toFixed(2), '%');
    console.log('可得Token数量:', data.actualObtainableTokenAmount.toString());
} else {
    console.log('模拟失败:', result.errorMessage);
}
```

## simulateSell 函数

### 功能描述
模拟卖出交易，分析价格滑点、流动性深度、交易完成率等关键指标。

### 函数签名
```javascript
async simulateSell(mint, sellTokenAmount)
```

### 参数说明
- `mint` (string): 代币地址
- `sellTokenAmount` (bigint|string|number): 卖出Token数量，单位为 lamports (1 Token = 1,000,000 lamports)

### 返回值格式
```javascript
{
    success: boolean,           // 是否成功
    errorCode: string|null,     // 错误代码 ('API_ERROR', 'DATA_ERROR', 'PARAM_ERROR')
    errorMessage: string|null,  // 错误信息
    data: {                     // 分析结果数据
        inputType: 'token',                        // 输入类型
        inputAmount: bigint,                       // 输入数量
        minAllowedPrice: bigint,                   // 允许的最低起始价格
        totalPriceSpan: bigint,                    // 交易价格区间
        transactionCompletionRate: number,         // 交易完成率 (%)
        idealSolAmount: bigint,                    // 理想SOL数量
        idealTokenAmount: bigint,                  // 理想Token数量
        actualObtainedSolAmount: bigint,           // 实际获得SOL数量
        actualConsumedTokenAmount: bigint,         // 实际消耗Token数量
        theoreticalSolAmount: bigint,              // 理论目标SOL数量
        minimumSlippagePercentage: number,         // 最小滑点百分比
        totalLiquiditySolAmount: bigint,           // 总流动性SOL数量
        totalLiquidityTokenAmount: bigint          // 总流动性Token数量
    }
}
```

### 使用示例

```javascript
// 模拟卖出
const result = await sdk.simulator.simulateSell(
    '2e3ss97Ynp16rrCTBxKLv2f8s9tkNWQmkebgC6GR7xf9', // 代币地址
    100000000n  // 100 Token
);

// 分析结果
if (result.success) {
    const data = result.data;
    console.log('交易完成率:', data.transactionCompletionRate.toFixed(2), '%');
    console.log('最小滑点:', data.minimumSlippagePercentage.toFixed(2), '%');
    console.log('可得SOL数量:', data.actualObtainedSolAmount.toString());
} else {
    console.log('模拟失败:', result.errorMessage);
}
```

## 结果解读

### 交易完成率 (transactionCompletionRate)
- **100%**: 交易可以完全完成
- **< 100%**: 流动性不足，只能部分完成

### 滑点 (minimumSlippagePercentage)
- **< 1%**: 滑点很小，交易成本较低
- **1-5%**: 滑点适中，交易成本可接受
- **> 5%**: 滑点较大，交易成本较高

### 流动性深度
通过流动性数量与输入金额的比值判断：
- **比值 > 10**: 流动性充足，市场深度良好
- **比值 2-10**: 流动性一般，需要注意大额交易
- **比值 < 2**: 流动性不足，建议分批交易

### 往返分析
通过对比买入和卖出的模拟结果，可以分析往返交易的成本：
- **往返损失 < 1%**: 流动性良好，交易成本低
- **往返损失 1-5%**: 交易成本适中，需要注意
- **往返损失 > 5%**: 交易成本较高，建议谨慎

### 错误处理

函数不会抛出异常，所有错误都通过返回值的 `errorCode` 和 `errorMessage` 字段返回：

- `PARAM_ERROR`: 参数错误（如无效的代币地址或金额）
- `API_ERROR`: API 调用失败（如网络错误或服务不可用）
- `DATA_ERROR`: 数据处理错误（如价格数据异常）

### 注意事项

1. **网络要求**: 需要配置正确的 Fast API 端点
2. **数据实时性**: 模拟结果基于当前市场数据，实际交易时可能有差异
3. **精度**: 所有金额都使用 bigint 类型以保证精度
4. **性能**: 函数会调用多个 API，建议适当控制调用频率

## 运行测试

```bash
# 运行 simulateBuy 测试
node tests/example-simulator-buy.js

# 运行 simulateSell 测试
node tests/example-simulator-sell.js
```

测试将演示：
- 基本的 simulateBuy 和 simulateSell 功能
- 多种交易金额的对比分析
- 买入卖出对比和往返分析
- 结果解读和建议
