# 🚀 Nền Tảng Phân Tích & Nghiên Cứu Định Lượng Crypto (Production-Grade)

> Hệ thống Data Engineering thời gian thực & nền tảng nghiên cứu giao dịch định lượng  
> Tài sản trọng tâm: BTC • ETH • BNB

---

## 📌 Tóm Tắt Điều Hành (Executive Summary)

Đây là một nền tảng xử lý dữ liệu và nghiên cứu định lượng được thiết kế theo định hướng production, với mục tiêu:

- Thu thập dữ liệu thị trường crypto theo thời gian thực
- Chuẩn hóa và tổ chức dữ liệu theo mô hình Data Warehouse (Dim-Fact)
- Xây dựng hệ thống chấm điểm (scoring) tín hiệu giao dịch có tính xác định (deterministic)
- Backtest và đánh giá độ ổn định chiến lược
- Khai phá pattern giao dịch thắng bằng FP-Growth
- Trình diễn phân tích qua Dashboard

Hệ thống được xây dựng với tư duy:

- Scalability (khả năng mở rộng)
- Idempotency (chạy lại không trùng lặp)
- Fault tolerance (chịu lỗi)
- Traceability (truy vết dữ liệu)

---

# 1️⃣ Bối Cảnh & Động Lực Xây Dựng

Tài sản số đang dần trở thành một lớp tài sản được quản lý và thể chế hóa.

Thách thức chính:

- Biến động mạnh
- Nhiễu tín hiệu cao
- Nhà đầu tư nhỏ lẻ thiếu công cụ định lượng
- Quyết định giao dịch dựa trên cảm tính

Dự án này được xây dựng nhằm:

- Chuyển đổi dữ liệu thô thành insight định lượng
- Tạo hệ thống scoring minh bạch
- Kiểm chứng chiến lược bằng thống kê
- Hỗ trợ quyết định dựa trên dữ liệu

---

# 2️⃣ Kiến Trúc Tổng Thể Hệ Thống

## 🏗 System Architecture

![System Architecture](images/System_Architecture.png)

Hệ thống gồm 5 tầng:

### 🔹 Tầng Thu Thập Dữ Liệu
- Binance WebSocket / API
- News Crawler
- Kafka Streaming

### 🔹 Tầng Xử Lý
- Spark (Batch & Streaming)
- Indicator Engine
- Metric & Scoring Engine
- Backtest Engine
- FP-Growth Mining

### 🔹 Tầng Lưu Trữ
- MySQL Data Warehouse (Dim-Fact)

### 🔹 Tầng Điều Phối
- Airflow DAG
- Retry & Failure Handling
- Idempotent Job Execution

### 🔹 Tầng Trình Diễn
- Flask API
- Dashboard phân tích

---

# 3️⃣ Thiết Kế Data Engineering

## 🔄 Ingestion Real-Time

- Kafka giúp tách biệt producer & consumer
- Hỗ trợ replay dữ liệu
- Có thể scale ngang khi volume tăng

## ⚡ Xử Lý Phân Tán (Spark)

Spark được dùng để:

- Tính indicator (RSI, MACD, EMA, BB, ADX, VWAP, ATR, OBV)
- Tính metric giao dịch
- Xây dựng market score
- Xác nhận backtest
- Chuẩn bị dữ liệu mining

Thiết kế đảm bảo:

- Partition theo symbol
- Anti-duplicate write
- Idempotent JDBC insert
- Tách biệt xử lý theo coin

---

# 4️⃣ Kiến Trúc Data Warehouse

## 🗄 Mô Hình Dim-Fact

![Warehouse Schema](images/warehouse_schema_crypto.png)

![News Warehouse Schema](images/warehouse_schema_news.png)

### Dimension Tables
- dim_symbol
- dim_interval
- dim_indicator_type
- dim_metric
- tag_dim

### Fact Tables
- fact_kline
- fact_indicator
- fact_metric_value
- fact_prediction
- fact_prediction_result
- news_sentiment_weighted_fact
- fp_growth_win_patterns
- fp_growth_win_rules

---

## Vì Sao Chọn Dim-Fact?

- Tách biệt context và event
- Tối ưu truy vấn phân tích
- Lưu trữ lịch sử rõ ràng
- Dễ mở rộng metric mới
- Phù hợp chuẩn Data Warehouse

