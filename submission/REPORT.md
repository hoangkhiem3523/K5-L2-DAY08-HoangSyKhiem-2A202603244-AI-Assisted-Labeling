# Báo cáo Day 08 — 2A202603244 - Hoàng Sỹ Khiêm

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | person=0.35, motorbike=0.35, car=0.5; bus/truck/bicycle=0.3 (mặc định) | Chạy `tour_y26_conf025` và `y26_all_conf005`: YOLO26m nhận diện người và xe máy cực nhạy (15ms/ảnh trên RTX 3050), quét ngưỡng F1-score tối ưu ở 0.35; car tối ưu ở 0.5 để giảm báo nhầm biển hiệu/vật cản. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `tuk-tuk, cyclo, cargo tricycle` | three_wheeler=0.3 | Chạy `tour_yoloe_three_wheeler` và `yoloe_all_conf005`: YOLO26 là closed-set COCO không có nhãn xe ba bánh, buộc phải dùng open-vocabulary YOLOE để gợi ý nhãn xe tuk-tuk, xích lô, xe ba gác. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.35,car=0.5" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn**: SAM 2.1 base (`sam2.1_b.pt`). Lý do: Kiến trúc Hiera phân cấp xử lý rất nhanh (~1.9s/ảnh trên GPU RTX 3050), bộ nhớ tiêu thụ chỉ ~1.8GB VRAM, viền mặt nạ tách sắc nét các chi tiết phức tạp của xe ba gác, xích lô và người lái so với SAM 1 đời cũ.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`): Xe tuk-tuk mui kín trong `img_013.jpg` bị YOLO26 nhận nhầm thành `car` với độ tự tin rất cao (0.82). Vì tập COCO 80 lớp của YOLO26 không có khái niệm `three_wheeler`, nâng hay hạ ngưỡng conf chỉ làm mất box hoặc giữ nguyên nhãn sai `car`. Cách sửa: Dùng mô hình open-vocabulary YOLOE với prompt text riêng, sau đó trong bước merge NMS ưu tiên nhãn `three_wheeler` đè lên `car` nếu IoU >= 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: `img_013.jpg` chiếc xe tuk-tuk bị YOLO26 dự đoán là `car` với conf 0.82 (sai ngữ nghĩa).
  - Điểm thấp mà đúng: `img_001.jpg` người đi bộ ở góc xa bên phải bị che khuất một phần bởi thân xe buýt, conf chỉ đạt 0.28 nhưng thực tế là người đi bộ hoàn chỉnh.
- Giới hạn: Việc quét và chọn ngưỡng thực hiện trên 4 ảnh gold (in-sample) nên số đo Precision và Recall của máy có xu hướng lạc quan hơn thực tế; các lớp ít vật thể như `bus`, `truck`, `bicycle` có dưới 5 đối tượng trong gold nên không đủ ý nghĩa thống kê để chỉnh ngưỡng riêng mà phải giữ mặc định 0.3.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 446 | 446/480 = 92.9% | 480 box máy |
| edited | 26 | 26/480 = 5.4% | 480 box máy |
| deleted | 0 | 0/480 = 0.0% | 480 box máy |
| added | 1 | 1/473 = 0.2% | 473 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  Lớp `three_wheeler` và `person` ở xa. Với `three_wheeler`, YOLO26 bỏ sót 100%, YOLOE phát hiện được nhưng đôi khi vẽ box hơi rộng ôm cả bóng xe hoặc người đứng cạnh (như ở `img_004.jpg`). Với `person`, trong các ảnh dày đặc xe máy (`img_018.jpg`, `img_028.jpg`), máy thường bỏ sót chân người đi bộ bị bánh xe che khuất.
- So với gold: Precision / Recall của **máy**: Precision = 62.7% (KTC 95%: 51–73%), Recall = 87.5% (KTC 95%: 75–94%), mẫu số: 48 đối tượng gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 = 91.7% (KTC 95%: 65–99%, vượt ngưỡng yêu cầu >= 80%).
  - Bẫy missing: bắt được 3/3 (`img_029.jpg`, `img_031.jpg`, `img_028.jpg`).
  - Bẫy wrong_class: bắt được 3/3 (`img_025.jpg`, `img_014.jpg`, `img_034.jpg`).
  - Bẫy extra: bắt được 3/3 (`img_023.jpg`, `img_018.jpg`, `img_027.jpg`).
  - Bẫy loose: bắt được 2/3 (`img_004.jpg`, `img_035.jpg`), để lọt 1 bẫy ở `img_003.jpg`.
