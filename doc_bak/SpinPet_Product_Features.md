# SpinPet - Decentralized Trading Product Features Documentation

## Product Overview

SpinPet is a decentralized trading platform based on the Solana blockchain that combines spot trading and leveraged trading functions, providing users with flexible investment and trading options. The platform adopts an Automated Market Maker (AMM) mechanism, allowing users to trade tokens and perform leveraged long/short operations without traditional order book matching.

## Core Features

- **Spot Trading**: Instant token buy/sell with abundant liquidity trading experience
- **Leveraged Trading**: Support 2-5x leverage for long/short positions, amplifying investment returns
- **Automated Market Maker**: Mathematical formula-based price discovery mechanism ensuring 24/7 liquidity
- **Lending Function**: Built-in lending pools supporting fund borrowing for leveraged trading
- **Auto Liquidation**: Intelligent risk control system protecting user and platform fund safety

## I. CurveAMM Principles

### 1.1 What is CurveAMM

CurveAMM is the core trading engine of the SpinPet platform, implementing automated market making based on the "constant product formula" (x × y = k). This formula ensures:
- The product of two token reserves remains constant at all times
- Prices are automatically adjusted by supply and demand
- 24-hour uninterrupted liquidity provision

### 1.2 Working Principles

**Reserve Pool Mechanism**:
- The platform maintains a reserve pool containing SOL and tokens
- Initial state: 30 SOL + 1,073,000,000 tokens
- When users buy tokens, SOL reserves increase while token reserves decrease
- When users sell tokens, token reserves increase while SOL reserves decrease

**Price Discovery**:
- Price = SOL Reserve Amount ÷ Token Reserve Amount
- Larger trading volumes have greater price impact (slippage)
- Initial price approximately 0.0000000279 SOL/token

**Mathematical Guarantee**:
- After each transaction, the k-value (reserve product) remains unchanged or slightly increases
- This ensures liquidity providers receive trading fee profits
- Prevents arbitragers from infinitely extracting value

### 1.3 Precision and Security

**Multi-layer Precision Control**:
- SOL uses 9 decimal places precision (compliant with Solana standard)
- Tokens use 6 decimal places precision
- Price uses 28-bit precision factor ensuring calculation accuracy

**Security Measures**:
- All calculations use overflow-checking secure algorithms
- Rounding bias factors protect liquidity pools from precision loss
- Minimum price limits prevent extreme situations

## II. Buy/Sell Trading Principles

### 2.1 Spot Buy Mechanism

**Basic Process**:
1. User specifies the desired token purchase amount
2. System calculates required SOL based on current price and slippage
3. Check if other users' stop-loss orders are triggered
4. Execute transaction and update reserve pool status
5. Collect trading fees

**Price Calculation Example**:
- Assume user wants to buy 1000 tokens
- Current price: 0.00003 SOL/token
- Theoretical cost: 1000 × 0.00003 = 0.03 SOL
- Actual cost will be slightly higher due to slippage, e.g., 0.031 SOL

**Stop-loss Order Processing**:
- If buying pushes price up to trigger other users' stop-loss prices
- System automatically liquidates related short orders
- Funds obtained from liquidation participate in current transaction
- Ensures market order and fund flow

### 2.2 Spot Sell Mechanism

**Basic Process**:
1. User specifies the token amount to sell
2. System calculates obtainable SOL amount (minus fees)
3. Check if long users' stop-loss is triggered
4. Execute transaction, user receives SOL
5. Update price and reserve status

**Slippage Protection**:
- Users can set minimum acceptable SOL amount
- If excessive slippage causes returns below expectations, transaction automatically cancels
- Protects users from malicious front-running attacks

**Fee Mechanism**:
- Trading fees deducted from output amount
- Fee rates can be dynamically adjusted (typically 0.1-1%)
- Fees used for platform operation maintenance and liquidity provider rewards

### 2.3 Price Impact and Slippage

**Slippage Calculation**:
- 0.1% trading volume: approximately 0.1% slippage
- 1% trading volume: approximately 1% slippage
- 10% trading volume: approximately 9% slippage
- 50% trading volume: approximately 33% slippage

**Best Practice Recommendations**:
- Large transactions recommended to be executed in batches to reduce slippage
- Monitor liquidity depth, avoid trading during low liquidity periods
- Set reasonable slippage tolerance

## III. Leveraged Trading Principles

### 3.1 Leveraged Long

**Basic Concept**:
Leveraged long refers to using less margin to borrow more funds to buy tokens, expecting profit from token price increases.

**Specific Process**:
1. **Opening Position Phase**:
   - User provides margin (e.g., 100 SOL)
   - Select leverage multiplier (e.g., 3x)
   - System borrows additional funds from lending pool (200 SOL)
   - Use total funds (300 SOL) to buy tokens
   - Establish long order record

