# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 480; đối tượng sau duyệt: 491; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 446 | 446/480 = 92.9% | 480 box máy |
| edited (sửa) | 9 | 9/480 = 1.9% | 480 box máy |
| deleted (xoá) | 25 | 25/480 = 5.2% | 480 box máy |
| added (vẽ thêm) | 36 | 36/491 = 7.3% | 491 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 34.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 253 | 4 | 3 | 9 |
| bicycle | 6 | 0 | 3 | 2 |
| motorbike | 128 | 1 | 2 | 6 |
| car | 37 | 0 | 2 | 15 |
| bus | 3 | 0 | 0 | 0 |
| truck | 5 | 0 | 2 | 1 |
| three_wheeler | 14 | 4 | 13 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 48 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 25 | 6 | 62.7% (KTC 95%: 51–73%) | 87.5% (KTC 95%: 75–94%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - wrong_class: 3/3
  - loose: 3/3
  - extra: 3/3
  - lọt: img_029.jpg (missing), img_028.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
