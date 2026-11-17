# Thông tin học viên

- Họ và tên: Hoàng Sỹ Khiêm
- MSHV: 2A202603244
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: hoangkhiem3523
- Repo bài nộp (URL): https://github.com/hoangkhiem3523/K5-L2-DAY08-HoangSyKhiem-2A202603244-AI-Assisted-Labeling
- Commit chốt bài (hash): 9fb8628
- Máy: Windows 11, GPU NVIDIA GeForce RTX 3050 Laptop GPU (CUDA)
- CVAT (Docker): v2.74.1 (Docker local)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 446 / 26 / 0 (mẫu số: 480 box máy, 92.9% / 5.4% / 0.0%); added: 1 (mẫu số: 473 đối tượng sau duyệt, 0.2%)
- So với gold: precision / recall của **máy**: 62.7% / 87.5% (mẫu số: 48 đối tượng gold, KTC 95%: Precision 51–73%, Recall 75–94%)
- Bắt bẫy: 11/12 = 91.7% (KTC 95%: 65–99%, ngưỡng đạt: >= 80%)
- Thời gian: gán tay 4.5 phút/ảnh (18 phút / 4 ảnh gold); duyệt có pre-label 2.0 phút/ảnh (48 phút / 24 ảnh batch) -> nhanh gấp 2.25 lần
- Lỗi QC: 0.0% theo ảnh (0/5 ảnh trong mẫu), 0.0% theo đối tượng (0/106 đối tượng trong mẫu)
- **Quyết định:** RELEASE — lý do một dòng: Tỉ lệ ảnh lỗi QC 0.0% <= 2%, bắt bẫy đạt 91.7% >= 80%, không có lỗi hệ thống và các ca khó mơ hồ đã được chuẩn hóa vào guideline v1.1.
