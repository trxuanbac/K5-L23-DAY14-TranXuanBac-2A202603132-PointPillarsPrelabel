# Báo cáo Day 14 — Trần Xuân Bắc — 2A202603132

- Mã học viên dùng với `--student`: `2A202603132`
- `seed_tag` trong `frames_A.txt`: `393590736f`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

- **Tham số dự đoán** Tôi dự đoán tăng ngưỡng tin cậy score_thr lên 0.4 và đặt min_points=5 để loại bỏ các dự đoán có độ tin cậy thấp hoặc quá ít điểm LiDAR
- **Lý do dự đoán từ ảnh BEV hoặc số liệu tập A** Kết quả prelabel ban đầu trên tập A tạo ra 76 cuboid, trong khi nhãn tham chiếu chỉ có 38 cuboid. Số lượng cuboid cần xóa lên tới 41, cho thấy mô hình đang tạo ra khá nhiều dự đoán dư thừa. Vì vậy, tôi dự đoán tăng score_thr sẽ giúp hạn chế các đối tượng nhận diện không chắc chắn, còn tăng min_points giúp loại bỏ các hộp được tạo từ quá ít điểm LiDAR. Tôi kỳ vọng cấu hình này sẽ giảm số cuboid cần xóa và giảm tổng chi phí chỉnh sửa. Tuy nhiên, cần theo dõi nguy cơ bỏ sót các đối tượng nhỏ hoặc ở xa. 

## 2. Tập A: sửa tay và tune

Tập A gồm ba frame. Cấu hình gốc tạo 76 cuboid, còn bộ nhãn tham chiếu đã lưu trên CVAT gồm 38 cuboid. Chi phí được tính bằng `0,3 × xóa + 1,0 × thêm + 0,5 × sửa` trong phạm vi 50 m.

| Frame | Prelabel gốc | Tham chiếu | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|---:|---:|
| `s01f18` | 31 | 10 | 22 | 1 | 3 | 9,10 |
| `s02f10` | 28 | 17 | 13 | 2 | 0 | 5,90 |
| `s04f48` | 17 | 11 | 6 | 0 | 0 | 1,80 |
| **Tổng** | **76** | **38** | **41** | **3** | **3** | **16,80** |

Công cụ hỗ trợ đối chiếu điểm LiDAR với ảnh camera đã được dùng để rà và xóa sơ bộ 26 cuboid prelabel thừa trong job 33. Các số xóa, thêm và sửa trong bảng là kết quả đo giữa `pred_A.json` và `ref_A.json` sau khi hoàn tất bước sửa tập A.

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | 41 | 3 | 3 | 16.80 |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_<ID>/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_<ID>/frames_B.txt --config d14_<ID>/best_config.json --gt d14_<ID>/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Tập A chỉ có 3 frame nên kết quả chọn cấu hình có thể phụ thuộc vào cảnh và số lượng vật thể của các frame đó.
- PointPillars trong lab được huấn luyện trên KITTI. Lớp Truck và Bus được suy ra theo kích thước của hộp dự đoán Car, nên cần đối chiếu lại bằng ảnh và LiDAR.
- Chi phí công sửa là điểm có trọng số theo số thao tác. Muốn quy đổi thành thời gian cần đo thời gian gán nhãn thực tế.
