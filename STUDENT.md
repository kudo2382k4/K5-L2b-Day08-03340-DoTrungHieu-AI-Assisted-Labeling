# Thông tin học viên

- Họ và tên: Đỗ Trung Hiếu
- MSHV: 2A202603340
- Lớp (L2 / L3 / khác): L2b
- Tài khoản GitHub: kudo2382k4
- Repo bài nộp (URL): https://github.com/kudo2382k4/K5-L2b-Day08-03340-DoTrungHieu-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: Windows 11, không có GPU (chạy CPU)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 392/433 = 90.5% / 18/433 = 4.2% / 22/433 = 5.1% (mẫu số: 433 box máy); added: 45/455 = 9.9% (mẫu số: 455 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 42/61 = 68.9% / 42/58 = 72.4% (4 ảnh gold, 58 đối tượng; lạc quan vì ngưỡng chọn trên chính gold)
- Bắt bẫy: 9/12 = 75.0% (lọt 2 extra, 1 loose)
- Thời gian: gán tay 29/4 ≈ 7.3 phút/ảnh; duyệt có pre-label 76/24 ≈ 3.2 phút/ảnh
- Lỗi QC (vòng 2, sau sửa): 33.3% theo ảnh (2/6), 2.8% theo đối tượng (3/107); vòng 1: 6/6 ảnh, 12/108 đối tượng
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 2/6 > 2%, bỏ sót lặp lại (3/3 lỗi vòng 2) và bắt bẫy 75% < 80%
