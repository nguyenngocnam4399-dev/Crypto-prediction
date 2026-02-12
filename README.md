# Crypto Data Pipeline (with Apache Kafka)

### Data Engineering Capstone Project  
**Author:** Nguyễn Ngọc Nam  
**Mentor:** Cù Hữu Hoàng  
**Location:** Ho Chi Minh City, Vietnam — 2025  

---

## 1️⃣ Introduction
This project was developed to design a structured, scalable, and empirically testable quantitative trading research system. It standardizes the entire workflow from raw market data, technical indicators, and logical conditions (metrics), to trading decisions and performance confirmation, ensuring clear separation between processing layers to prevent data leakage. Instead of relying on opaque machine learning models, the system implements a deterministic scoring mechanism based on weighted metrics and the concept of edge, allowing transparent evaluation of directional dominance between buyers and sellers. From a technical perspective, the Data Warehouse architecture follows a fact-driven design with clearly defined grain, idempotent ETL processes, and full traceability of the signal lifecycle. Beyond signal generation, the platform also mines recurring market structures using FP-Growth to assess the sustainability of edge. Overall, this framework prioritizes transparency, scalability, and experimental validation over short-term optimization or overfitting.

---

## 2️⃣ Overview
This project implements an end-to-end quantitative trading research pipeline built on a layered Data Warehouse architecture. It transforms real-time crypto market data into atomic indicators, configurable trading metrics, and deterministic buy/sell signals through a weighted scoring engine. Signals are independently validated via adaptive backtesting and further analyzed using structural pattern mining (FP-Growth) to assess edge sustainability. The system prioritizes transparency, reproducibility, and extensible research design.  

---

## 3️⃣ System Architecture

![System Architecture](images/System_Architecture.png)

The system follows a layered architecture that clearly separates data ingestion, transformation, signal generation, validation, and analytics. Each layer has a well-defined responsibility to ensure scalability, traceability, and experimental control.

🔹 1. Data Ingestion Layer

Real-time crypto market data is streamed via Kafka.

Spark Streaming processes incoming OHLCV data.

Cleaned data is stored in fact_kline within the Data Warehouse.

This layer ensures immutable, time-series market truth.

🔹 2. Indicator Computation Layer

Atomic technical indicators (RSI, MACD, EMA, ADX, BB, VWAP, etc.) are computed using Spark.

Each indicator is stored independently in fact_indicator.

Indicators are partitioned by symbol and interval to maintain clear grain definition.

This guarantees transparency and recomputability.

🔹 3. Metric Abstraction Layer

Indicators are transformed into logical trading conditions via configurable metrics (dim_metric).

Metrics are evaluated and stored in fact_metric_value.

Logic includes threshold checks, trend direction, cross detection, and volatility filters.

This enables strategy tuning without code modification.

🔹 4. Prediction Engine

BUY and SELL scores are computed independently using weighted metrics.

Edge and confidence are calculated to measure directional dominance.

Signals are stored in fact_prediction.

This layer remains deterministic and fully explainable.

🔹 5. Backtesting & Confirmation Engine

Predictions are evaluated using adaptive TP/SL logic within a future lookahead window.

Results are stored in fact_prediction_result.

Confirmation is separated from prediction to avoid leakage.

This ensures realistic performance validation.

🔹 6. Analytics & Pattern Mining

Performance metrics (equity curve, rolling stability, expectancy) are derived from warehouse facts.

FP-Growth mining extracts recurring structural win patterns.

Flask API exposes analytics endpoints for visualization.

This layer supports experimental research and structural edge validation.

---

## 4️⃣ Data Warehouse Design
Data Warehouse Schema
![Warehouse ERD](images/warehouse_schema.png)
The warehouse follows a fact-driven design where market data, indicators, metrics, predictions, and confirmation results are stored as separate fact tables with clearly defined grain. Dimension tables provide normalization for symbols, intervals, indicators, and metrics.

---

## 🔹 Core Dimensions

- **dim_symbol** – tradable assets  
- **dim_interval** – timeframe definition  
- **dim_indicator_type** – atomic indicator registry  
- **dim_metric** – configurable trading logic  

---

## 🔹 Fact Layers

