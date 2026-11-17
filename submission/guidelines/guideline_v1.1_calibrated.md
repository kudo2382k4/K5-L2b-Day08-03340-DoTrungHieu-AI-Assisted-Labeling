# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603340 - Đỗ Trung Hiếu

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy dày đặc **gần camera**: từng xe hay `ignore_region`? | **Gán từng xe** nếu box cao ≥ 20 px và còn thấy ≥ 20% xe (bánh, ống xả, đuôi xe đều tính là phần nhìn thấy). `ignore_region` **chỉ** cho cụm ≥ 5 vật cao < 20 px ở xa. Không dùng `ignore_region` để bỏ qua xe gần camera bị người ngồi trên che. | img_005, img_018 | CAL-05 |
| 3 | Hàng nhô ra / chất cao trên xe: box xe có gồm hàng không? | Box xe ba gác tính từ **mép thùng / mui gắn liền xe**. Hàng chất cao hơn thùng (buồng chuối, rau) **không** tính vào box xe; áp dụng như quy tắc nhô ~20% của 4.1.5 nhưng theo chiều đứng. | img_008 | CAL-08 |
| 4 | Người ngồi trong xe ba bánh có mui (chỉ thấy một phần) | Người lái / hành khách là **box `person` riêng** khi còn thấy ≥ 20%. Trong mask, người **nằm ngoài** mask xe (người ở phía trước khung xe theo 4.2.4). | img_013 | CAL-06 |
| 5 | Kính chắn gió / cửa sổ xe thuộc mask xe không? | **Kính thuộc mask xe.** Khoảng hở bên hông cabin tuk-tuk (thấy nền hoặc người phía sau) là **lỗ**: bỏ ra khỏi mask nếu lớn hơn ~5% diện tích xe (4.2.2). | img_017, img_013 | CAL-07 |
| 7 | Có đặt mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | **Không đặt.** Box sai vẫn có `conf` cao (box ma motorbike 0.61 trên tán cây, thúng gánh hàng 0.63 bị gọi three_wheeler). `conf` chỉ dùng để sắp thứ tự duyệt; mọi box đều phải được nhìn. | img_014 | CAL-09 |

**Chưa chốt** (không có ảnh ví dụ trong lô 28 ảnh nên chưa đủ căn cứ):
- Câu 1 (xe đạp điện / xe máy điện nhỏ): không gặp ca nào trong gold + batch. Tạm giữ quy tắc 4.3: chọn lớp tin nhất và tick `can_review`.
- Câu 6 (box chồng nhau trong ảnh đêm): lô này không có ảnh đêm. Riêng box trùng ban ngày đã có quy tắc mới (mục 2, CAL-03).

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của tôi | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_004, img_011 | Tuk-tuk có cabin / mui | YOLO26 gọi `car` (0.34–0.42) hoặc `truck` (0.77), có khi bỏ sót hẳn (tuk-tuk đỏ lớn img_004) | Luôn `three_wheeler` (CAL-01) | **Mục 2:** xe 3 bánh, kể cả có cabin kín, là `three_wheeler`. Đếm bánh nhìn thấy trước khi nhận lớp của máy. |
| img_014, img_002 | Gánh hàng rong (quang gánh hai thúng); bán tải; xe tay ga | YOLOE (prompt three_wheeler) gán `three_wheeler` cho thúng (0.33 / 0.63), bán tải (0.37), xe tay ga (0.40) | Xoá box thúng; bán tải → `car`; xe tay ga → `motorbike` (CAL-02) | **Mục 2, dòng `three_wheeler`, cột "Không gồm":** gánh hàng, thúng, xe đẩy không có bánh xe. |
| img_003 | Hai box chồng nhau trên cùng một xe ba gác | YOLOE: một box ôm cả xe (0.90), một box chỉ phủ nửa dưới | Giữ box ôm trọn xe, xoá box phủ một phần (CAL-03) | **Mục 4.1, quy tắc 8 (mới):** hai box cùng lớp có IoU > 0.5 trên cùng một vật → chỉ giữ một box ôm trọn vật. |
| img_033 | Xe đạp ngược sáng, có người đang đạp | YOLO26 gọi `motorbike` (0.69) | `bicycle` (CAL-04) | **Mục 4.3:** phân biệt `bicycle` / `motorbike` bằng bánh mảnh, bàn đạp, có hay không thân máy; không theo lớp của máy. Không chắc → `can_review`. |
| img_013 | Mask SAM của tuk-tuk dính người lái | SAM 2.1-b với 1 điểm âm: mask gồm cả mặt, tay, quần người lái và người ở quán phía sau | Cắt người ra: 3 điểm âm lên từng phần người + box bao xe (CAL-06) | **Mục 4.2, quy trình click (mới):** điểm âm chỉ đặt **ngoài** vật (đặt lên chính xe là vô tác dụng, img_017); xe có người lái → điểm âm cho từng phần người; mask sụp thì thêm box gợi ý. |
| img_005 | Hai box `car` (xe van ở cuối đường) bị đầu người che nhiều | Máy bỏ sót (YOLO26); thêm tay khi duyệt, tick `can_review` | Giữ, vì còn thấy kính trước và nóc xe (≥ 20%) (CAL-05) | Áp dụng 4.1.2 như cũ; ví dụ cho câu 2 ở mục 1. |
| img_014, img_018 | Box ma nằm ngoài mặt đường (tán cây, biển hiệu VIB) | Bẫy extra (box `motorbike` giả) | Xoá. Đây là lỗi của người duyệt, **không** đổi định nghĩa lớp (CAL-10) | **Không đổi guideline.** Thêm vào checklist duyệt: đi lần lượt từng box trong panel Objects, kể cả box ở vùng không thể có xe. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2, 3, 4, 5, 7 (câu 1 và 6 chưa chốt vì không có ảnh ví dụ). Mục 2: thêm "Không gồm" cho `three_wheeler`, nhấn mạnh xe 3 bánh có cabin vẫn là `three_wheeler`. Mục 4.1: thêm quy tắc 8 về box trùng. Mục 4.2: người lái nằm ngoài mask xe, quy trình click SAM. Mục 4.3: phân biệt bicycle / motorbike. | CAL-01 … CAL-09 (CAL-10 chỉ đổi checklist, không đổi guideline) |
