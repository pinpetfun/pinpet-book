# SpinPet Trading Flow Diagrams

This document provides detailed flowcharts explaining the execution processes of various trading methods on the SpinPet platform, helping users and developers understand the system's operational mechanisms.

## Table of Contents
1. [Spot Buy Trading Flow](#spot-buy-trading-flow)
2. [Spot Sell Trading Flow](#spot-sell-trading-flow)
3. [Leveraged Long Trading Flow](#leveraged-long-trading-flow)
4. [Leveraged Short Trading Flow](#leveraged-short-trading-flow)
5. [Active Position Closing Flow](#active-position-closing-flow)
6. [Forced Liquidation Flow](#forced-liquidation-flow)
7. [System Architecture Diagram](#system-architecture-diagram)

---

## Spot Buy Trading Flow

Spot buying is the most basic trading method, where users directly purchase tokens with SOL.

```mermaid
flowchart TD
    A[User initiates buy request] --> B{Validate transaction parameters}
    B -->|Invalid parameters| C[Return error message]
    B -->|Valid parameters| D[Get current price and reserves]
    
    D --> E[Check if short positions need liquidation]
    E -->|Positions need liquidation| F[Execute forced liquidation]
    E -->|No liquidation needed| G[Calculate required SOL]
    F --> G
    
    G --> H{Check user SOL balance}
    H -->|Insufficient balance| I[Transaction failed]
    H -->|Sufficient balance| J[Calculate slippage and fees]
    
    J --> K{Is slippage acceptable}
    K -->|Slippage too high| L[Transaction cancelled]
    K -->|Slippage acceptable| M[Deduct SOL from user account]
    
    M --> N[Update liquidity pool reserves]
    N --> O[Calculate new price]
    O --> P[Transfer tokens to user]
    P --> Q[Collect transaction fees]
    Q --> R[Record transaction log]
    R --> S[Transaction completed successfully]
    
    style A fill:#e1f5fe
    style S fill:#c8e6c9
    style C fill:#ffcdd2
    style I fill:#ffcdd2
    style L fill:#ffcdd2
```

### Key Steps Explanation:
1. **Parameter Validation**: Check buy amount, maximum SOL limit and other parameters
2. **Liquidation Check**: If price increase would trigger short position stop-loss, liquidation must occur first
3. **Slippage Protection**: Ensure actual price doesn't exceed user's acceptable range
4. **Atomic Execution**: All operations either succeed completely or rollback entirely

---

## Spot Sell Trading Flow

Spot selling is the process of users exchanging their held tokens back to SOL.

```mermaid
flowchart TD
    A[User initiates sell request] --> B{Validate user token balance}
    B -->|Insufficient balance| C[Return insufficient balance error]
    B -->|Sufficient balance| D[Get current price and reserves]
    
    D --> E[Check if long positions need liquidation]
    E -->|Positions need liquidation| F[Execute forced liquidation]
    E -->|No liquidation needed| G[Calculate receivable SOL amount]
    F --> G
    
    G --> H[Calculate transaction fees]
    H --> I[Calculate actual SOL received]
    I --> J{Check minimum output requirement}
    J -->|Below minimum| K[Transaction cancelled]
    J -->|Meets requirement| L[Deduct tokens from user account]
    
    L --> M[Update liquidity pool reserves]
    M --> N[Calculate new price]
    N --> O[Transfer SOL to user]
    O --> P[Collect transaction fees]
    P --> Q[Record transaction log]
    Q --> R[Transaction completed successfully]
    
    style A fill:#e1f5fe
    style R fill:#c8e6c9
    style C fill:#ffcdd2
    style K fill:#ffcdd2
```

### Key Steps Explanation:
1. **Balance Validation**: Ensure user has sufficient tokens for trading
2. **Liquidation Check**: Price decline may trigger long position stop-loss
3. **Minimum Output Protection**: Prevent user losses due to excessive slippage

---

## Leveraged Long Trading Flow

Leveraged long allows users to borrow funds to amplify buying power, expecting profit from price increases.

```mermaid
flowchart TD
    A[User initiates long request] --> B{Validate leverage parameters}
    B -->|Invalid parameters| C[Return parameter error]
    B -->|Valid parameters| D[Check lending pool SOL balance]
    
    D -->|Insufficient balance| E[Lending pool insufficient balance]
    D -->|Sufficient balance| F[Calculate required margin]
    
    F --> G{Is user margin sufficient}
    G -->|Insufficient| H[Insufficient margin error]
    G -->|Sufficient| I[Borrow SOL from lending pool]
    
    I --> J[Calculate total buying power<br/>Margin + Borrowed amount]
    J --> K[Execute token purchase<br/>Reference spot buy flow]
    K -->|Purchase failed| L[Return borrowed funds, transaction failed]
    K -->|Purchase successful| M[Calculate future closing cost]
    
    M --> N[Create long order record]
    N --> O[Set stop-loss price]
    O --> P[Insert into long order linked list]
    P --> Q[Transfer margin from user]
    Q --> R[Record lending information]
    R --> S[Long position opened successfully]
    
    style A fill:#e1f5fe
    style S fill:#c8e6c9
    style C fill:#ffcdd2
    style E fill:#ffcdd2
    style H fill:#ffcdd2
    style L fill:#ffcdd2
```

### Key Steps Explanation:
1. **Leverage Calculation**: Calculate borrowing amount based on user's selected multiplier
2. **Risk Assessment**: Ensure user has sufficient margin to bear potential losses
3. **Order Management**: Create linked list structure to manage all long orders
4. **Stop-loss Setting**: Automatically set reasonable stop-loss price to protect funds

---

## Leveraged Short Trading Flow

Leveraged short allows users to borrow tokens and sell them immediately, expecting profit when prices decline.

```mermaid
flowchart TD
    A[User initiates short request] --> B{Validate short parameters}
    B -->|Invalid parameters| C[Return parameter error]
    B -->|Valid parameters| D[Check lending pool token balance]
    
    D -->|Insufficient balance| E[Lending pool token insufficient]
    D -->|Sufficient balance| F[Calculate required margin]
    
    F --> G{Is user margin sufficient}
    G -->|Insufficient| H[Insufficient margin error]
    G -->|Sufficient| I[Borrow tokens from lending pool]
    
    I --> J[Immediately sell borrowed tokens<br/>Reference spot sell flow]
    J -->|Sell failed| K[Return tokens, transaction failed]
    J -->|Sell successful| L[Receive SOL proceeds]
    
    L --> M[Calculate future buyback cost]
    M --> N[Create short order record]
    N --> O[Set stop-loss price]
    O --> P[Insert into short order linked list]
    P --> Q[Transfer margin from user]
    Q --> R[Record lending information]
    R --> S[Short position opened successfully]
    
    style A fill:#e1f5fe
    style S fill:#c8e6c9
    style C fill:#ffcdd2
    style E fill:#ffcdd2
    style H fill:#ffcdd2
    style K fill:#ffcdd2
```

### Key Steps Explanation:
1. **Token Lending**: Borrow specified amount of tokens from token lending pool
2. **Immediate Sale**: Sell borrowed tokens immediately in the market to receive SOL
3. **Risk Control**: Set stop-loss price to prevent excessive losses from price increases
4. **Order Tracking**: Record lending information for subsequent closing operations

---

## Active Position Closing Flow

Users can actively close positions at any time to end leveraged trades and settle profits/losses.

```mermaid
flowchart TD
    A[User initiates closing request] --> B{Validate order ownership}
    B -->|Not user's order| C[Permission error]
    B -->|Validation passed| D[Get order information]
    
    D --> E{Determine order type}
    E -->|Long order| F[Sell held tokens]
    E -->|Short order| G[Buy tokens to repay loan]
    
    F --> H[Calculate sell proceeds]
    G --> I[Calculate buy cost]
    
    H --> J[Deduct loan principal]
    I --> K[Return borrowed tokens]
    
    J --> L[Deduct interest and fees]
    K --> M[Deduct interest and fees]
    
    L --> N[Calculate net profit/loss]
    M --> O[Calculate net profit/loss]
    
    N --> P[Remove from order linked list]
    O --> P
    P --> Q[Update lending pool balance]
    Q --> R{Profit or loss}
    R -->|Profit| S[Transfer profit to user]
    R -->|Loss| T[Deduct loss from margin]
    S --> U[Position closed successfully]
    T --> U
    
    style A fill:#e1f5fe
    style U fill:#c8e6c9
    style C fill:#ffcdd2
```

### Key Steps Explanation:
1. **Permission Validation**: Ensure only order owner can close position
2. **Reverse Operation**: Long closing requires selling, short closing requires buying
3. **Debt Settlement**: Prioritize repaying loan principal and interest
4. **Profit/Loss Settlement**: Calculate final profit or loss

---

## Forced Liquidation Flow

When stop-loss conditions are triggered or orders expire, the system automatically executes forced liquidation.

```mermaid
flowchart TD
    A[Liquidation condition triggered] --> B{Liquidation trigger type}
    B -->|Price trigger| C[Price breaks stop-loss line]
    B -->|Time trigger| D[Order expired]
    B -->|Manual trigger| E[Other user initiates forced liquidation]
    
    C --> F[Validate liquidation conditions]
    D --> F
    E --> F
    
    F -->|Conditions not met| G[Liquidation failed]
    F -->|Conditions met| H[Begin forced liquidation]
    
    H --> I{Order type}
    I -->|Long order| J[Force sell tokens]
    I -->|Short order| K[Force buy tokens]
    
    J --> L[Calculate sell proceeds]
    K --> M[Calculate buy cost]
    
    L --> N[Repay SOL loan]
    M --> O[Repay token loan]
    
    N --> P[Deduct interest and liquidation fees]
    O --> Q[Deduct interest and liquidation fees]
    
    P --> R[Calculate remaining margin]
    Q --> S[Calculate remaining margin]
    
    R --> T[Pay reward to liquidator]
    S --> T
    T --> U[Return remaining funds to user]
    U --> V[Remove order from linked list]
    V --> W[Update system state]
    W --> X[Forced liquidation completed]
    
    style A fill:#fff3e0
    style X fill:#c8e6c9
    style G fill:#ffcdd2
```

### Key Steps Explanation:
1. **Trigger Conditions**: Price stop-loss, order expiration, or manual trigger
2. **Priority Debt Repayment**: Ensure lender fund safety
3. **Incentive Mechanism**: Provide certain rewards to liquidation executors
4. **Fund Protection**: Try to protect user's remaining margin

---

## System Architecture Diagram

The overall SpinPet system architecture and component relationships.

```mermaid
graph TB
    subgraph "User Interface Layer"
        A[Web Frontend] 
        B[Mobile App]
        C[API Interface]
    end
    
    subgraph "Business Logic Layer"
        D[Spot Trading Module]
        E[Leverage Trading Module] 
        F[Order Management Module]
        G[Risk Control System]
    end
    
    subgraph "Core Algorithm Layer"
        H[CurveAMM Engine]
        I[Price Calculator]
        J[Slippage Protection]
        K[Fee Calculator]
    end
    
    subgraph "Data Storage Layer"
        L[User Accounts]
        M[Order Linked Lists]
        N[Liquidity Pools]
        O[Lending Pools]
    end
    
    subgraph "Blockchain Layer"
        P[Solana Program]
        Q[Token Program]
        R[System Program]
    end
    
    A --> D
    B --> E
    C --> F
    
    D --> H
    E --> H
    F --> G
    G --> I
    
    H --> L
    I --> M
    J --> N
    K --> O
    
    L --> P
    M --> Q
    N --> R
    O --> P
    
    style H fill:#e3f2fd
    style G fill:#fff3e0
    style P fill:#f3e5f5
```

### System Features:
1. **Layered Architecture**: Clear layered design for easy maintenance and expansion
2. **Modularization**: Relatively independent functional modules with reduced coupling
3. **Security**: Multi-layer risk control protection ensuring fund safety
4. **High Performance**: Based on Solana's high-speed blockchain infrastructure

---

## Trading Flow Summary

### Spot Trading Characteristics:
- **Immediacy**: Transactions execute immediately without waiting for matching
- **Transparency**: Prices determined entirely by algorithms, open and transparent
- **Liquidity**: 24/7 liquidity provision with no trading time restrictions

### Leveraged Trading Characteristics:
- **Amplification Effect**: Both returns and risks are amplified
- **Lending Mechanism**: Need to pay lending interest
- **Risk Control Protection**: Multiple security mechanisms to prevent risks

### Position Closing Mechanism Characteristics:
- **Flexibility**: Users can actively close positions at any time
- **Automation**: Automatic execution when trigger conditions are met
- **Protection**: Priority protection of lender and user funds

Through these flowcharts, users can clearly understand the execution steps and risk control mechanisms of each trading method, enabling them to make more informed trading decisions. 