---

# 5️⃣ Framework Modeling & Scoring

## 🧮 Market Scoring

Market Score =  
Trend + Momentum + Volume + Volatility

Confidence Score =  
Market Score / Max Score

### Cơ Chế Bảo Vệ (Guard)

- Conflict Detection
- Weak Edge Filter
- Confidence Band Filter
- No-Trade Logic

Mục tiêu:

- Tránh overtrading
- Hạn chế false signal
- Giảm nhiễu trong regime squeeze

---

# 6️⃣ Backtest & Quản Trị Rủi Ro

Backtest đánh giá:

- TP / SL động
- Lookahead window
- Win/Loss classification
- PnL normalization
- Rolling expectancy
- Phân tích theo regime

Đảm bảo:

- Kiểm tra tính sống sót (survivability)
- Phát hiện suy giảm edge
- Tránh overfitting

---

# 7️⃣ Phân Tích Hiệu Suất

## 📈 Equity Curve & Drawdown

![Equity Curve](images/equity_curve.png)

- Tăng trưởng vốn
- Maximum drawdown
- Đánh giá rủi ro

---

## 📉 Rolling Expectancy

![Rolling Expectancy](images/rolling_expectancy.png)

- Kiểm tra edge theo thời gian
- Phát hiện giai đoạn mất lợi thế

---

## 📊 Rolling Winrate

![Rolling Winrate](images/rolling_winrate.png)

- Đánh giá độ ổn định mô hình
- Phân tích độ nhạy regime

---

## 📡 Market Regime Radar

![Market Regime](images/market_regime.png)

Hiển thị:

- Trend strength
- Momentum alignment
- Volatility state

---

## 📉 Price Regression

![Price Regression](images/price_regression.png)

- Độ dốc xu hướng
- Bias thị trường
- Mean reversion behavior

---

## 📊 FP-Growth Rule Mining

![Rule Strength](images/rule_strength.png)

Sử dụng FP-Growth để:

- Tìm pattern giao dịch thắng lặp lại
- Đo strength bằng Support, Confidence, Lift
- Hỗ trợ tối ưu chiến lược

---

# 8️⃣ Yếu Tố Production

Hệ thống được thiết kế để:

- Có thể chạy lại không trùng dữ liệu
- Retry khi job lỗi
- Phân tách xử lý theo coin
- Hỗ trợ mở rộng thêm tài sản
- Kiểm soát duplicate bằng left-anti join
- Quản lý lịch chạy bằng Airflow

---

# 9️⃣ Tech Stack

| Layer | Công nghệ |
|--------|------------|
| Streaming | Kafka |
| Processing | Apache Spark |
| Orchestration | Airflow |
| Storage | MySQL |
| API | Flask |
| ML Mining | Spark ML (FP-Growth) |
| Visualization | Custom Dashboard |

---

# 🔟 Giá Trị Đạt Được

## 📊 Kiến Thức Tài Chính

- Hiểu cấu trúc OHLC
- Momentum & Volatility
- Risk Management
- Edge Quantification

## 🏗 Data Engineering

- Spark Distributed Processing
- Kafka Streaming
- Airflow Orchestration
- Idempotent Pipeline Design
- Data Warehouse Modeling

## 📈 Data Analytics & ML

- Feature Engineering
- Deterministic Scoring
- Backtesting Methodology
- Expectancy Modeling
- Association Rule Mining
- Regression Analysis

## 🧠 System Design Mindset

- Thiết kế scalable
- Phục hồi khi lỗi
- Kiến trúc phân tầng rõ ràng
- Tư duy production-grade

---

# 🏁 Kết Luận

Đây không chỉ là một hệ thống dự đoán crypto.

Đây là một nền tảng xử lý dữ liệu hoàn chỉnh:

- Ingestion thời gian thực
- Xử lý phân tán
- Data Warehouse chuẩn DW
- Modeling định lượng có kiểm chứng
- Pattern mining hỗ trợ tối ưu chiến lược
- Dashboard phục vụ end-user

Dự án thể hiện toàn bộ vòng đời dữ liệu:

Từ dữ liệu thô → xử lý → lưu trữ → modeling → phân tích → insight hành động.
