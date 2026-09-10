# HFT Limit Order Book & Execution Engine

Event-driven limit order book, microstructure signal research, execution
simulation, and risk controls, built in stages so each layer is testable
before the next one depends on it.

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
