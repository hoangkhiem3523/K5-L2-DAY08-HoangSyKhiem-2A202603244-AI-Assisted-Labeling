# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603244 - Hoàng Sỹ Khiêm

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: gán `bicycle` hay `motorbike`? | Phân loại theo bàn đạp cơ học: Nếu xe có bàn đạp trợ lực (pedals) và dáng xe đạp thanh mảnh -> gán `bicycle`. Nếu là xe máy điện/scooter không có bàn đạp (như VinFast Klara/Evo, Vespa điện) -> gán `motorbike`. | `img_004.jpg` | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc che khuất ở vỉa hè / bãi xe: vẽ từng xe hay `ignore_region`? | Nếu cụm xe máy từ 5 chiếc trở lên đỗ san sát, bị che khuất > 80% hoặc kích thước từng xe quá nhỏ (< 20px) khiến không thể phân tách viền rõ ràng -> vẽ 1 đa giác/hình chữ nhật `ignore_region` bao quanh toàn bộ cụm xe. Không cố vẽ từng xe để tránh nhiễu mô hình. | `img_018.jpg` | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk / xe ba gác có mui che và kính chắn gió chở khách trên phố | YOLO26m gán nhầm thành `car` do mui kín và cửa xe nhìn giống ô tô con | Sửa thành `three_wheeler` (sử dụng YOLOE open-vocabulary hoặc sửa tay) | Mục 2.1: Bổ sung định nghĩa: Mọi phương tiện 3 bánh có mui che, khung xe ba gác/xích lô cơ giới luôn phân loại là `three_wheeler`, tuyệt đối không gán `car`. |
| `img_003.jpg` | Xe xích lô / ba gác chở đồ cồng kềnh bị che khuất một phần | YOLO26 vẽ box quá lỏng (bẫy loose), chừa khoảng trống lớn quanh xe | Thu hẹp bounding box ôm sát bánh trước và đuôi xe | Mục 4.1: Bounding box của `three_wheeler` phải bám sát phần thân cơ học nhìn thấy, chừa không quá 5px khoảng hở so với mép thực tế. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | 1. Chốt phân loại xe đạp điện (`bicycle`) và xe máy điện (`motorbike`) dựa trên bàn đạp cơ học.<br/>2. Quy định dùng `ignore_region` cho các cụm xe máy dày đặc che khuất > 80%.<br/>3. Chuẩn hóa phân loại `three_wheeler` đối với xe tuk-tuk có mui kín.<br/>4. Siết chặt quy chuẩn ôm sát box cho xe ba gác. | CAL-01 (`img_013.jpg`), CAL-02 (`img_004.jpg`), CAL-03 (`img_018.jpg`), CAL-04 (`img_003.jpg`) |
