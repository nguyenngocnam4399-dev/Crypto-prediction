# 🚀 Deterministic Quantitative Trading Research Platform  
### Production-Grade Crypto Data Pipeline with Structured Edge Validation  

### Data Engineering Capstone Project  
**Author:** Nguyễn Ngọc Nam  
**Mentor:** Cù Hữu Hoàng  
**Location:** Ho Chi Minh City, Vietnam — 2025  

---

# 1️⃣ Introduction

This project designs a structured, scalable, and empirically testable quantitative trading research framework. It standardizes the full lifecycle of a trading signal—from real-time market ingestion, technical indicator computation, and metric abstraction, to deterministic prediction and independent confirmation—while strictly separating processing layers to prevent data leakage.

Instead of relying on opaque machine learning models, the system implements a weighted, metric-driven scoring engine based on the concept of **edge**, allowing transparent evaluation of directional dominance between buyers and sellers. The architecture follows a fact-driven Data Warehouse design with explicit grain definition, idempotent ETL processes, and full signal traceability. Structural pattern mining (FP-Growth) is applied to validated trades to assess edge sustainability. The framework prioritizes transparency, reproducibility, and experimental rigor over short-term optimization.

---

# 2️⃣ Overview

The platform implements an end-to-end quantitative research pipeline orchestrated by **Apache Airflow** and built on a layered Data Warehouse architecture. Real-time crypto market data is transformed into atomic indicators, configurable trading metrics, and deterministic buy/sell signals. Signals are validated via adaptive backtesting and analyzed using structural pattern mining to measure performance robustness and recurring market structures.

---

# 3️⃣ System Architecture

![System Architecture](images/System_Architecture.png)

The system follows a layered architecture separating ingestion, transformation, signal modeling, orchestration, validation, and analytics to ensure scalability, reproducibility, and controlled experimentation.

---

## 1. Data Ingestion

- Kafka streams real-time OHLCV market data  
- Spark Streaming processes and normalizes records  
- Clean data stored in `fact_kline`  

Establishes immutable time-series market truth.

---

## 2. Indicator Computation

- Atomic indicators (RSI, MACD, EMA, ADX, BB, VWAP, ATR, etc.) computed via Spark  
- Stored independently in `fact_indicator`  
- Partitioned by symbol and interval  

Ensures transparency and recomputability.

---

## 3. Metric Abstraction

- Logical trading conditions defined in `dim_metric`  
- Evaluated results stored in `fact_metric_value`  
- Supports threshold, trend, cross, and volatility logic  

Enables configuration-driven strategy design.

---

## 4. Prediction Engine

- Independent BUY and SELL scoring  
- Edge and confidence calculation  
- Results stored in `fact_prediction`  

Deterministic, explainable, and auditable.

---

## 5. Backtesting & Confirmation

- Adaptive TP/SL within controlled lookahead window  
- Results written to `fact_prediction_result`  
- Strict separation from prediction layer  

Prevents leakage and ensures realistic evaluation.

---

## 6. Orchestration Layer (Apache Airflow)

The entire workflow is coordinated using **Apache Airflow DAGs**.

- Dependency-controlled execution of Spark jobs  
- Scheduled metric computation, prediction, and confirmation  
- Modular DAG structure per symbol / pipeline stage  
- Retry and failure handling  
- Containerized deployment (Docker Compose)  
- CeleryExecutor for distributed task execution  

Airflow ensures reproducible, production-ready orchestration across ingestion, transformation, scoring, validation, and analytics layers.

---

## 7. Analytics & Pattern Mining

- Equity curve, rolling stability, expectancy metrics  
- Regime-based performance analysis  
- FP-Growth mining of structural win patterns  
- Flask API for visualization  

Supports systematic edge validation.

---

# 4️⃣ Data Warehouse Design

![Warehouse ERD](images/warehouse_schema.png)

The warehouse follows a fact-driven layered model. Market data, indicators, metrics, predictions, and confirmation results are stored in separate fact tables with explicitly defined grain to ensure traceability and reproducibility.

---

## Core Dimensions

- `dim_symbol` – tradable assets  
- `dim_interval` – timeframe registry  
- `dim_indicator_type` – atomic indicator definitions  
- `dim_metric` – configurable trading logic  

---

## Fact Layers

