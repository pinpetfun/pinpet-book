# 交易模块 - long 函数使用说明

## 概述

`long` 函数是 SpinPet SDK 交易模块中的保证金做多功能，允许用户通过提供保证金来开立做多仓位。

## 函数签名

```javascript
async long({ 
  mintAccount, 
  buyTokenAmount, 
  maxSolAmount, 
  marginSol, 
  closePrice, 
  payer 
}, options = {})
```

## 参数说明

### 必需参数

- **mintAccount** (`string|PublicKey`): 代币铸造账户地址
- **buyTokenAmount** (`anchor.BN`): 要购买的代币数量（6位精度）
- **maxSolAmount** (`anchor.BN`): 最大 SOL 花费（9位精度）
- **marginSol** (`anchor.BN`): 保证金数量（9位精度）
- **closePrice** (`anchor.BN`): 平仓价格（u128格式）
- **payer** (`PublicKey`): 支付者公钥

### 可选参数

- **options.computeUnits** (`number`): 计算单元限制，默认 1400000

## 返回值

返回一个包含以下信息的对象：

```javascript
{
  transaction,        // Transaction 对象
  signers: [],        // 签名者数组（空数组，只需要 payer 签名）
  accounts: {         // 相关账户信息
    mint,             // 代币铸造账户
    curveAccount,     // 曲线账户
    poolTokenAccount, // 池子代币账户
    poolSolAccount,   // 池子SOL账户
    userTokenAccount, // 用户代币账户
    payer,            // 支付者
    selfOrder         // 做多订单账户
  },
  orderData: {        // 订单数据
    ordersUsed,       // 使用的订单数量
    lpPairsCount,     // LP配对数量
    lpPairs,          // LP配对数组
    orderAccounts,    // 订单账户数组
    uniqueSeed,       // 唯一种子
    prevOrder,        // 前一个订单
    nextOrder         // 下一个订单
  }
}
```

## 使用示例

### 基本用法

```javascript
const { SpinPetSdk } = require('spin-sdk');
const anchor = require('@coral-xyz/anchor');

// 创建 SDK 实例
const sdk = new SpinPetSdk(connection, wallet, programId, options);

// 设置做多参数
const params = {
  mintAccount: "4Kq51Kt48FCwdo5CeKjRVPodH1ticHa7mZ5n5gqMEy1X",
  buyTokenAmount: new anchor.BN("10000000"),    // 10 tokens
  maxSolAmount: new anchor.BN("1100000000"),    // 1.1 SOL
  marginSol: new anchor.BN("2200000000"),       // 2.2 SOL
  closePrice: new anchor.BN("1000000000000000"), // 平仓价格
  payer: wallet.publicKey
};

// 调用 long 函数
const result = await sdk.trading.long(params);

// 发送交易
const signature = await sdk.connection.sendTransaction(
  result.transaction, 
  [wallet]
);

console.log('交易签名:', signature);
```

### 使用初始价格作为平仓价格

```javascript
const CurveAMM = require('spin-sdk/src/utils/curve_amm');

const params = {
  mintAccount: "4Kq51Kt48FCwdo5CeKjRVPodH1ticHa7mZ5n5gqMEy1X",
  buyTokenAmount: new anchor.BN("10000000"),
  maxSolAmount: new anchor.BN("1100000000"),
  marginSol: new anchor.BN("2200000000"),
  closePrice: new anchor.BN(CurveAMM.getInitialPrice().toString()),
  payer: wallet.publicKey
};

const result = await sdk.trading.long(params);
```

## 参数计算建议

### 1. buyTokenAmount（购买代币数量）
- 使用 6 位精度
- 例如：`10000000` 表示 10 个代币

### 2. maxSolAmount（最大SOL花费）
- 使用 9 位精度
- 建议设置为预期花费的 110%，以应对滑点
- 例如：`1100000000` 表示 1.1 SOL

### 3. marginSol（保证金数量）
- 使用 9 位精度
- 保证金决定了杠杆倍数
- 例如：`2200000000` 表示 2.2 SOL

### 4. closePrice（平仓价格）
- 使用 u128 格式
- 可以通过 `CurveAMM.getInitialPrice()` 获取初始价格
- 或者根据市场情况设置合适的止损价格

## 注意事项

1. **精度要求**: 所有数值参数必须使用正确的精度格式
2. **账户检查**: 函数会自动检查并创建用户代币账户
3. **订单数据**: 函数会自动获取和构建所需的订单数据
4. **PDA 生成**: 做多订单账户的 PDA 会根据用户地址、代币地址和随机种子生成
5. **交易签名**: 只需要支付者签名，不需要额外的签名者

## 错误处理

常见错误及解决方案：

- **参数类型错误**: 确保所有参数都是正确的 `anchor.BN` 类型
- **账户不存在**: 函数会自动处理代币账户的创建
- **网络错误**: 检查网络连接和 API 端点配置
- **余额不足**: 确保支付者有足够的 SOL 支付交易费用和保证金

## 测试示例

参考 `tests/example-trading-long.js` 文件中的完整测试示例，包括：

- 基本功能测试
- 多个测试用例
- 错误处理演示
- 参数验证

## 相关文件

- 源代码: `src/modules/trading.js`
- 测试文件: `tests/example-trading-long.js`
- 工具类: `src/utils/curve_amm.js`
- 模拟器: `src/modules/simulator/long_shrot_stop.js`