2. **Position Holding Phase**:
   - User holds tokens purchased through leverage
   - Need to pay borrowing interest
   - Set stop-loss price to prevent excessive losses
   - Can actively close position at any time

3. **Position Closing Phase**:
   - Active closing: User chooses to sell tokens
   - Passive closing: Automatic liquidation when price falls below stop-loss line
   - Timeout closing: Expired unclosed orders can be force-liquidated by anyone
   - After closing, repay loan and return remaining funds to user

**Profit Calculation Example**:
- Opening: 100 SOL margin + 200 SOL loan = 300 SOL to buy tokens
- Token price rises 50%, position value becomes 450 SOL
- Closing: 450 SOL - 200 SOL loan - interest = approximately 240 SOL returned to user
- Net profit: 240 - 100 = 140 SOL (140% return rate)

### 3.2 Leveraged Short

**Basic Concept**:
Leveraged short refers to borrowing tokens to sell immediately, expecting to buy back at lower prices when token prices fall, profiting from the difference.

**Specific Process**:
1. **Opening Position Phase**:
   - User provides margin (e.g., 100 SOL)
   - Borrow tokens from lending pool (e.g., choose 3x leverage)
   - Immediately sell borrowed tokens to receive SOL
   - Establish short order record

2. **Position Holding Phase**:
   - User holds short position
   - Bear borrowing interest costs
   - Set stop-loss price to prevent losses from price increases
   - Monitor margin ratio

3. **Position Closing Phase**:
   - Use SOL to buy back same amount of tokens
   - Repay borrowed tokens
   - Remaining SOL returned to user

**Profit Calculation Example**:
- Opening: Borrow 1000 tokens, price at 0.1 SOL/token, sell for 100 SOL
- Token price falls to 0.07 SOL/token
- Closing: Use 70 SOL to buy back 1000 tokens to repay loan
- Net profit: 100 - 70 = 30 SOL profit

### 3.3 Leverage Multipliers and Risks

**Leverage Options**:
- 2x leverage: Conservative, suitable for stable investors
- 3x leverage: Balanced risk and return
- 5x leverage: High risk high return, suitable for professional traders

**Risk Control**:
- **Margin Requirements**: Ensure users have sufficient funds to bear potential losses
- **Stop-loss Mechanism**: Automatic liquidation prevents loss expansion
- **Forced Liquidation**: Protect lender interests when margin is insufficient
- **Expiration Liquidation**: Prevent long-term occupation of lending resources

**Lending Pool Management**:
- SOL lending pool: Provides funds for long users
- Token lending pool: Provides tokens for short users
- Dynamic interest rate adjustment: Adjust borrowing costs based on supply and demand
- Reserve fund system: Ensure platform has sufficient funds for extreme situations

## IV. Product Advantages

### 4.1 Technical Advantages

**High Performance**:
- Based on Solana blockchain, transaction confirmation time approximately 400 milliseconds
- Low gas fees, approximately 0.00025 SOL per transaction
- Support high concurrent transactions without network congestion issues

**Security and Reliability**:
- Smart contracts undergo rigorous security audits
- Multi-signature protection for critical funds
- Decentralized architecture with no single point of failure risk

**Algorithm Optimization**:
- Precise mathematical models ensure reasonable pricing
- Anti-slippage algorithms protect users from price manipulation
- Intelligent risk control system

### 4.2 User Experience

**Simple and Easy to Use**:
- Intuitive trading interface
- One-click switching between spot/leverage modes
- Real-time price and profit display

**Flexibility**:
- Support multiple leverage multiplier choices
- Freely set stop-loss prices
- Can actively close positions at any time

**Transparency**:
- All transactions verifiable on-chain
- Real-time display of liquidity and prices
- Open source code subject to community supervision

## V. Use Cases

### 5.1 Spot Trading Scenarios

- **Long-term Investment**: Investors bullish on project prospects
- **Arbitrage Trading**: Utilizing price differences between different platforms
- **Liquidity Provision**: Earning trading fee shares

### 5.2 Leveraged Trading Scenarios

**Long Scenarios**:
- Bullish on a token but with limited funds
- Aggressive investors wanting to amplify investment returns
- Hedging other short positions

**Short Scenarios**:
- Bearish on token prices
- Hedging spot holding risks
- Earning profits in bear markets

## VI. Summary

SpinPet provides users with a comprehensive, secure and reliable decentralized trading platform through innovative CurveAMM technology and leveraged trading mechanisms. Whether ordinary users hoping to conduct simple token exchanges or professional traders seeking high returns, all can find suitable trading methods on the platform.

The platform's automated market maker mechanism ensures abundant liquidity, while the intelligent risk control system protects user fund safety. Through continuous technical optimization and product iteration, SpinPet is committed to becoming the most competitive decentralized trading platform in the Solana ecosystem. 