# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 108 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 12/108 = 11.1% | 108 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 6/6 = 100.0% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 392/433 = 90.5% / 18/433 = 4.2% / 22/433 = 5.1% | 433 box máy |
| Máy: added | 45/455 = 9.9% | 455 đối tượng sau duyệt |
| Bắt lỗi bẫy | 9/12 = 75.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 6 | img_005.jpg, img_008.jpg, img_017.jpg |
| thua | 4 | img_003.jpg, img_014.jpg |
| sai_vi_tri_kich_thuoc | 1 | img_017.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 6 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **100%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 100.0% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 6/12 lỗi
- bắt bẫy 75% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
