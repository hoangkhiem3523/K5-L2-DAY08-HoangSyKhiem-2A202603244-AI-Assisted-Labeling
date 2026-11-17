# Thông tin học viên

- Họ và tên: Hoàng Sỹ Khiêm
- MSHV: 2A202603244
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: hoangkhiem3523
- Repo bài nộp (URL): https://github.com/hoangkhiem3523/K5-L2-DAY08-HoangSyKhiem-2A202603244-AI-Assisted-Labeling
- Commit chốt bài (hash): 9a84b6a
- Máy: Windows 11, GPU NVIDIA GeForce RTX 3050 Laptop GPU (CUDA)
- CVAT (Docker): v2.74.1 (Docker local)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 461 / 5 / 14 (mẫu số: 480 box máy, 96.0% / 1.0% / 2.9%); added: 21 (mẫu số: 487 đối tượng sau duyệt, 4.3%)
- So với gold: precision / recall của **máy**: 62.7% / 87.5% (mẫu số: 48 đối tượng gold, KTC 95%: Precision 51–73%, Recall 75–94%)
- Bắt bẫy: 12/12 = 100.0% (KTC 95%: 76–100%, ngưỡng đạt: >= 80%)
- Thời gian: gán tay 4.5 phút/ảnh (18 phút / 4 ảnh gold); duyệt có pre-label 2.0 phút/ảnh (48 phút / 24 ảnh batch) -> nhanh gấp 2.25 lần
- Lỗi QC: 0.0% theo ảnh (0/5 ảnh trong mẫu), 0.0% theo đối tượng (0/106 đối tượng trong mẫu)
- **Quyết định:** RELEASE — lý do một dòng: Tỉ lệ ảnh lỗi QC 0.0% <= 2%, bắt bẫy đạt 12/12 = 100.0% >= 80%, không có lỗi hệ thống và các ca khó mơ hồ đã được chuẩn hóa vào guideline v1.1.
