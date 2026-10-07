# Render deployment

This package is a Node.js Web Service. Start on a **Deriv demo account**.

## 1. Add the files to a private GitHub repository

Upload the contents of this folder to a private repository. Do not upload `.env`, API tokens, generated trade logs, or measurement reports.

## 2. Create the Render Web Service

In Render, choose **New + → Web Service**, connect the repository, then set:

- **Build Command:** `npm ci`
- **Start Command:** `npm start`
- **Instance:** choose an always-on instance suitable for your use; a sleeping service cannot run a continuous bot.

## 3. Add persistent storage

The 500-trade report and trade log need to survive deploys/restarts. Attach a persistent disk mounted at `/var/data`, then set:

| Environment variable | Value |
|---|---|
| `TRADE_LOG_FILE` | `/var/data/trades.log.jsonl` |
| `MEASURE_REPORT_FILE` | `/var/data/measure_report.json` |

Without persistent storage, a restart can discard measurement progress.

## 4. Configure the environment

Set these in Render's Environment tab. Keep secrets out of GitHub and chat.

| Variable | Starting value |
|---|---|
| `DERIV_APP_ID` | Your Deriv application ID |
| `DERIV_API_TOKEN` | Your Deriv token with the required trading scope |
| `DERIV_ACCOUNT_TYPE` | `demo` |
| `ASSET` | `frxEURUSD` |
| `BOT_MODE` | `MEASURE` |
| `STRATEGY_MODE` | `ALPHA4` |
| `STAKE` | `1` |
| `DURATION_VALUE` | `30` |
| `DURATION_UNIT` | `m` |
| `PAYOUT_EDGE_BUFFER_PCT` | `0.5` |
| `NEWS_RETRY_COUNT` | `0` |
| `DASHBOARD_TOKEN` | A long, unique random value |

The other strategy, session, news, and risk defaults are documented in `.env.example`. `STRATEGY_MODE=ALPHA4` is the default frozen exhaustion/reversal strategy; `PULLBACK`, `CONFLUENCE`, and `SIMPLE` remain available for comparison. Default TRADE sizing is 0.5% of equity, martingale is off, daily loss stop is 3%, drawdown reduction begins at 5%, drawdown stop is 10%, and the minimum quoted payout is 78%.

For an existing Render service, changing package defaults does not override saved environment values: set `DURATION_VALUE=15`, `DURATION_UNIT=m`, and `NEWS_RETRY_COUNT=0` in Render, then redeploy/restart the service. Keep `NEWS_FAIL_CLOSED=true`; if both calendar sources fail, the bot should pause trading until it gets a fresh calendar.

Do not set `BOT_MODE=TRADE` before this exact strategy has completed MEASURE and you have reviewed its report. The TRADE gate also requires the measured 95% lower win-rate bound to clear each contract quote's break-even plus the configured buffer.

## 5. Verify it is running

In Render's Logs tab, look for `WebSocket authenticated` and a successful historical-candle load. Open the service URL with the dashboard token:

```text
https://<your-service>.onrender.com/?token=<your-DASHBOARD_TOKEN>
```

The dashboard shows the selected strategy, measurement progress, proposal break-even, and recent trades. Keep the service on demo while evaluating it. Passing the code's thresholds does not guarantee future profitability.