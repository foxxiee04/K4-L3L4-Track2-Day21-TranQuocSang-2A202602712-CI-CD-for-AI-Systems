# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Trần Quốc Sang |
| MSSV | 2A202602712 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/foxxiee04/K4-L3L4-Track2-Day21-TranQuocSang-2A202602712-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ tham số lần 3 đạt F1-score cao nhất (0.7149), vượt ngưỡng Quality Gate (f1 >= 0.65). Dù lần 1 có accuracy cao nhất (0.8780), F1-score lại thấp hơn (0.7109 so với 0.7149), khẳng định accuracy cao không đồng nghĩa mô hình bắt tốt lớp thiểu số. Giữa learning_rate và n_estimators có sự đánh đổi: ở lần 2, giảm learning_rate xuống 0.05 và chỉ dùng 50 cây nông (max_depth=2) khiến mô hình underfitting, F1 giảm còn 0.6051 (trượt Quality Gate). Tăng n_estimators lên 200 cùng max_depth=5 giúp mô hình học sâu và bắt trọn quan hệ phi tuyến.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult mất cân bằng lớp rõ rệt: chỉ 24.8% mẫu thuộc lớp thu nhập > 50K (lớp dương), còn 75.2% thuộc lớp <= 50K (lớp âm). Một mô hình đơn giản luôn đoán nhãn 0 vẫn đạt accuracy 75.2% nhưng vô dụng vì bỏ sót toàn bộ người thu nhập cao. Do đó, hệ thống chọn F1-score của lớp dương làm tiêu chí đánh giá vì đây là trung bình điều hòa giữa Precision và Recall của nhóm thiểu số. Không dùng average="macro" hay average="weighted" vì lớp đa số (75.2%) sẽ kéo điểm lên cao giả tạo, làm mất ý nghĩa kiểm soát của Quality Gate.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Xác thực quyền truy cập S3 bucket trên AWS cho DVC | User IAM chưa được cấp chính sách thao tác trên S3 | Gán quyền AmazonS3FullAccess cho IAM user và cấu hình DVC remote tới S3 |
| Lỗi unpickle mô hình scikit-learn khi chạy API trên EC2 | Phiên bản scikit-learn trên VM (1.7.2) lệch so với bản huấn luyện (1.4.2) | Cài đặt chính xác scikit-learn==1.4.2 trên VM để tương thích hoàn toàn |
| Xung đột SQLAlchemy và MLflow | SQLAlchemy 2.1 không tương thích với MLflow 2.13 SQLite backend | Ghim phiên bản sqlalchemy<2.1 trong requirements.txt |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung 22.361 mẫu từ train_batch2, F1-score tăng nhẹ từ 0.7149 lên 0.7354 và accuracy tăng từ 0.8740 lên 0.8820. Do hai batch cùng nguồn và cùng phân phối, mô hình đã học phần lớn đặc trưng chính từ batch đầu nên thêm dữ liệu chỉ cải thiện nhẹ chỉ số. Quan trọng nhất, quy trình CI/CD đã tự động kích hoạt huấn luyện lại và kiểm tra Quality Gate thành công khi nhận commit dữ liệu DVC mới mà không cần can thiệp thủ công.
