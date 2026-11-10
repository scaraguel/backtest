# Files

- [Broker, Order Execution, and Accounting](broker-execution-and-accounting.md) - Architecture of the Broker facade and its collaborators, covering order submission, reservation, deterministic eligibility and fill ordering, execution costs, position and cash mutation, rejection, and portfolio capture.
- [Event-Driven Runtime and Strategy Lifecycle](event-driven-runtime.md) - Explains how Engine owns and executes a single backtest run, including causal callback boundaries, per-bar ordering, pre-open targets, session decisions, failure cleanup, and result generation.
- [Risk Controls, Stops, and Rebalancing](risk-controls-and-rebalancing.md) - How position exits, portfolio limits, broker controls, target-weight sizing, and schedules interact with normal order execution while retaining the cause of forced actions.
- [Causal Runtime Services and Scheduling](runtime-services.md) - Run-scoped causal data views, history windows, publication and eligibility masking, and calendar-aware callback scheduling for backtests.
