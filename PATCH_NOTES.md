# v2.4.0 — Alpha #4 default + conservative money management

- Made the frozen Alpha #4 exhaustion/reversal strategy the default strategy (`STRATEGY_MODE=ALPHA4`).
- CALL branch:
  - 2–3.5 ATR downward exhaustion over 3 completed 5m candles
  - RSI(14) < 30
  - lower wick >= 70% of signal candle range
  - close-location >= 45%
  - 30-minute Rise/Fall contract
- PUT branch:
  - 2–3.5 ATR upward exhaustion over 3 completed 5m candles
  - RSI(14) > 70
  - upper wick >= 65%
  - close-location >= 45%
  - ATR14 percentile >= 67% over the recent 200 ATR observations
  - 30-minute Rise/Fall contract
- Signals are evaluated from completed 5m candles and submitted at the first tick of the next 5m candle, matching the research entry convention.
- Disabled martingale by default.
- TRADE-mode position sizing defaults to 0.5% of current equity.
- At 5% peak-to-current drawdown, risk is automatically reduced by 50%.
- At 10% peak-to-current drawdown, new trading stops and requires manual review.
- Daily loss stop remains 3% of day-start balance.
- Daily profit target is disabled by default so profitable days are not artificially capped.
- Added a minimum quoted payout gate of 78% in TRADE mode, in addition to the existing 95% Wilson lower-bound / quote break-even gate.
- Existing session/news/ping protections remain enabled.
- Existing PULLBACK, CONFLUENCE, and SIMPLE strategies remain available for comparison.
- Existing measurement reports are fingerprinted and will not unlock TRADE for this changed strategy; a fresh MEASURE run is required.

This package does not claim Alpha #4 is profitable live. Keep `BOT_MODE=MEASURE` and the Deriv account on demo while validating the exact deployed build.
