# Tài Liệu Bàn Giao Dữ Liệu - Dự Án Fraud Detection

Tài liệu này cung cấp thông tin chi tiết về bộ dữ liệu đã qua tiền xử lý cho bài toán phát hiện gian lận (Fraud Detection). Dữ liệu đã được làm sạch, kỹ thuật hóa đặc trưng (Feature Engineering) và tối ưu hóa hoàn toàn để sẵn sàng cho giai đoạn huấn luyện mô hình.

**Link truy cập bộ dữ liệu:** [Google Drive Dataset](https://drive.google.com/drive/folders/18n0sYfK3abH_ar9jkgYlZrQ0ham59ycd?usp=drive_link)

---

## 1. Danh sách tệp tin bàn giao
Dữ liệu bao gồm 3 tệp CSV định dạng chuẩn, được phân tách theo trục thời gian để đảm bảo tính khách quan và tránh hiện tượng rò rỉ dữ liệu (Data Leakage):

* **`train_final.csv`**: Sử dụng trực tiếp để huấn luyện mô hình.
* **`val_final.csv`**: Sử dụng cho quá trình kiểm định, thực hiện Early Stopping và tìm ngưỡng cắt (Threshold) tối ưu.
* **`test_final.csv`**: Tập kiểm thử mù (Blind Test). Tập dữ liệu này chỉ được sử dụng để đánh giá hiệu suất cuối cùng, tuyệt đối không can thiệp vào quá trình huấn luyện và tinh chỉnh.

---

## 2. Các bước tiền xử lý đã thực hiện
Hệ thống dữ liệu đã được xử lý chuyên sâu qua các bước sau:

* **Xử lý dữ liệu khuyết thiếu (Missing Values):** Đã được xử lý triệt để. Các cột dữ liệu số được điền giá trị Trung vị (Median), các cột phân loại được điền nhãn 'Unknown'.
* **Mã hóa (Encoding):** Tất cả các biến định danh (Categorical) như `merchant`, `job`, `zip`... đã được mã hóa bằng phương pháp **Target Encoding** (áp dụng K-Fold và Smoothing = 20). Dữ liệu hiện tại 100% ở dạng số. 
    * *Lưu ý:* Vui lòng không sử dụng One-Hot Encoding để tránh lãng phí tài nguyên hệ thống (nổ RAM).
* **Kỹ thuật hóa đặc trưng (Feature Engineering):** Đã bổ sung các biến hành vi có tính phân loại cao:
    * `travel_speed`: Vận tốc di chuyển giữa hai lần giao dịch liên tiếp.
    * `nb_tx_24h` / `avg_amt_24h`: Số lượng và giá trị giao dịch trung bình trong vòng 24 giờ.
* **Lựa chọn đặc trưng (Feature Selection):** Đã thực hiện phân tích SHAP trên GPU để loại bỏ các biến nhiễu. Bộ dữ liệu hiện chỉ giữ lại 20 đặc trưng quan trọng nhất.

---

## 3. Yêu cầu kỹ thuật khi huấn luyện mô hình
Để đạt được kết quả tốt nhất, quy trình huấn luyện cần tuân thủ các nguyên tắc sau:

### Mục tiêu và Phân phối dữ liệu
* **Biến mục tiêu:** `is_fraud` (1: Gian lận, 0: Giao dịch bình thường).
* **Xử lý mất cân bằng (Imbalance Data):** Đây là yêu cầu quan trọng nhất. Tỷ lệ dữ liệu hiện tại là **172:1** (Âm tính áp đảo). Nếu không xử lý, mô hình sẽ đạt Accuracy rất cao nhưng không có khả năng phát hiện gian lận thực tế.
    * *Giải pháp:* Cần thiết lập các tham số xử lý imbalance (Ví dụ: `scale_pos_weight = 172` trong XGBoost/LightGBM hoặc sử dụng `class_weight='balanced'`).

### Chỉ số đánh giá (Metrics)
* **Tuyệt đối không dùng Accuracy** làm chỉ số đo lường chính.
* Ưu tiên sử dụng **F2-Score** hoặc **Precision-Recall AUC**. 
* Đặc thù bài toán ưu tiên **Recall** để hạn chế tối đa việc bỏ lọt các giao dịch gian lận.

### Ngưỡng so sánh (Baseline)
Kết quả thử nghiệm Baseline bằng mô hình LightGBM chưa tinh chỉnh đã đạt mức **F2-Score: 0.9389 (94%)**. Các phương án tối ưu hóa và tinh chỉnh tham số (Tuning) sau này cần đạt được hiệu suất cao hơn con số này.

---
*Mọi thắc mắc về cấu trúc dữ liệu vui lòng phản hồi lại bộ phận xử lý dữ liệu.*
