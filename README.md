# Industrial IoT Sensor Anomaly Detection

Phát hiện bất thường trên dữ liệu cảm biến máy bơm công nghiệp nhằm cảnh báo sớm trước khi máy hỏng (Predictive Maintenance).

**Môn học:** Khai phá dữ liệu

**Đề tài:** 15 – Industrial IoT Sensor Anomaly Detection

**Nhóm thực hiện:**
- B23DCCN069 – Đặng Gia Bảo
- B23DCDT021 – Vũ Thế Anh

## Dữ liệu

- **Nguồn:** Kaggle – [Pump Sensor Data for Predictive Maintenance](https://www.kaggle.com/datasets/nphantawee/pump-sensor-data), file `sensor.csv`
- **Quy mô:** 220.320 dòng × 55 cột; đo mỗi phút từ 01/04/2018 đến 31/08/2018
- **Nội dung:** 52 cảm biến (`sensor_00` … `sensor_51`, tên gốc đã ẩn danh) và cột `machine_status` (NORMAL / RECOVERING / BROKEN)
- Máy bơm hỏng **7 lần** trong giai đoạn này (chỉ 7 dòng BROKEN → dữ liệu mất cân bằng cực mạnh)

Mô tả chi tiết từng cột: xem `data_dictionary.pdf`.

## Cấu trúc thư mục

| File | Nội dung |
|---|---|
| `Industrial-IoT-Sensor-Anomaly-Detection.ipynb` | Notebook chính: EDA, tiền xử lý, 3 mô hình, đánh giá, tuning, trực quan hóa, kết luận (đã có sẵn kết quả chạy) |
| `data_dictionary.pdf` | Mô tả các cột dữ liệu và cách xử lý |
| `README.md` | Giới thiệu dự án |

## Phương pháp

1. **EDA:** giá trị khuyết, phân phối trạng thái máy, cảm biến theo thời gian, ma trận tương quan, outlier theo IQR.
2. **Tiền xử lý:** loại `sensor_15` (trống 100%) và `sensor_50` (thiếu 34,96%); nội suy theo thời gian; gộp theo khung 10 phút; gán nhãn `pre_failure` cho 24 giờ trước mỗi lần hỏng (chỉ dùng để đánh giá); chia train/test **theo thời gian**; chuẩn hóa chỉ trên dữ liệu NORMAL của tập train.
3. **Mô hình phát hiện bất thường không giám sát:** Isolation Forest, Local Outlier Factor (LOF), Autoencoder. Mô hình chỉ học trạng thái bình thường.
4. **Đánh giá:**
   - Mức từng khung: Precision, Recall, F1, ROC-AUC, PR-AUC.
   - Mức sự kiện: số lần hỏng phát hiện được, thời gian cảnh báo sớm, tỷ lệ thời gian báo động nhầm.
5. **Tuning:** thử 192 cấu hình (tham số mô hình × ngưỡng × persist), chọn theo số lần hỏng phát hiện được và mức báo động nhầm.

## Kết quả chính (3 lần hỏng trong tập test)

| Mô hình (sau tuning) | Phát hiện | Cảnh báo sớm TB | Thời gian báo động nhầm |
|---|---|---|---|
| LOF (`n_neighbors=100`) | 3/3 | 21,2 giờ | 13,9% |
| Autoencoder (`bottleneck=16`) | 2/3 | 12,1 giờ | 9,8% |
| Isolation Forest | 1/3 | 3,8 giờ | 0,17% |

- `sensor_04`, `sensor_00`, `sensor_05` lệch mạnh nhất trong 24 giờ trước khi hỏng.
- `sensor_51` chạm giá trị lớn nhất của thang đo khoảng **9 ngày** trước lần hỏng 28/06/2018 – dấu hiệu cảnh báo sớm nhất.
- Hai lần hỏng 08/07 và 25/07 xảy ra đột ngột, gần như không có dấu hiệu báo trước trong dữ liệu.

## Xem kết quả

Notebook đã lưu sẵn toàn bộ kết quả chạy (bảng, biểu đồ, nhận xét): chỉ cần bấm vào file `.ipynb` trên GitHub để xem, không cần chạy lại.

## Chạy lại (tùy chọn)

Nếu muốn tự chạy lại để kiểm chứng:

1. Mở trang dataset trên Kaggle → **Code → New Notebook**.
2. **File → Import Notebook** → chọn `Industrial-IoT-Sensor-Anomaly-Detection.ipynb`.
3. Kiểm tra dataset `pump-sensor-data` đã có ở mục **Input** (nếu chưa: **Add Input**).
4. **Run All**. Không cần GPU.

Lưu ý: Autoencoder có yếu tố ngẫu nhiên khi huấn luyện nên số liệu khi chạy lại có thể lệch nhẹ so với kết quả đã lưu.

**Thư viện:** pandas, numpy, matplotlib, seaborn, scikit-learn, tensorflow (đều có sẵn trên Kaggle).
