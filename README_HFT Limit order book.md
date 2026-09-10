# HFT Limit Order Book & Execution Engine

Event-driven limit order book, microstructure signal research, execution
simulation, and risk controls, built in stages so each layer is testable
before the next one depends on it.

## Status

| Phase | Module | Status |
|---|---|---|
| 1 | Market Data Engine (event schema + synthetic generator) | Done |
| 2 | Order Book Engine (price-level book, FIFO queue, matching) | Done |
| 3 | Feature Engine (spread, mid, OBI, microprice, OFI) | Done |
| 4 | Research — does OBI/OFI/microprice predict returns? | Not started |
| 5 | Strategy rules | Not started |
| 6 | Execution Simulator (queue position, partial fills, slippage) | Partial (queue position lives in the book already) |
| 7 | Risk Engine (position/loss limits, kill switch) | Not started |
| 8 | Backtesting (walk-forward, train/val/test) | Not started |
| 9 | C++ port of latency-sensitive components | Not started |
| 10 | Latency experiments | Not started |
| 11 | Dashboard | Not started |
| 12 | Docs / CV writeup | This file, will expand |

## Run it

```bash
python3 tests/test_orderbook.py        # unit tests
python3 -m src.run_pipeline --n_events 200000 --out data/features.csv
```

## Known limitations (Phase 1 stand-in)

- `synthetic_generator.py` produces a *structurally* realistic event stream
  (self-consistent order ids, Poisson-ish arrivals, mean-reverting mid) but
  it is not real market data. Swap it for a `historical_loader.py` reading
  LOBSTER/ITCH/NSE tick data later — nothing downstream changes, since both
  emit the same `MarketEvent` stream.
- The generator doesn't know when the order book internally matches a
  crossing ADD, so a later CANCEL/TRADE for an already-filled order is
  silently ignored by the book. Harmless with synthetic data; real feeds
  don't have this issue since they reflect actual exchange state.
- `queue_position()` exists in the book already (Module 8 groundwork) but
  isn't wired into a standalone execution simulator yet.

## Benchmarks

Pure Python, 200,000 events, single instrument, 5-level snapshots every 5
events: **~121,000 events/sec** on this machine. This is the baseline
Phase 9 (C++ port) will be measured against.


## Results & Engineering Findings

The project was deliberately evaluated as a research and execution
infrastructure project rather than assuming that a microstructure signal
is profitable.

### Research Findings

Initial directional strategies produced negative P&L on synthetic data.
A contrarian formulation reduced the loss but remained unprofitable after
transaction costs.

Parameter analysis showed that stronger OBI thresholds reduced losses,
but no tested configuration established a robust profitable edge.

Walk-forward validation is used to reduce the risk of relying on a single
train/test split.

### Execution Modeling

The engine models:

- Transaction costs
- Slippage
- Position limits
- Execution latency
- Queue-position effects

### Systems Engineering

A C++20 limit-order-book implementation was developed alongside the
Python research implementation.

The C++ engine is benchmarked in events/second and nanoseconds/event.

### Important Limitation

The current market data is synthetic. Therefore performance results
should not be interpreted as evidence of live trading profitability.

The next research step is replaying historical exchange-level order-book
data.