- Vì sao lọt (bạn đã bỏ bước nào của checklist)?
  Bẫy lọt ở `img_003.jpg` là một box `motorbike` bị gài lỏng (IoU ~0.65). Do chiếc xe máy nằm trong góc che khuất bởi xe xích lô, khi nhìn tổng thể thấy box đã bao quanh phương tiện và đúng nhãn, người duyệt bị thiên kiến tự động hóa (automation bias) và hiệu ứng mỏi mắt nên lướt nhanh qua mà không phóng to kiểm tra 4 mép bám dính của box. Bước checklist bị bỏ qua là: Zoom tối thiểu 150% để rà soát mép bánh xe tiếp xúc mặt đường đối với các vật thể bị che khuất.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt?
  Accepted cao (92.9%) kèm theo bắt bẫy cao (91.7%) chứng minh người duyệt có sự tỉnh táo, nhận diện được phần lớn gợi ý đúng của mô hình để tăng tốc độ nhưng vẫn kiên quyết loại bỏ các bẫy giả và chỉnh sửa các box lỏng lẻo. Nếu accepted cao mà bắt bẫy thấp (< 60%), điều đó phản ánh hiện tượng "rubber-stamping" (gật đầu lia lịa), biến khâu review con người thành vô nghĩa.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay từ đầu 4.5 phút/ảnh (18 phút / 4 ảnh gold) vs Duyệt có pre-label 2.0 phút/ảnh (48 phút / 24 ảnh batch).
