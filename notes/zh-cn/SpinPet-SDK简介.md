# SpinPet SDK 简介 / SpinPet SDK Introduction

## 概述 / Overview

SpinPet SDK 是一个用于与 Solana Anchor 智能合约交互的 JavaScript SDK，专门为 SpinPet 协议设计。它支持 Node.js 和浏览器环境，提供模块化的交易、代币管理、订单管理等功能。

SpinPet SDK is a JavaScript SDK for interacting with Solana Anchor smart contracts related to the SpinPet protocol. It supports both Node.js and browser environments and provides modular functionality for trading, token management, order management, and more.

## 核心特性 / Core Features

### 🚀 模块化设计 / Modular Design
- **交易模块 (TradingModule)**: 买卖交易、多空交易功能
- **快速模块 (FastModule)**: API 数据获取和处理
- **代币模块 (TokenModule)**: 代币创建和管理
- **参数模块 (ParamModule)**: 协议参数配置
- **模拟器模块 (SimulatorModule)**: 交易模拟计算

### 🔧 跨平台兼容 / Cross-Platform Compatibility
- 支持 Node.js 服务器环境
- 支持浏览器前端环境
- 提供 CommonJS、ESM、UMD 多种构建格式

### 🛡️ 安全设计 / Security Design
- SDK 核心不进行签名操作
- 设计用于浏览器钱包集成
- 支持外部签名流程

## 快速开始 / Quick Start

### 安装 / Installation
```bash
npm install @spinpet/sdk
```

### 基本用法 / Basic Usage
```javascript
import { SpinPetSdk } from '@spinpet/sdk';
import { Connection, PublicKey } from '@solana/web3.js';

// 初始化连接 / Initialize connection
const connection = new Connection('https://api.mainnet-beta.solana.com');
const programId = new PublicKey('your-program-id');

// 创建 SDK 实例 / Create SDK instance
const sdk = new SpinPetSdk(connection, wallet, programId);

// 使用交易功能 / Use trading functionality
const buyTx = await sdk.trading.buy(mintPubkey, amountIn, minAmountOut);
```

## 主要模块介绍 / Main Modules

### 交易模块 / Trading Module
```javascript
// 买入交易 / Buy transaction
await sdk.trading.buy(mint, amountIn, minAmountOut);

// 卖出交易 / Sell transaction  
await sdk.trading.sell(mint, amountIn, minAmountOut);

// 开多头 / Open long position
await sdk.trading.long(mint, amountIn, borrowAmount);

// 开空头 / Open short position
await sdk.trading.short(mint, amountIn, borrowAmount);
```

### 快速数据模块 / Fast Data Module
```javascript
// 获取订单数据 / Get order data
const orders = await sdk.fast.orders(mint);

// 获取代币信息 / Get mint info
const mintInfo = await sdk.fast.mintInfo(mint);

// 查找相关订单 / Find related orders
const { prev, next } = await sdk.fast.findPrevNext(orders, targetPrice);
```

### 模拟器模块 / Simulator Module
```javascript
// 模拟买入 / Simulate buy
const buyResult = await sdk.simulator.simulateBuy(mint, amountIn);

// 模拟卖出 / Simulate sell
const sellResult = await sdk.simulator.simulateSell(mint, amountIn);
```

## 开发指南 / Development Guide

### 项目结构 / Project Structure
```
src/
├── sdk.js              # 主 SDK 类 / Main SDK class
├── modules/            # 功能模块 / Feature modules
│   ├── trading.js      # 交易模块 / Trading module
│   ├── fast.js         # 快速数据模块 / Fast data module
│   ├── token.js        # 代币模块 / Token module
│   ├── param.js        # 参数模块 / Parameter module
│   └── simulator.js    # 模拟器模块 / Simulator module
└── utils/              # 工具函数 / Utilities
    ├── constants.js    # 常量定义 / Constants
    └── curve_amm.js    # AMM 曲线计算 / AMM curve calculations
```

### 构建和测试 / Build and Test
```bash
# 安装依赖 / Install dependencies
npm install

# 构建项目 / Build project
npm run build

# 运行测试 / Run tests
node tests/example-trading-buy.js

# 代码检查 / Lint code
npm run lint
```

## 文档资源 / Documentation Resources

- [API 文档](./API_DOC_CN.md)
- [模拟器使用说明](./simulator-usage.md) 
- [交易模块详解](./交易模块-long函数使用说明.md)
- [工程说明](./工程说明.md)

## 支持与贡献 / Support and Contribution

如有问题或建议，请提交 Issue 或 Pull Request。

For questions or suggestions, please submit an Issue or Pull Request.

---

**版本 / Version**: 1.0.0  
**许可证 / License**: MIT  
**维护者 / Maintainer**: SpinPet Team