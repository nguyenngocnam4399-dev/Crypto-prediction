# 🚀 Deterministic Quantitative Trading Research Platform  
### Hệ Thống Phân Tích & Dự Đoán Tài Sản Số Dựa Trên Dữ Liệu Thời Gian Thực  

**Author:** Nguyễn Ngọc Nam  
**Mentor:** Phạm Long Vân - Data Manager  
**Location:** Ho Chi Minh City, Vietnam — 2025  

---

# 1️⃣ Bối Cảnh Thực Tiễn & Động Lực Xây Dựng

Tài sản số (crypto assets) như BTC, ETH và BNB đang dần trở thành một lớp tài sản có ảnh hưởng thực sự trong hệ sinh thái tài chính toàn cầu. Không chỉ nhà đầu tư cá nhân, mà cả tổ chức và chính phủ cũng bắt đầu quan tâm đến việc quản lý và cấp phép loại tài sản này. Khi khung pháp lý dần hình thành, nhu cầu về một hệ thống phân tích dữ liệu đáng tin cậy và có cơ chế kiểm chứng trở nên cấp thiết.

Trong thực tế, phần lớn nhà đầu tư cá nhân hiện nay vẫn ra quyết định dựa trên cảm tính, tin tức rời rạc hoặc các công cụ phân tích thiếu kiểm định hiệu suất. Họ thiếu một môi trường có thể:

- Thu thập dữ liệu liên tục  
- Phân tích có cấu trúc  
- Kiểm định tín hiệu trước khi sử dụng  
- Đánh giá độ ổn định chiến lược theo thời gian  

Dự án này được xây dựng xuất phát từ nhu cầu đó: cung cấp cho người dùng một hệ thống phân tích định lượng có khả năng xử lý dữ liệu liên tục, tích hợp cả yếu tố kỹ thuật lẫn tâm lý thị trường, đồng thời có cơ chế kiểm chứng rõ ràng.

Hệ thống tập trung vào BTC, ETH và BNB vì đây là các tài sản có:

- Thanh khoản cao  
- Lịch sử dữ liệu dài  
- Độ ổn định tương đối  
- Ảnh hưởng lớn đến thị trường  

Việc chọn nhóm tài sản này giúp đảm bảo tính thực tiễn và tính bền vững của phân tích.

---

# 2️⃣ Xác Định Bài Toán & Hướng Giải Quyết

Bài toán đặt ra không đơn thuần là sinh tín hiệu BUY/SELL, mà là xây dựng một hệ thống hoàn chỉnh có khả năng:

- Thu thập dữ liệu real-time  
- Chuẩn hóa và lưu trữ có cấu trúc  
- Phân tích kỹ thuật & sentiment  
- Sinh tín hiệu có thể giải thích  
- Kiểm chứng hiệu suất  
- Trình diễn cho end-user  

Để giải quyết bài toán này, hệ thống được chia thành các giai đoạn rõ ràng.

---

# 3️⃣ Thu Thập Dữ Liệu & Phân Tích EDA

## 3.1 Dữ Liệu Giá (OHLCV)

Dữ liệu được stream real-time từ Binance thông qua API/WebSocket.

Các trường chính:

- Open  
- High  
- Low  
- Close  
- Volume  

Việc thu thập OHLCV xuất phát từ phân tích EDA:

- Close phản ánh điểm đồng thuận cuối cùng của thị trường trong một khoảng thời gian.  
- High và Low cho biết mức độ biến động và sức ép cung cầu.  
- Volume phản ánh dòng tiền và xác nhận breakout hoặc đảo chiều.  

Các chỉ báo kỹ thuật như RSI, EMA, MACD đều được tính toán từ cấu trúc OHLCV này. Do đó, đây là nền tảng không thể thiếu.

## 3.2 Dữ Liệu Tin Tức & Sentiment

Tin tức được crawl từ các trang crypto và xử lý sentiment.

Lý do thu thập sentiment:

- Thị trường crypto phản ứng mạnh với tin tức.  
- Tâm lý nhà đầu tư ảnh hưởng trực tiếp đến biến động ngắn hạn.  
- Một số biến động không thể giải thích chỉ bằng indicator kỹ thuật.  

Việc kết hợp kỹ thuật và sentiment giúp giảm phụ thuộc vào một nguồn tín hiệu duy nhất.

---

# 4️⃣ Thiết Kế Kiến Trúc Hệ Thống

Luồng tổng thể:

Market / News → Kafka → Spark → Data Warehouse → Metric Engine → Prediction → Backtesting → Analytics → Web

