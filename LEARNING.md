# Prediction Markets

This challenge implements a binary prediction market backed by ETH. The market creates YES and NO ERC-20 outcome tokens, keeps inventory in the market contract, prices trades from the relative quantity of each outcome already sold, and pays the winning token at a fixed collateral value after an oracle reports the result.

## Market lifecycle

1. The liquidity provider deploys the market with ETH collateral, an oracle, an initial YES probability, and a percentage of outcome tokens to lock.
2. The contract mints equal total supplies of YES and NO. A probability-weighted portion is assigned to the liquidity provider while the remaining inventory stays in the contract for trading.
3. Users buy outcome tokens at the average of the probability before and after their trade. ETH paid for trades is recorded as LP revenue.
4. Users can sell tokens back after granting the market an allowance. The inverse inventory change determines their ETH return.
5. The oracle reports YES or NO exactly once. Trading and liquidity changes then stop.
6. Winning-token holders redeem at the initial fixed token value. The liquidity provider resolves its remaining winning inventory and withdraws accrued trading revenue.

## Core accounting

The market derives probability from sold supply rather than reserve ratios directly:

```text
tokensSold = totalSupply - tokenReserve
probability = tokensSold / (yesSold + noSold)
tradePrice = tokenValue × averageProbability × tokenAmount
```

Using the average probability before and after the trade charges for the whole movement along the price curve. The calculation uses 18-decimal fixed-point arithmetic throughout and divides only at the end to preserve precision.

ETH is separated into collateral and LP trading revenue. Collateral backs winning-token redemption at `initialTokenValue`; trade payments fund subsequent sells and become LP revenue after resolution.

## Safety properties

- Initial probability and the locked percentage must both be between 1 and 99.
- The market rejects zero-value trades, incorrect buy payments, insufficient reserves, balances, and allowances.
- The liquidity provider cannot trade or redeem user outcome tokens.
- Only the configured oracle can report, and reporting can happen only once.
- Liquidity changes and trading are disabled once the result is reported.
- Winning tokens are burned before ETH is paid, and accounting is updated before external calls.

## Validation

The official Hardhat suite passes all 42 tests across deployment, liquidity, oracle reporting, resolution, pricing, buying, selling, and redemption. Solidity compilation, frontend type checking, and the Next.js production build also pass.

## Production considerations

This implementation is intentionally educational. A production market would need decentralized and dispute-aware resolution, explicit fee policy, stronger liquidity mathematics, bounded rounding analysis, reentrancy protection, emergency controls, and formal economic testing against manipulation and insolvency.
