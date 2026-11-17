# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 107 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 3/107 = 2.8% | 107 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/6 = 33.3% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 392/433 = 90.5% / 18/433 = 4.2% / 22/433 = 5.1% | 433 box máy |
| Máy: added | 45/455 = 9.9% | 455 đối tượng sau duyệt |
| Bắt lỗi bẫy | 9/12 = 75.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 3 | img_005.jpg, img_033.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 2 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **70%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 33.3% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 3/3 lỗi
- bắt bẫy 75% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