## 4.1 Vì Sao Sử Dụng Kafka?

Kafka đóng vai trò lớp đệm giữa ingestion và processing.

Lý do chọn Kafka:

- Thị trường hoạt động 24/7, cần xử lý streaming  
- Tránh phụ thuộc trực tiếp vào API  
- Cho phép retry và xử lý lại khi Spark job lỗi  
- Tăng tính ổn định hệ thống  

Nếu không có lớp trung gian này, hệ thống sẽ dễ mất dữ liệu khi upstream gặp sự cố.

---

## 4.2 Vì Sao Sử Dụng Spark?

Spark hỗ trợ:

- Xử lý rolling window  
- Tính toán indicator phân tán  
- Xử lý sentiment theo batch lớn  

Do dữ liệu thị trường tăng liên tục, việc xử lý đơn luồng sẽ không đủ hiệu quả.

---

## 4.3 Vì Sao Thiết Kế Theo Dim–Fact?

Hệ thống sử dụng mô hình Data Warehouse với dimension và fact tách biệt.

Dimension chứa thông tin mô tả (symbol, interval, indicator).  
Fact chứa sự kiện (giá, metric, prediction).

Lý do thiết kế này:

- Giảm redundancy  
- Chuẩn hóa dữ liệu  
- Hỗ trợ truy vết vòng đời tín hiệu  
- Dễ audit và kiểm định  

Nếu lưu toàn bộ trong một bảng lớn, hệ thống sẽ khó kiểm soát và khó mở rộng.

---

# 5️⃣ Orchestration & Reliability

Apache Airflow được sử dụng để:

- Lập lịch chạy định kỳ  
- Quản lý dependency  
- Retry khi lỗi  
- Ghi log  

Hệ thống được thiết kế idempotent để tránh duplicate dữ liệu.  
Nếu một job thất bại, hệ thống có thể chạy lại mà không làm sai lệch kết quả.

Monitoring giúp đảm bảo pipeline không bị gián đoạn.

---

# 6️⃣ Indicator, Metric & Prediction Engine

Indicator kỹ thuật được tính toán và chuyển thành metric logic (ví dụ: RSI < 30).

Prediction Engine tính toán:

buy_score và sell_score dựa trên metric có trọng số.

Tín hiệu được phân loại dựa trên edge (chênh lệch điểm).

Hệ thống được thiết kế deterministic để:

- Có thể giải thích  
- Có thể audit  
- Tránh black-box  

---

# 7️⃣ Vì Sao Sử Dụng FP-Growth?

Thay vì sử dụng mô hình ML black-box ngay từ đầu, hệ thống sử dụng FP-Growth để:

- Tìm pattern thường xuất hiện trong trade thắng  
- Tính lift và confidence  
- Kiểm chứng cấu trúc chiến lược  

FP-Growth không dùng để dự đoán trực tiếp, mà để xác nhận tính bền vững của tổ hợp điều kiện.

Điều này giúp tăng độ tin cậy trước khi mở rộng sang ML phức tạp hơn.

---

# 8️⃣ Trình Diễn Cho End-User

Web interface hiển thị:

- Tín hiệu BUY / SELL  
- Edge & Confidence  
- Equity Curve  
- Win Rate & Drawdown  

Thiết kế UI tập trung vào khả năng giúp người dùng đánh giá nhanh tín hiệu mà không cần hiểu kiến trúc phía sau.

---

# 🔟 Kết Luận & Giá Trị Đạt Được

Dự án này không chỉ đơn thuần là xây dựng một pipeline dữ liệu crypto, mà là quá trình thiết kế một hệ thống định lượng dựa trên nhu cầu thực tế của thị trường tài sản số.

Quá trình thực hiện giúp nâng cao hiểu biết về domain tài chính, từ cấu trúc OHLCV đến quản trị rủi ro. Đồng thời, năng lực Data Engineering được phát triển thông qua việc xây dựng pipeline streaming với Kafka, xử lý phân tán bằng Spark, tổ chức Dim–Fact theo chuẩn Data Warehouse và vận hành bằng Airflow.

Quan trọng hơn, dự án hình thành tư duy thiết kế hệ thống theo hướng có kiểm soát, có kiểm chứng và có khả năng tạo giá trị cho end-user. Thay vì viết mã xử lý rời rạc, hệ thống được xây dựng như một nền tảng có khả năng mở rộng, audit và cải tiến liên tục.

Đây là bước chuyển từ tư duy lập trình sang tư duy kiến trúc hệ thống phục vụ thực tiễn.
