# 🚀 Nền Tảng Phân Tích & Nghiên Cứu Định Lượng Crypto (Production-Grade)

> Hệ thống Data Engineering thời gian thực & nền tảng nghiên cứu giao dịch định lượng  
> Tài sản trọng tâm: BTC • ETH • BNB

---

# 🌍 Bối Cảnh Thị Trường & Nhu Cầu Thực Tế

Trong những năm gần đây, tài sản số như BTC, ETH và BNB đang dần trở thành một lớp tài sản được quan tâm rộng rãi tại Việt Nam và toàn cầu.

- Số lượng nhà đầu tư tăng mạnh  
- Biến động giá cao  
- Quyết định giao dịch thường dựa trên cảm tính  
- Thiếu hệ thống định lượng minh bạch  

Trong bối cảnh tài sản số đang dần được quản lý và thể chế hóa, nhu cầu về một hệ thống:

- Phân tích dữ liệu real-time  
- Đưa ra tín hiệu có cơ sở thống kê  
- Đánh giá hiệu suất minh bạch  
- Hỗ trợ quyết định khách quan  

trở nên cấp thiết.

Dự án này được xây dựng nhằm giải quyết nhu cầu đó bằng cách kết hợp Data Engineering, Data Warehouse và mô hình thống kê.

---

## 📌 Tóm Tắt Điều Hành (Executive Summary)

Đây là một nền tảng xử lý dữ liệu và nghiên cứu định lượng được thiết kế theo định hướng production, với mục tiêu:

- Thu thập dữ liệu thị trường crypto theo thời gian thực  
- Chuẩn hóa và tổ chức dữ liệu theo mô hình Data Warehouse (Dim-Fact)  
- Xây dựng hệ thống chấm điểm (scoring) tín hiệu giao dịch có tính xác định  
- Backtest và đánh giá độ ổn định chiến lược  
- Khai phá pattern giao dịch thắng bằng FP-Growth  
- Trình diễn phân tích qua Dashboard  

Hệ thống được xây dựng với tư duy:

- Scalability  
- Idempotency  
- Fault tolerance  
- Traceability  

---

# 1️⃣ Kiến Trúc Tổng Thể Hệ Thống

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

# 2️⃣ Thiết Kế Data Engineering

## 🔄 Ingestion Real-Time

- Kafka giúp tách biệt producer & consumer  
- Hỗ trợ replay dữ liệu  
- Scale ngang khi volume tăng  
- Giảm phụ thuộc trực tiếp vào API  

## ⚡ Xử Lý Phân Tán (Spark)

Spark được dùng để:

- Tính indicator (RSI, MACD, EMA, BB, ADX, VWAP, ATR, OBV)  
- Tính metric giao dịch  
- Xây dựng market score  
- Xác nhận backtest  
- Chuẩn bị dữ liệu mining  

Thiết kế đảm bảo:

- Partition theo symbol  
- Anti-duplicate write (left-anti join)  
- Idempotent JDBC insert  
- Tách biệt xử lý theo từng tài sản  

---

# 3️⃣ Data Understanding (EDA) & Data Dictionary Reasoning

Việc thu thập dữ liệu không chỉ mang tính kỹ thuật mà dựa trên hành vi hình thành giá trong thị trường crypto.

## 📊 Market Data (OHLCV)

| Trường | Vai trò |
|--------|--------|
| Open/Close | Xác định cấu trúc nến |
| High/Low | Đo biên độ dao động |
| Volume | Xác nhận dòng tiền |
| Timestamp | Phân tích theo chu kỳ |

OHLCV là nền tảng của mọi indicator. Nếu không chuẩn hóa theo timeframe, phân tích sẽ mất ý nghĩa.

---

## 📈 Momentum Indicators (RSI, MACD)

- RSI đo trạng thái quá mua/quá bán  
- MACD phát hiện chuyển pha động lượng  
- Histogram MACD nhận diện momentum weakening  