| Table                     | Grain                                      | Purpose                          |
|---------------------------|--------------------------------------------|----------------------------------|
| `fact_kline`              | (symbol, interval, close_time)             | Raw market data                 |
| `fact_indicator`          | (symbol, interval, indicator, timestamp)   | Atomic indicator values         |
| `fact_metric_value`       | (symbol, interval, metric, calculating_at) | Evaluated trading conditions    |
| `fact_prediction`         | (symbol, interval, predicting_at)          | Deterministic trading decisions |
| `fact_prediction_result`  | (prediction_id)                            | Realized trade outcomes         |

---

### Design Principles

- Clear grain definition  
- No cross-layer coupling  
- Idempotent ETL  
- Independent prediction and confirmation  
- Fully traceable signal lifecycle

---

# 5️⃣ Indicator Engineering

The indicator layer transforms raw market data into atomic, recomputable technical signals. Each indicator is calculated independently using Spark and stored in `fact_indicator` with a clearly defined grain:

(symbol_id, interval_id, indicator_type, timestamp)

Supported indicators include RSI, MACD, EMA, ADX, Bollinger Bands, ATR, VWAP deviation, and volume-based metrics.

---

## Design Principles

- **Atomic Storage** – Each row represents a single indicator value at a single timestamp.  
- **No Composite Features** – Indicators are not mixed or pre-aggregated.  
- **Recomputable** – All indicators can be regenerated from `fact_kline`.  
- **Window-Based Computation** – Rolling and exponential calculations use Spark window functions.  
- **Partitioned Processing** – Computation is distributed by symbol and interval for scalability.  
- **Idempotent Writes** – Unique constraints prevent duplicate calculations.

---

This design ensures transparency, avoids hidden transformations, and allows clean separation between raw technical signals and higher-level trading logic.

---

# 6️⃣ Metric Abstraction Layer

The metric layer converts continuous indicator values into structured, configurable trading conditions. Instead of hardcoding strategy rules in application logic, all trading conditions are defined in `dim_metric` and evaluated dynamically into `fact_metric_value`.

Each metric specifies:

- Anchor indicator (`indicator_type_id`)
- Threshold range (`threshold_start`, `threshold_end`)
- Direction logic (ABOVE, BELOW, BETWEEN, TREND_UP, CROSS_UP, etc.)
- Window size and unit
- Weight
- Active flag

---

## Purpose

- Separate technical signals from trading logic  
- Enable configuration-driven strategy design  
- Allow historical re-evaluation without code changes  
- Support metric-level experimentation  

---

## Data Flow

fact_indicator → metric evaluation → fact_metric_value

Each metric is evaluated per symbol, interval, and timestamp, producing structured conditions (typically binary or weighted values) that are later aggregated in the prediction engine.

---

## Design Principles

- **Config-Driven Logic** – Strategy rules live in the database, not in code  
- **Independent Evaluation** – Metrics are computed separately from scoring  
- **Reproducible** – Historical metric states remain traceable  
- **Extensible** – New trading conditions can be added without refactoring  

---

This abstraction layer provides the structural foundation for deterministic scoring and controlled strategy experimentation.

---

# 7️⃣ Prediction Engine

The prediction engine aggregates evaluated metrics into deterministic trading decisions using a weighted scoring model. BUY and SELL signals are computed independently to avoid directional contamination and ensure structural clarity.

---

## Scoring Logic

buy_score  = Σ(weighted BUY metrics)  
sell_score = Σ(weighted SELL metrics)

edge = |buy_score − sell_score|  
confidence = max(buy_score, sell_score) / MAX_SCORE  

---

## Decision Rules

- **BUY** → buy_score ≥ threshold and buy_score > sell_score  
- **SELL** → sell_score ≥ threshold and sell_score > buy_score  
- **SIDEWAY** → otherwise  

Additional safeguards include:

- Conflict detection (simultaneous strong BUY & SELL conditions)  
- No-trade filters  
- Minimum edge requirement  
- Confidence band filtering  

---

## Output

Signals are stored in `fact_prediction`, including:

- action  
- market_score (edge)  
- confidence_score  
- structural metric breakdown  

---

## Design Principles

- **Deterministic & Explainable** – No black-box models  
- **Edge-Based Separation** – Clear directional dominance measurement  
- **Metric Transparency** – Full traceability of contributing conditions  
- **Decoupled from Backtesting** – Prediction and confirmation are independent  

