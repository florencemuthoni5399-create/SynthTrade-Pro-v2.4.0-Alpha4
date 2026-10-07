# SynthTrade Pro v2.4.0 — Alpha #4

Node.js 18+ headless bot for Deriv `frxEURUSD` Rise/Fall contracts. **Alpha #4 is now the default strategy.**

## Default strategy

### Alpha 4-C — CALL / Rise
Uses completed 5-minute candles:

- 3-bar downward move: 2.0–3.5 ATR
- RSI(14) < 30
- lower rejection wick >= 70% of candle range
- close-location >= 45%
- 30-minute contract

### Alpha 4-P — PUT / Fall

- 3-bar upward move: 2.0–3.5 ATR
- RSI(14) > 70
- upper rejection wick >= 65% of candle range
- close-location >= 45%
- ATR14 in the high-volatility regime (>= 67th percentile of the recent 200 ATR observations)
- 30-minute contract

Signals are calculated only from closed 5-minute candles. When a qualifying candle closes, the bot waits for the first tick of the next 5-minute candle and requests the contract quote, matching the research entry convention.

## Money management

Default TRADE-mode controls:

- **0.5% of current equity per trade**
- no martingale
- 3% daily loss stop
- 5% peak-to-current drawdown: automatically halve the risk percentage
- 10% peak-to-current drawdown: stop opening new trades and require manual review
- no daily profit cap by default
- minimum quoted payout: 78%
- existing quote-aware statistical payout gate remains enabled

MEASURE mode deliberately remains a flat `STAKE=1` experiment and ignores fractional-risk sizing. This keeps measurement comparable and prevents money management from contaminating the strategy test.

## MEASURE / TRADE

Keep:

```text
DERIV_ACCOUNT_TYPE=demo
BOT_MODE=MEASURE
STRATEGY_MODE=ALPHA4
DURATION_VALUE=30
DURATION_UNIT=m
```

A fresh measurement report is required after this strategy change. Old reports are rejected by the strategy fingerprint.

Do not move to `BOT_MODE=TRADE` merely because a report passes its gate. Review the exact 500-trade report first.

## Existing protections

- EUR/USD: `frxEURUSD`
- 5-minute completed candles
- UTC session gate: 08:00–17:00
- EUR/USD medium/high news blackout: ±30 minutes
- fail-closed news calendar with fallback
- execution ping floor
- quote-before-buy
- quote break-even calculation
- 95% Wilson lower-bound payout gate
- persistent measurement/trade logs supported

## Render

Set the environment variables from `.env.example`. In particular:

- `STRATEGY_MODE=ALPHA4`
- `DURATION_VALUE=30`
- `DURATION_UNIT=m`
- `BOT_MODE=MEASURE`
- `DERIV_ACCOUNT_TYPE=demo`
- `MARTINGALE_ENABLED=false`
- `RISK_PER_TRADE_PCT=0.5`
- `MAX_DAILY_LOSS_PCT=3`
- `DRAWDOWN_REDUCE_AT_PCT=5`
- `DRAWDOWN_STOP_AT_PCT=10`
- `MIN_PAYOUT_PCT=78`
- `NEWS_RETRY_COUNT=0`

For persistent Render storage:

```text
TRADE_LOG_FILE=/var/data/trades.log.jsonl
MEASURE_REPORT_FILE=/var/data/measure_report.json
```

Keep secrets such as `DERIV_API_TOKEN` and `DASHBOARD_TOKEN` in Render's environment settings, never in the repository.

## Important

The Alpha #4 historical research showed a promising research edge but also a CALL weakness in 2026. The bot therefore freezes the researched rules rather than trying to repair that period through further curve fitting. Live profitability is not guaranteed.