Crypto thường xuất hiện pha overbought/oversold mạnh → RSI rất hữu ích.

---

## 📉 Trend Indicators (EMA, ADX)

- EMA200 xác định xu hướng dài hạn  
- EMA20/50 cho trung & ngắn hạn  
- ADX đo sức mạnh xu hướng  

Mục tiêu: tránh giao dịch ngược xu hướng chính.

---

## 📊 Volatility Indicators (BB, ATR)

Crypto có đặc điểm:

- Giai đoạn squeeze  
- Giai đoạn expansion  

BB đo độ mở biên  
ATR hỗ trợ thiết lập Stop Loss

---

## 📊 Volume Metrics

Volume xác nhận tín hiệu:

- Breakout không có volume → dễ false  
- Divergence volume → cảnh báo suy yếu  

---

## 📰 News & Sentiment

Crypto phản ứng mạnh với:

- Regulation  
- ETF  
- Exchange hack  
- Macro event  

Sentiment giúp bổ sung góc nhìn tâm lý thị trường.

---

## 🎯 Mục Tiêu EDA

- Xác định indicator đóng góp Win Rate  
- Loại bỏ metric không có ý nghĩa  
- Tối ưu trọng số scoring  
- Giảm overfitting  

---

# 4️⃣ Kiến Trúc Data Warehouse

## 🗄 Dim-Fact Modeling

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

### Vì Sao Chọn Dim-Fact?

- Tách context và event  
- Tối ưu phân tích  
- Lưu lịch sử rõ ràng  
- Mở rộng linh hoạt  

---

# 5️⃣ Framework Modeling & Scoring

Market Score =  
Trend + Momentum + Volume + Volatility  

Confidence Score =  
Market Score / Max Score  

Guard Logic:

- Conflict Detection  
- Weak Edge Filter  
- Confidence Band Filter  
- No-Trade Logic  

---

# 6️⃣ Backtest & Risk Modeling

Backtest đánh giá:

- TP / SL động  
- Win/Loss classification  
- PnL normalization  
- Rolling expectancy  
- Regime analysis  

Đảm bảo:

- Survivability  
- Edge stability  
- Tránh overfitting  

---

# 7️⃣ Phân Tích Hiệu Suất

## 📈 Equity Curve

![Equity Curve](images/equity_curve.png)

## 📉 Rolling Expectancy

![Rolling Expectancy](images/rolling_expectancy.png)

## 📊 Rolling Winrate

![Rolling Winrate](images/rolling_winrate.png)

## 📡 Market Regime

![Market Regime](images/market_regime.png)

## 📉 Regression

![Price Regression](images/price_regression.png)

## 📊 FP-Growth Rules

![Rule Strength](images/rule_strength.png)

---

# 8️⃣ Production Considerations

- Idempotent pipeline  
- Retry logic qua Airflow  
- Duplicate prevention  
- Config-driven metric activation  
- Partition-aware Spark execution  

---

# 9️⃣ Tech Stack

| Layer | Công nghệ |
|--------|------------|
| Streaming | Kafka |
| Processing | Apache Spark |
| Orchestration | Airflow |
| Storage | MySQL |
| API | Flask |
| ML | Spark ML (FP-Growth) |

---

# 🔟 Giá Trị Đạt Được

## Financial
- Hiểu cấu trúc thị trường
- Momentum & Volatility
- Risk management

## Data Engineering
- Spark
- Kafka
- Airflow
- DW Modeling
- Idempotent design

## Analytics & ML
- Feature engineering
- Deterministic scoring
- Backtesting
- Pattern mining
- Regression

## System Design
- Scalable architecture
- Fault recovery
- Production mindset

---

# 🏁 Kết Luận

Đây không chỉ là hệ thống dự đoán crypto.

Đây là nền tảng xử lý dữ liệu hoàn chỉnh:

Từ ingestion → processing → warehouse → modeling → analytics → actionable insight.