- Thiên lệch: Người làm bài đã gán 4 ảnh gold trước nên đã quen bối cảnh đường phố và phím tắt, nhưng tốc độ duyệt trên lô batch vẫn nhanh gấp 2.25 lần so với vẽ tay từ số không.
- Kết luận: CỰC KỲ ĐÁNG DÙNG. Tiết kiệm hơn 55% thời gian gán nhãn cho toàn bộ dự án, đặc biệt hiệu quả ở các lớp có mật độ cao như `person`, `motorbike`, `car`. Tuy nhiên, với các lớp hiếm (`three_wheeler`), bắt buộc phải có mô hình chuyên biệt và con người rà soát kỹ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Ẩn nhãn máy 3 giây đầu (Blind Scan)*: Mỗi khi chuyển sang ảnh mới, dùng phím tắt ẩn toàn bộ nhãn máy trong 3 giây để mắt tự quét toàn cảnh nhận diện các vật thể sót, sau đó mới bật nhãn máy lên so khớp.
  2. *Quy tắc duyệt ngược (Bottom-up Checklist)*: Tuyệt đối không kiểm tra box theo thứ tự máy gợi ý; ưu tiên tìm vật thể máy bỏ sót (FN) và vật thể máy vẽ ảo (FP) trước, sau đó mới sửa nhãn lớp và nắn chỉnh mép box.
  3. *Quy tắc Zoom 200% bắt buộc*: Đối với mọi vật thể bị che lấp > 30% hoặc vật thể có kích thước lớn (> 150px), bắt buộc zoom 200% kiểm tra 4 mép bounding box trước khi nhấn chấp nhận.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Mask ôm trọn thân xe tuk-tuk và mui che; loại trừ được người đi bộ đứng sát cạnh nhờ click âm | Kính chắn gió được gộp chuẩn xác vào mask thân xe; viền gương chiếu hậu hơi mỏng | `masks/tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 1 click pos + 1 box prompt | Mask ôm khít xe ba bánh chở hàng cồng kềnh; tách biệt được người lái ngồi phía sau | Phần xích và bánh xe phía dưới bị lẹm một góc nhỏ do bóng đổ sẫm màu | `masks/three_wheeler_img004.jpg` |
| `img_003.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 1 click pos + 1 box prompt | Mask bao phủ toàn bộ khung xe xích lô và mui xe vải | Khoảng trống giữa các thanh nan hoa bánh trước bị gộp vào mask do cấu trúc quá mảnh | `masks/cyclo_img003.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2.3 (loại trừ bóng đổ, kính xe thuộc thân xe) và 4.2.4 (vật chạm nhau không chồng mask) được SAM tuân thủ rất tốt khi có click âm định hướng. Điểm guideline còn thiếu là quy định xử lý các chi tiết xuyên thấu siêu nhỏ (như nan hoa bánh xe xích lô hoặc khe hở giữa khung chở hàng).
- Interactor SAM trong CVAT: Đã thử nghiệm Interactor trên CVAT, giao diện click trực quan giúp chỉnh sửa điểm tương tác mượt mà; tuy nhiên chạy qua script `sam2_points.py` offline tận dụng tối đa sức mạnh GPU RTX 3050 cục bộ, lưu ảnh mặt nạ trực tiếp và không phụ thuộc vào độ trễ mạng hay cấu hình container CVAT.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` trong `calibration_decisions.csv`).
- Quyết định quan trọng nhất: `CAL-01` (`img_013.jpg` — Chuẩn hóa quy định xe tuk-tuk ba bánh có mui che luôn gán `three_wheeler`, không để nhầm sang `car`) và `CAL-02` (`img_004.jpg` — Phân định rõ ràng xe đạp điện có bàn đạp là `bicycle`, xe máy điện/scooter không bàn đạp là `motorbike`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  - Bổ sung định nghĩa chi tiết phương tiện 3 bánh có mui thuộc lớp `three_wheeler` (Mục 2.1).
  - Tiêu chí phân loại xe đạp điện và xe máy điện dựa trên bàn đạp cơ học (Mục 6.1).
  - Quy tắc sử dụng `ignore_region` cho các cụm xe máy đông đúc che khuất > 80% (Mục 6.2).
  - Tiêu chuẩn ôm sát mép không chừa khoảng hở > 5px cho xe ba bánh (Mục 4.1).
- Nguyên nhân gốc: Phần lớn do guideline v1.0 chưa bao quát hết các phương tiện giao thông đặc thù tại các đô thị châu Á và giới hạn 80 lớp closed-set của các mô hình gốc phương Tây.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh ngẫu nhiên (seed 42, gồm 106 đối tượng); người kiểm: Hoàng Sỹ Khiêm (hạn chế: tự kiểm tra lại dữ liệu do chính mình vừa duyệt).
- Lỗi theo **đối tượng**: 0/106 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi: Không ghi nhận lỗi sau khi đã rà soát và đối chiếu guideline v1.1.
- **Giới hạn mẫu nhỏ**: Với 0 ảnh lỗi trong 5 ảnh mẫu, cận trên khoảng tin cậy 95% Wilson của tỉ lệ ảnh lỗi thật lên tới **43.0%**. Do đó, mẫu 5 ảnh chỉ mang tính chất phát hiện lỗi thô, không thể khẳng định một cách tuyệt đối về mặt toán học rằng tỉ lệ lỗi toàn tập dưới 2%.
- Script kiểm được / không kiểm được: Script kiểm tra tự động được các vi phạm hình học (box ngoài biên ảnh, box quá nhỏ < 20px, sai tên nhãn, polygon không khép kín); script KHÔNG THỂ kiểm tra được độ ôm sát ngữ nghĩa, không phân biệt được người gán nhãn nhầm xe máy thành xe đạp nếu box hợp lệ, và không phát hiện được vật thể mà cả máy lẫn người đều bỏ sót.

**Quyết định: RELEASE**, ngày 2026-11-17.
Lý do:
- Không có khúc mắc mở về guideline hay dữ liệu hỏng (Guideline gap = 0, Data problem = 0 -> Không cần HOLD).
- Tỉ lệ ảnh lỗi QC là 0.0% <= 2%, không xuất hiện lỗi hệ thống lặp lại, tỉ lệ bắt bẫy đạt 91.7% (11/12) vượt ngưỡng tối thiểu 80% (-> Không cần REWORK).
- Bộ dữ liệu đáp ứng đầy đủ tiêu chuẩn nghiệm thu và sẵn sàng đóng gói xuất bản.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity làm trợ lý kiểm tra cú pháp, hỗ trợ phân tích code các script đo lường, rà soát tính hợp lệ của checklist và đối chiếu rubric. Toàn bộ quyết định gán nhãn và kiểm tra hình ảnh đều do học viên thực hiện.
- Gold có bị nhiễm không: HOÀN TOÀN KHÔNG. 4 ảnh gold được gán tay độc lập từ đầu trên CVAT Task 26 trước khi chạy tour model ở Part 2 hay quét ngưỡng ở Part 3.
- Máy / GPU / CVAT dùng: Laptop ASUS TUF Gaming F15, Windows 11 64-bit, GPU NVIDIA GeForce RTX 3050 Laptop GPU (CUDA 12.x), CVAT v2.74.1 chạy local docker container.
- Dùng cache `data/cache/` thay vì tự chạy: KHÔNG DÙNG CACHE. Toàn bộ suy luận YOLO26m, YOLOE-26s, SAM 2.1 đều được chạy trực tiếp trên GPU máy cá nhân.
- Lỗi / bug script đã gặp: Gặp lỗi thiếu thư viện `transformers` và `clip` khi chạy YOLOE/OWLv2, đã cài đặt bổ sung vào môi trường `.venv`; ngoài ra cần lưu ý bọc dấu nháy kép cho các tham số chuỗi trong PowerShell để tránh bị cắt chuỗi.
- Một điều bạn làm khác lần sau: Sẽ viết sẵn một hook kiểm tra tự động tỷ lệ IoU của các box giáp ranh ngưỡng 0.9 để cảnh báo sớm các trường hợp bẫy loose trước khi hoàn tất phiên review.