---

This layer produces structured trading hypotheses while preserving full auditability and experimental control.

---

# 8️⃣ Backtesting & Confirmation Framework

The confirmation framework evaluates trading predictions independently using an adaptive take-profit / stop-loss mechanism within a controlled lookahead window. Prediction and confirmation are strictly decoupled to prevent data leakage.

---

## Confirmation Logic

For each prediction:

1. Define a future lookahead window (e.g., N hours)
2. Compute:
   - max_return
   - min_return
3. Apply adaptive TP / SL logic
4. Confirm only if TP or SL is hit

BUY logic:
- WIN → price reaches take-profit
- LOSS → price hits stop-loss

SELL logic:
- WIN → price drops to take-profit
- LOSS → price rises to stop-loss

If neither condition is met within the lookahead window, the signal may remain unconfirmed.

---

## Adaptive Risk Model

- Strong edge → wider TP / longer lookahead  
- Weak edge → tighter SL / shorter window  
- No double confirmation  
- Idempotent write to result table  

---

## Output

Results are stored in `fact_prediction_result`, including:

- pnl_pct  
- exit_price  
- result_status  
- confirmed_at  

---

## Design Principles

- **Leakage Prevention** – Prediction and validation separated  
- **Controlled Lookahead** – No future information leakage  
- **Adaptive Risk Management** – Reward scales with conviction  
- **Reproducible Backtesting** – Fully warehouse-driven  

---

This framework transforms trading hypotheses into validated performance records while preserving realism and experimental integrity.

---

# 9️⃣ Analytics & Performance Evaluation

The analytics layer evaluates confirmed trades using structured performance metrics derived directly from warehouse facts. All evaluations are reproducible and based on `fact_prediction` and `fact_prediction_result`.

---

## Core Metrics

- **Win Rate** – Ratio of winning trades  
- **Average PnL** – Mean percentage return per trade  
- **Expectancy**  
  E = WinRate × AvgWin − (1 − WinRate) × AvgLoss  
- **Rolling Stability** – 5 / 10 / 20 trade rolling win rates  
- **Equity Curve** – Cumulative capital growth simulation  
- **Drawdown** – Peak-to-trough capital decline  

---

## Regime Analysis

Performance is analyzed under different structural regimes:

- Trend (TREND_UP / TREND_DOWN)  
- Momentum (POS / NEG)  
- Volatility (HIGH / LOW)  
- Edge strength (STRONG / WEAK)  

This helps assess conditional performance and robustness.

---

## Structural Pattern Mining

FP-Growth is applied to confirmed WIN trades to extract:

- Frequent structural conditions  
- Association rules  
- Confidence  
- Lift (edge strength contribution)  

This validates recurring market structures that support sustained edge.

---

## Design Principles

- Fully warehouse-driven analytics  
- No hidden transformations  
- Clear separation between signal generation and evaluation  
- Structural validation beyond simple win rate  

---

This layer transforms confirmed trade results into measurable insights, enabling systematic strategy evaluation and structural edge validation.

---

# 🔟 Tech Stack & Engineering Practices

## Tech Stack

- **Python** – Core language  
- **PySpark** – Distributed computation (batch & streaming)  
- **Spark ML (FPGrowth)** – Structural pattern mining  
- **Kafka** – Real-time market data ingestion  
- **MySQL 8** – Data Warehouse storage  
- **Flask** – Analytics API layer  
- **NumPy / Pandas / Scikit-learn** – Statistical evaluation  

---

## Engineering Practices

- **Layered Architecture** – Clear separation between ingestion, transformation, prediction, and validation  
- **Fact-Driven Warehouse Design** – Explicit grain definition for each table  
- **Idempotent ETL** – Anti-join writes and unique constraints prevent duplication  
- **UTC Normalization** – Consistent time handling across pipeline  
- **Window-Based Computation** – Efficient rolling calculations in Spark  
- **Config-Driven Strategy Logic** – Metrics defined in database, not hardcoded  
- **Prediction / Confirmation Separation** – Strict leakage prevention  
- **Traceable Signal Lifecycle** – Full auditability from raw data to confirmed result  

---

This stack and engineering approach ensure scalability, reproducibility, and experimental control, making the system suitable for quantitative research and production-oriented data workflows.