| Table                    | Grain                                      | Role |
|--------------------------|--------------------------------------------|------|
| `fact_kline`             | (symbol, interval, close_time)             | Market truth |
| `fact_indicator`         | (symbol, interval, indicator, timestamp)   | Atomic signals |
| `fact_metric_value`      | (symbol, interval, metric, calculating_at) | Logical conditions |
| `fact_prediction`        | (symbol, interval, predicting_at)          | Trading hypothesis |
| `fact_prediction_result` | (prediction_id)                            | Realized outcome |

---

### Design Principles

- Explicit grain definition  
- Idempotent ETL  
- Layered separation of concerns  
- No signal-result coupling  
- Fully traceable signal lifecycle  

---

# 5️⃣ Indicator Engineering

Indicators transform raw market data into atomic, recomputable technical states stored in:

(symbol_id, interval_id, indicator_type, timestamp)

Supported indicators include RSI, MACD, EMA, ADX, Bollinger Bands, ATR, VWAP deviation, and volume-based metrics.

### Principles

- Atomic storage  
- No composite pre-aggregation  
- Spark window-based computation  
- Partitioned distributed processing  
- Fully recomputable from `fact_kline`  

---

# 6️⃣ Metric Abstraction Layer

Metrics convert continuous indicator values into structured trading conditions defined in `dim_metric`.

Each metric specifies:

- Anchor indicator  
- Threshold range  
- Direction logic  
- Window configuration  
- Weight  
- Active status  

Evaluated results are stored in `fact_metric_value`.

### Purpose

- Config-driven strategy logic  
- Independent evaluation from scoring  
- Historical reproducibility  
- Strategy experimentation without refactoring  

---

# 7️⃣ Prediction Engine

buy_score  = Σ(weighted BUY metrics)  
sell_score = Σ(weighted SELL metrics)  

edge = |buy_score − sell_score|  
confidence = max(score) / MAX_SCORE  

### Decision Rules

- BUY → dominant and above threshold  
- SELL → dominant and above threshold  
- SIDEWAY → otherwise  

Safeguards:

- Conflict detection  
- Edge gating  
- Confidence filtering  

Stored in `fact_prediction`.

---

# 8️⃣ Backtesting & Confirmation Framework

Predictions are validated independently using adaptive TP/SL within a controlled lookahead window.

### Process

1. Define future window  
2. Compute max/min return  
3. Apply adaptive TP/SL  
4. Confirm only if triggered  

Results stored in `fact_prediction_result`.

### Principles

- Leakage prevention  
- Controlled lookahead  
- Edge-scaled risk  
- Idempotent result writing  

---

# 9️⃣ Analytics & Performance Evaluation

Performance evaluation is fully warehouse-driven.

### Core Metrics

- Win Rate  
- Average PnL  
- Expectancy  
- Rolling stability  
- Equity curve  
- Drawdown  

### Regime Analysis

Performance segmented by trend, momentum, volatility, and edge strength.

### Structural Pattern Mining

FP-Growth extracts frequent structural conditions, association rules, confidence, and lift to validate sustained edge.

---

# 🔟 Tech Stack & Engineering Practices

## Tech Stack

- Python  
- PySpark (batch & streaming)  
- Apache Kafka  
- Apache Airflow (CeleryExecutor)  
- Spark ML (FPGrowth)  
- MySQL 8  
- Flask  
- NumPy / Pandas / Scikit-learn  

---

## Engineering Practices

- Layered architecture  
- Fact-driven warehouse modeling  
- Explicit grain control  
- Idempotent ETL & anti-join writes  
- UTC normalization  
- Window-based distributed computation  
- Config-driven strategy logic  
- Strict prediction/confirmation separation  
- DAG-based workflow orchestration  
- Containerized deployment  

---

# 🧠 Skills Demonstrated

## Data Engineering

- End-to-end streaming + batch pipeline design  
- Apache Airflow DAG orchestration  
- Distributed Spark computation  
- Fact/dimension warehouse modeling  
- Idempotent ETL architecture  
- Time-series data management  

## Quantitative Modeling

- Deterministic weighted scoring  
- Edge-based signal separation  
- Conflict detection logic  
- Adaptive risk modeling  
- Lookahead leakage prevention  

## Research & Validation

- Expectancy and rolling stability analysis  
- Equity curve and drawdown evaluation  
- Structural edge mining with FP-Growth  
- Fully reproducible warehouse-driven experimentation  

---

# 📈 Value Generated

- Converts raw market data into structured trading intelligence  
- Enables deterministic and explainable signal modeling  
- Validates trading hypotheses with controlled confirmation  
- Identifies recurring structural market patterns  
- Establishes a scalable, orchestration-ready quantitative research framework  
- Provides foundation for future ML, portfolio modeling, or production deployment  
