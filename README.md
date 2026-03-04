# LSE Trading Agent for OpenClaw

FTSE 350 trading analysis agent for the London Stock Exchange. Combines technical analysis, LLM-powered news sentiment, and risk management to generate trade signals.

## What it does

- Screens the full FTSE 350 for opportunities based on a composite signal (trend, momentum, volatility, volume)
- Deep-dives any LSE ticker with all major indicators: Bollinger Bands, RSI, MACD, 50/200-day EMA crossovers, ATR, VWAP, OBV
- Fetches recent news headlines for LLM-powered sentiment analysis
- Backtests strategies against historical data with realistic UK transaction costs (0.5% SDRT, slippage)
- Enforces risk rules: half-Kelly position sizing, ATR-based trailing stops, 15% drawdown circuit breaker, sector exposure limits
- Tracks paper portfolio with live prices from Yahoo Finance

## Install

```bash
clawhub install lse-trading-agent
```

Or clone and install locally:

```bash
git clone https://github.com/ankit-aglawe/openclaw-lse-trading-agent.git
cd openclaw-lse-trading-agent
# Copy to your OpenClaw skills directory
cp -r . ~/.openclaw/skills/lse-trading-agent/
```

## Commands

| Command | What it does |
|---------|-------------|
| `/lse-scan` | Screen FTSE 350 for trading opportunities |
| `/lse-analyze HSBA.L` | Full technical + sentiment analysis on a ticker |
| `/lse-sentiment BP.L` | News headlines for LLM sentiment analysis |
| `/lse-backtest VOD.L --years 5` | Backtest the strategy on historical data |
| `/lse-portfolio` | View current positions and P&L |
| `/lse-risk` | Check risk metrics, exposure, drawdown |

## Scripts

All scripts output JSON and can be run directly:

```bash
# Scan FTSE 350
uv run scripts/screener.py --top 20
uv run scripts/screener.py --sector "Financials" --top 10

# Analyse a ticker
uv run scripts/indicators.py HSBA.L --period 1y

# Get news headlines
uv run scripts/sentiment.py HSBA.L

# Backtest a strategy
uv run scripts/backtest.py HSBA.L --years 5 --initial-capital 10000

# Check risk on a trade
uv run scripts/risk.py --action BUY --ticker HSBA.L --price 678.5 --portfolio-value 50000

# Manage portfolio
uv run scripts/portfolio.py --init 50000
uv run scripts/portfolio.py --add HSBA.L 100 678.5
uv run scripts/portfolio.py --show
```

## Signal logic

The composite score combines five inputs:

| Signal | Weight | Source |
|--------|--------|--------|
| Trend | 25% | EMA 50/200 crossovers |
| Momentum | 25% | RSI (14) + MACD histogram |
| Volatility | 15% | Bollinger Band position |
| Volume | 15% | OBV trend + VWAP position |
| Sentiment | 20% | LLM analysis of news headlines |

Score ranges from -1.0 (strong sell) to +1.0 (strong buy). Only recommends trades with |score| > 0.4.

## Risk management

These rules are enforced on every trade and cannot be overridden:

- Max 2% of portfolio at risk per trade
- Position sized via half-Kelly criterion, capped at 5% of portfolio
- ATR-based trailing stop-loss (ATR x 2.0)
- 15% portfolio drawdown triggers circuit breaker (halts all new trades)
- 3% daily loss limit
- Max 25% exposure in any single GICS sector
- 0.5% SDRT accounted for on all UK equity purchases

## Requirements

- OpenClaw
- Python 3.12+ (via `uv`)

## Data sources

- **Yahoo Finance** (via yfinance) for OHLCV data, fundamentals, news headlines
- **LLM** (your configured provider) for sentiment analysis and signal synthesis

## Disclaimer

This is for educational and research purposes only. Not financial advice. Past performance does not guarantee future results. Always do your own research.

## License

MIT
