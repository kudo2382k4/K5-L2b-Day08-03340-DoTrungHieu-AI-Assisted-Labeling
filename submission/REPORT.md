# Báo cáo Day 08 — 2A202603340 - Đỗ Trung Hiếu

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / motorbike / car | YOLO26m (`yolo26m.pt`, cache `--conf 0.05`, `r05`) | person 0.35, motorbike 0.5, car 0.35 (chuỗi `--thr` đã dùng khi merge) | YOLO26 bắt person / motorbike / car tốt, box khít (`r01`); ngưỡng 0.25 nhiều box trùng, 0.5 mất người / xe nhỏ ở xa (`r02`), nên cần ngưỡng theo lớp |
| bicycle / bus / truck | YOLO26m | `--conf` mặc định 0.3 | < 5 vật trong gold (bicycle 3, bus 1, truck 0): không chọn ngưỡng riêng (`qc/threshold_sweep.md`) |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt `--classes three_wheeler` (cache `r06`) | `--conf` mặc định 0.3 (không có trong chuỗi `--thr`) | YOLO26 không có lớp này: gọi tuk-tuk là car / truck hoặc bỏ sót (`r01`, `r02`); YOLOE tìm được tuk-tuk / xích lô nhưng có box sai và box lỏng (`r03`) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  - **Ghi nhận sai sót:** chuỗi `--thr` tôi dán là chuỗi **ví dụ** trong đề, không phải chuỗi do `threshold_sweep` gợi ý cho dữ liệu của tôi (`motorbike=0.1,three_wheeler=0.15`). Hệ quả đo được trên gold (`qc/threshold_sweep.csv`): motorbike ở 0.5 chỉ đạt recall 9/14 = 64% (so với 13/14 = 93% ở 0.1), three_wheeler ở 0.3 đạt recall 5/7 = 71% (so với 7/7 ở 0.15). Điều này giải thích một phần vì sao lúc duyệt phải **vẽ thêm 21 motorbike** (mục 2). Lần sau phải chép đúng chuỗi từ `threshold_sweep.md`.
- **Phiên bản SAM tôi chọn:** **SAM 2.1-b** (`sam2.1_b.pt`). So với MobileSAM trên cùng tuk-tuk img_013 (`r07`, `r08`): MobileSAM nhanh hơn (~4.8 s so với ~8.3 s mỗi lần chạy trên CPU) nhưng mask vỡ 2 mảnh và thủng ở sàn xe; SAM 2.1-b cho mask liền khối, bám biên mui / bánh mượt hơn, và khi thêm box + đủ điểm âm thì tách người lái sạch nhất (`r11`). Máy chỉ có CPU nên không chạy SAM cho cả lô, chỉ dùng cho vài vật cần mask. Không dùng SAM 3 (cần xin quyền truy cập).
- Lỗi **không sửa được bằng đổi ngưỡng:** YOLO26 (model COCO) **không có lớp xe ba bánh**, nên tuk-tuk bị gọi `car` (img_013 0.42, img_004 0.34–0.37) hoặc `truck` (img_011 0.77), và tuk-tuk đỏ lớn ở img_004 bị bỏ sót hoàn toàn. Hạ hay tăng `--conf` (`r01` → `r02`) không đổi được lớp. Cách sửa: dùng **model thứ hai** (YOLOE với prompt `three_wheeler`) cho lớp này, rồi người duyệt đổi lớp / vẽ tay phần còn sót. Tương tự, person / car không đạt recall 90% ở bất kỳ ngưỡng nào (người bị che, ở xa): phải vẽ tay khi duyệt.
- Box **điểm cao mà sai:** YOLOE gán `three_wheeler` 0.63 cho một thúng của gánh hàng rong ở img_014; YOLO26 gọi tuk-tuk là `truck` 0.77 ở img_011; bẫy box ma `motorbike` 0.61 trên tán cây ở img_014. **Điểm thấp mà đúng:** xích lô `three_wheeler` 0.27 ở img_032 (`r03`); person 0.26–0.36 bị che ở img_002 bị mất khi tăng ngưỡng lên 0.5 (`r02`). Vì vậy `conf` chỉ dùng để sắp thứ tự duyệt, không dùng để tự chấp nhận box (CAL-09).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng được chọn và chấm trên **cùng 4 ảnh gold** (`qc/threshold_sweep.md`), nên precision/recall ở ngưỡng đã chọn là con số **lạc quan**; với ít ảnh như vậy ngưỡng chỉ là ước lượng thô. Ngưỡng gợi ý (nhưng **không** được dùng khi merge, xem ghi nhận sai sót ở trên): `--thr "motorbike=0.1,three_wheeler=0.15"` (motorbike recall 13/14 = 93%, precision 13/19 = 68%; three_wheeler recall 7/7 = 100%, precision 7/9 = 78%). **Không ép ngưỡng riêng** cho bicycle (3 vật), bus (1 vật), truck (0 vật): dùng `--conf` mặc định 0.3. three_wheeler chỉ có 7 vật nên ngưỡng 0.15 vẫn là kết luận yếu và cần người duyệt để loại box sai (bán tải / xe tay ga bị gọi three_wheeler ở img_002, xem `r03`). person (recall tối đa 20/25 = 80%) và car (recall 7/8 = 87.5% ở mọi ngưỡng 0.1–0.85) **không đạt recall 90%** ở bất kỳ ngưỡng nào ≤ 0.5: đây là lỗi bỏ sót của model (người bị che / ở xa), không sửa được bằng đổi ngưỡng, phải vẽ tay bổ sung khi duyệt.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 392 | 392/433 = 90.5% | 433 box máy |
| edited | 18 | 18/433 = 4.2% | 433 box máy |
| deleted | 22 | 22/433 = 5.1% | 433 box máy |
| added | 45 | 45/455 = 9.9% | 455 đối tượng sau duyệt |

Nguồn: `qc/review_stats.md` (24 ảnh batch, accepted = IoU ≥ 0.9; đã loại 26 đối tượng quanh bẫy). Trong deleted + added có 4 ca là **đổi lớp** (box đúng chỗ, sai lớp), nên số vật máy **sót thật** ≈ 45 − 4 = 41/455 ≈ 9%.

| Lớp | accepted | edited | deleted | added | accepted / box máy của lớp |
|---|---|---|---|---|---|
| person | 246 | 4 | 3 | 5 | 246/253 = 97% |
| motorbike | 83 | 5 | 1 | 21 | 83/89 = 93% |
| car | 44 | 0 | 4 | 11 | 44/48 = 92% |
| three_wheeler | 9 | 8 | 9 | 4 | 9/26 = **35%** |
| bicycle / bus / truck | 4 / 3 / 3 | 1 / 0 / 0 | 3 / 0 / 2 | 1 / 0 / 3 | quá ít vật để kết luận |

- **Lớp máy tệ nhất: `three_wheeler`.** Chỉ 9/26 box máy được giữ nguyên, 8 phải sửa và 9 phải xoá. Lỗi chủ yếu đến từ YOLOE: (a) box trùng chồng nhau trên cùng một xe (ví dụ img_003: một box ôm cả xe ba gác, một box chỉ phủ nửa dưới); (b) box lỏng kéo dài ra nền (img_011 ở bước tour model); (c) gọi nhầm xe khác là xe ba bánh (bán tải / xe tay ga ở img_002, `r03`). Đây là lỗi về **chất lượng box và nhận dạng** của YOLOE chứ không do ngưỡng quá thấp: three_wheeler được merge ở ngưỡng mặc định 0.3, và vẫn có box sai với conf 0.33–0.63 (thúng gánh hàng img_014).
- **Máy bỏ sót nhiều nhất ở `motorbike` (+21) và `car` (+11)**, tập trung ở các ảnh xe máy dày đặc: xe bị người ngồi trên che gần hết, chỉ thấy bánh / ống xả (ví dụ img_005: thêm 3 xe máy bị che và 2 xe van ở cuối đường). Với motorbike, một phần là do merge ở ngưỡng 0.5 thay vì 0.1 như sweep gợi ý (mục 1). `person` gần như không phải sửa (97% accepted).
- **So với gold** (`qc/review_stats.md` mục 2, 4 ảnh gold, **mẫu số 58 đối tượng gold**): máy có precision 42/61 = 68.9% (KTC 95%: 56–79%), recall 42/58 = 72.4% (KTC 95%: 60–82%). Con số này **lạc quan** (ngưỡng được chọn trên chính 4 ảnh này) và khoảng tin cậy rất rộng.
- *Ghi chú:* img_005 có một phần chỉnh sửa do Claude thực hiện qua API của CVAT (xem mục 8), nên số edited/added của ảnh này không hoàn toàn là của tôi.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **9/12 = 75%** (KTC 95%: 47–91%), **dưới ngưỡng đạt 80%**. Theo loại: missing 3/3, wrong_class 3/3, loose 2/3, **extra 1/3**. Bẫy lọt: img_014 (extra), img_018 (extra), img_013 (loose).
- Vì sao lọt (đối chiếu `prelabel/traps_key.json`, mở sau khi export):
  - **img_014 (extra):** một box `motorbike` giả nằm trên **tán cây / nền trời** ở góc trên bên phải, nơi không có xe nào. **img_018 (extra):** một box `motorbike` giả nằm trên **biển hiệu ngân hàng (VIB)** ở mặt tiền toà nhà. Cả hai đều nằm **ngoài mặt đường**. Ở bước 1–2 tôi chỉ quét và đếm vùng đường, nơi có xe và người, nên bỏ qua các box ở vùng "không thể có xe" phía trên ảnh. Tôi bắt hết bẫy *thiếu vật* (3/3) và *sai lớp* (3/3) vì hai lỗi đó nằm ngay trong vùng tôi đang soi.
  - **img_013 (loose):** box `truck` (xe taxi van phía sau chiếc tuk-tuk, góc trên bên phải) bị nới rộng khoảng 25 px lên trên và 14 px sang trái. Ảnh này có nhiều box chồng nhau trên chiếc tuk-tuk ở tiền cảnh; tôi tập trung sửa tuk-tuk và người ngồi trong xe nên không kiểm đủ 4 cạnh của box xe nhỏ ở hậu cảnh (bỏ bước 3).
- Accepted cao (90.5%) mà bắt bẫy chỉ 75% cho thấy **thiên kiến tự động hoá**: khi máy đã vẽ box, tôi có xu hướng mặc định giữ lại; lỗi "thừa" (cần chủ động xoá) lọt nhiều hơn lỗi "thiếu" (dễ thấy vì có chỗ trống). Con số accepted cao vì vậy không chứng minh máy tốt đến 90%: một phần là do tôi chấp nhận quá dễ. Chỉ có 12 bẫy nên khoảng tin cậy rộng; đọc cùng danh sách bẫy lọt chứ không chỉ con số %.
- Kết quả trên là **lần duyệt đầu** (`labels/batch_reviewed.json`). Các bẫy lọt và 4 box < 20 px mà `validate_labels.py` cảnh báo (img_018 #152, #167; img_023 #252; img_027 #335) sẽ được sửa trong vòng QC Part 6 (`labels/qc_fixed.json`), không ghi đè số đo lần đầu.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **29 phút / 4 ảnh ≈ 7.3 phút/ảnh** (gold) vs duyệt **76 phút / 24 ảnh ≈ 3.2 phút/ảnh** (batch_review; img_021 không ghi phút) → duyệt nhanh hơn khoảng **2.3 lần**. Ảnh dày tốn lâu nhất (img_005: 10 phút, img_003: 8 phút). Thiên lệch: tôi gán gold trước nên đã quen kiểu cảnh và guideline khi duyệt; ảnh gold và ảnh batch cũng khác nhau, nên tỉ lệ 2.3× chỉ là ước lượng, có thể đang thổi phồng lợi ích của máy.
- Kết luận: **có, nhưng theo lớp.**
  - Dùng máy cho `person`, `motorbike`, `car`: accepted 92–97%, chỉ phải vẽ thêm xe bị che ở ảnh dày.
  - **Không tin máy cho `three_wheeler`**: accepted 9/26 = 35%, gần 2/3 box phải sửa hoặc xoá. Với lớp này, box máy chỉ nên là gợi ý vị trí, mỗi box phải kiểm tra đầy đủ (hoặc vẽ tay).
  - Lợi ích về tốc độ chỉ có giá trị nếu người duyệt vẫn bắt được lỗi: bắt bẫy 75% < 80% cho thấy duyệt nhanh đã kéo theo lọt box ma.
- **Ba quy tắc giảm thiên kiến của tôi** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Đếm trước, nhìn box sau:** trước khi bật nhãn máy, ghi số vật đếm được theo lớp; sau đó so với số box. Lệch ≥ 1 thì phải tìm ra vật lệch. *Kiểm:* mỗi ảnh có số đếm trước ghi lại; lô sau bắt bẫy missing vẫn đạt 3/3.
  2. **Duyệt theo danh sách box, không theo vùng mắt nhìn:** đi lần lượt từng box trong panel Objects (từ trên xuống), kể cả box nằm ngoài mặt đường (trên trời, mặt tiền nhà, biển hiệu). Mỗi box phải được quyết định giữ / sửa / xoá; box có conf < 0.4 và **mọi** box `three_wheeler` phải kiểm đủ 4 cạnh. *Kiểm:* bẫy extra và loose đạt 3/3 ở lô sau; số box đã bấm chọn = Items của ảnh.
  3. **Accepted quá cao thì duyệt lại mẫu:** nếu tỉ lệ accepted của lô > 85%, rút ngẫu nhiên 5 ảnh, duyệt lại từ đầu bằng Filters theo từng lớp (`Label == ...`), và ghi số lỗi tìm thêm. *Kiểm:* có dòng duyệt lại trong `time_log.csv` và số lỗi tìm thêm được ghi trong báo cáo.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái) | SAM 2.1-b (`sam2.1_b.pt`) | 2 dương + 1 âm (áo người lái) | **Có**: 1 điểm âm chưa đủ; mask vẫn dính mặt, tay, quần, chân người lái và cả người ngồi ở quán phía sau (nhìn xuyên qua cabin) | Viền mask lốm đốm ở áo người lái; 45.4% ảnh (to hơn xe thật) | `tuktuk_img013_sam2.jpg` |
| img_013 (cùng vật, đối chiếu) | MobileSAM (`mobile_sam.pt`) | 2 dương + 1 âm (cùng toạ độ) | **Ít hơn**: tách được áo và quần người lái nhưng vẫn dính mặt và bàn chân; người phía sau phần lớn đã tách | Mask vỡ thành 2 polygon; sàn xe và vùng cabin đỏ bị thủng lỗ; 35.2% ảnh | `tuktuk_img013_sam1.jpg` |
| img_013 (làm lại) | SAM 2.1-b | 2 dương + **3 âm** (áo, quần, mặt) + **box** bao xe | **Không**: tách sạch người lái (mặt, áo, quần, chân) và người phía sau; giữ đủ mui, thân, ghế, bánh | Còn vài lỗ nhỏ ở lưng ghế sau; 30.5% ảnh | `tuktuk_img013_sam2_fix.jpg` |
| img_004 (tuk-tuk đỏ lớn, YOLO26 bỏ sót) | SAM 2.1-b | 2 dương + 1 âm (người đứng giữa hai xe) | Không có người lái; túi đồ trong cabin nằm trong mask (thuộc xe theo 4.1.5) | Mép dưới gầm xe lốm đốm, ăn một ít bóng tối dưới gầm (trái 4.2.3) | `tuktuk_img004_sam2.jpg` |
| img_017 (tuk-tuk hồng đỗ bên đường) | SAM 2.1-b | 2 dương + 1 âm (đặt trên **mui xe**) | Không có người lái / hàng hoá | Điểm âm đặt nhầm lên chính xe nên vô tác dụng (mui vẫn trong mask); nền nhìn xuyên qua cabin hở bị gộp vào mask | `tuktuk_img017_sam2.jpg` |

- **1 điểm âm có đủ tách người lái không?** **Không**, với SAM 2.1 ở img_013. Thêm điểm âm theo từng bước cho thấy SAM chỉ dùng điểm thì rất khó cân bằng: 2 điểm âm (áo + quần) thì quần đã tách nhưng mặt, tay, chân vẫn dính; **3 điểm âm không có box** thì tách được người lái nhưng mask **sụp** chỉ còn mui và viền thân xe (14% ảnh, mất ghế và bánh). Cách ổn định là **3 điểm âm + một box gợi ý bao xe**: box giữ cho SAM biết phạm vi vật, điểm âm cắt người ra.
- **SAM 2.1 vs MobileSAM** (cùng 3 click trên img_013, CPU, gồm cả nạp model): MobileSAM ~4.8 s, SAM 2.1-b ~8.3 s. Với 1 điểm âm, MobileSAM tách người lái tốt hơn nhưng mask bị vỡ lỗ ở sàn và cabin; SAM 2.1 cho mask liền khối, bám biên mui và bánh xe mượt hơn nhưng "nuốt" người. Khi đã thêm box + đủ điểm âm, SAM 2.1-b cho kết quả sạch nhất → **chọn SAM 2.1-b**, chấp nhận chậm hơn khoảng 1.7×.
- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - **Giúp:** 4.2.4 (hai vật chạm nhau thì mỗi vật một polygon, không chồng) cho câu trả lời rõ: người lái là `person` riêng nên phải cắt khỏi mask xe. 4.2.2 (lỗ lớn thì bỏ ra) giúp quyết định loại phần người ngồi ở quán nhìn thấy xuyên qua cabin hở ở img_013.
  - **Thiếu:** (1) Không nói phần **thân người lái bị khung xe che một nửa** thuộc về ai khi nhìn xuyên qua (vùng chạm thuộc "vật phía trước", nhưng người lái ngồi *trong* xe, không rõ ai ở trước). (2) Không quy định **nền nhìn xuyên qua cabin hở** (img_017) có tính là lỗ hay không khi nó nằm giữa mui và thân. (3) Không có hướng dẫn **cách click** (ví dụ "bắt buộc đặt điểm âm lên từng phần người lái; nếu mask sụp thì thêm box"): img_017 cho thấy điểm âm đặt nhầm lên chính xe là vô tác dụng. Ba điểm này nên đưa vào `guideline_v1.1_calibrated.md`.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: **không làm** (phần tuỳ chọn); chỉ dùng `sam2_points.py` offline vì máy chỉ có CPU.

## 6. Calibration (Part 6)

- Số quyết định: **10** (`qc/calibration_decisions.csv`: 4 `loi_may`, 5 `cau_hoi_mo`, 1 `bay_lot`).
  - Quan trọng nhất: **CAL-01** (YOLO26 gọi tuk-tuk là car / truck → luôn `three_wheeler`) và **CAL-02** (YOLOE gán `three_wheeler` cho thúng gánh hàng, bán tải, xe tay ga → xoá / đổi lớp). Hai ca này giải thích phần lớn vì sao `three_wheeler` chỉ đạt accepted 9/26 = 35%.
  - **CAL-06** cũng quan trọng vì nó nối detection với mask: người lái là box `person` riêng và phải nằm ngoài mask xe (img_013, Part 5).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  - Chốt câu hỏi mở 2 (xe máy dày gần camera: gán từng xe, `ignore_region` chỉ cho cụm nhỏ ở xa), 3 (hàng chất cao không tính vào box xe), 4 (người trong xe ba bánh), 5 (kính thuộc xe, cabin hở là lỗ), 7 (không đặt ngưỡng conf tự chấp nhận). Câu 1 và 6 **chưa chốt** vì lô 28 ảnh không có ảnh ví dụ (không có xe đạp điện, không có ảnh đêm).
  - Thêm "Không gồm" cho `three_wheeler` (gánh hàng, thúng, xe đẩy), quy tắc box trùng (mục 4.1 quy tắc 8), phân biệt bicycle / motorbike (4.3), quy trình click SAM (4.2).
- Nguyên nhân gốc:
  - **Máy** (CAL-01 → CAL-04): YOLO26 không có lớp xe ba bánh; YOLOE nhận nhầm vật có hình dạng "thùng có bánh / có khung" thành xe ba bánh và sinh box trùng. Không cần đổi guideline, chỉ cần quy tắc rõ để người duyệt sửa nhanh.
  - **Guideline mơ hồ** (CAL-05 → CAL-09): các câu hỏi mở ở mục 6 của v1.0, đã chốt bằng ảnh ví dụ trong v1.1.
  - **Người duyệt** (CAL-10): bẫy box ma lọt vì tôi chỉ quét vùng mặt đường. Đây không phải lỗi guideline, nên chỉ thêm vào checklist duyệt.

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** (1 ảnh `can_review` là img_005 + 5 ảnh ngẫu nhiên, seed 42: img_003, img_008, img_014, img_017, img_033).
  - Người kiểm: phiếu QC (`qc_review.csv`) do **Claude** soi ảnh và điền, tôi xem lại và đồng ý (cột `reviewer = claude`, xem mục 8). **Hạn chế:** người kiểm không độc lập với người duyệt: cùng một quy trình, cùng dùng AI, nên có thể cùng bỏ sót một kiểu lỗi.
  - Làm **hai vòng** QC trên cùng 6 ảnh (seed 42): vòng 1 trên `batch_reviewed.json`, sửa trong CVAT → `labels/qc_fixed.json` → `build_release.py` → vòng 2.

| Vòng | Lỗi theo **đối tượng** | Ảnh có lỗi | Cận trên 95% (ảnh lỗi) | Nguồn |
|---|---|---|---|---|
| 1 | 12/108 = 11.1% | 6/6 = 100% | 100% | `qc/qc_review_round1.csv`, `qc/qc_report_auto_round1.md` |
| 2 | **3/107 = 2.8%** | **2/6 = 33.3%** | **70%** | `qc/qc_review.csv`, `qc/qc_report_auto.md` |

  Ngưỡng Release: ảnh lỗi ≤ 2%. Cả hai vòng đều vượt.
- Top lỗi (vòng 1, 12 lỗi), mỗi loại một ảnh ví dụ:
  - **Bỏ sót** 6/12: person trên xe máy và xe đạp ở img_033; người đứng sau xe chở chuối ở img_008; tuk-tuk mui bạc ở img_017; thiếu `ignore_region` ở img_005.
  - **Thừa** 4/12: hai thúng gánh hàng bị gán `three_wheeler` và box ma trên tán cây ở img_014; box person trùng ở img_003.
  - **Sai vị trí / kích thước** 1/12: tuk-tuk lỏng cạnh trái ~47 px ở img_017.
  - **Sai lớp** 1/12: xe đạp gán `motorbike` ở img_033.
  - Không có lỗi sai thuộc tính hay sai mask trong mẫu.
  - Vòng 2 còn 3 lỗi, đều là **bỏ sót**: 2 người trên xe máy #400 ở img_033, `ignore_region` ở img_005.
- **Giới hạn mẫu nhỏ:** với 2 ảnh lỗi trên 6 ảnh, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **70%**. Ngay cả khi vòng sau có 0/6 ảnh lỗi thì cận trên vẫn khoảng 39%, nên ngưỡng 2% **không thể được chứng minh** với mẫu cỡ này; muốn kết luận phải kiểm mẫu lớn hơn nhiều (gần như cả lô 24 ảnh).
- Script kiểm được / không kiểm được:
  - **Kiểm được:** định dạng COCO, lớp hợp lệ, box cao < 20 px (`validate_labels.py` đã cảnh báo 4 box), số đối tượng, tỉ lệ accepted / edited / deleted / added, tỉ lệ bắt bẫy, cận trên thống kê.
  - **Không kiểm được:** box có ôm sát hay không (khi không có gold); vật **cả máy và người cùng bỏ sót** (chỉ thấy khi soi ảnh, như 2 người trên xe máy ở img_033); vật không phải phương tiện bị gán lớp phương tiện (thúng gánh hàng); người duyệt có thật sự làm đủ checklist 4 bước hay không.

**Quyết định: REWORK**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release):
- **Không Hold:** guideline đã đủ rõ để sửa (v1.1 chốt 5/7 câu hỏi mở; câu 1 và 6 không gặp trong lô); dữ liệu không hỏng (`validate_labels.py --all` OK, 24 ảnh).
- **Rework:**
  - Ảnh có lỗi 2/6 = 33.3% > 2% ngay cả sau một vòng sửa.
  - Lỗi lặp có hệ thống: **bỏ sót** chiếm 6/12 lỗi vòng 1 và 3/3 lỗi vòng 2, tập trung ở người ngồi trên xe nhỏ ở xa.
  - Bắt bẫy 9/12 = 75% < 80% (lọt 2/3 bẫy box ma), nên chưa tin được rằng phần còn lại của lô (18 ảnh ngoài mẫu) đã sạch.
- **Không Release:** số liệu không đạt bất kỳ ngưỡng nào.

Việc cần làm tiếp:
1. Thêm 2 person trên xe máy #400 ở img_033. Quyết định `ignore_region` cho cụm ở xa ở img_005 theo quy tắc CAL-05.
2. **Duyệt lại cả 18 ảnh ngoài mẫu** theo guideline v1.1 và checklist mới: đi lần lượt từng box trong panel Objects, kể cả box ngoài mặt đường; kiểm mọi box `three_wheeler` có đủ 3 bánh; tìm người ngồi trên mọi xe máy / xe đạp.
3. Merge lại pre-label với đúng chuỗi ngưỡng từ sweep (`motorbike=0.1,three_wheeler=0.15`) cho lô sau, để giảm số motorbike phải vẽ thêm.
4. Export `qc_fixed.json`, chạy lại `build_release.py`, rút mẫu QC **lớn hơn** (ví dụ `--n 10`, seed khác) và ghi vòng 3 vào báo cáo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** có, dùng **Claude Code** (Claude Opus) trong suốt Part 2–6:
  - **Part 2–3:** xem ảnh `viz/` và viết nhận xét cho `model_log.csv` (r01–r04); đọc `threshold_sweep` và viết đoạn giới hạn ở mục 1.
  - **Part 4:** hướng dẫn dùng Filters trong CVAT. **Claude trực tiếp sửa nhãn img_005** qua API của CVAT local: xoá 1 box trùng, kéo sát 2 box, thêm 5 xe. Vì vậy số edited / added của ảnh này không hoàn toàn là của tôi. 23 ảnh còn lại tôi tự duyệt; thời gian ở `time_log.csv` là thời gian thật của tôi.
  - **Part 4 (báo cáo):** chạy `validate_labels.py`, viết nháp mục 2–4 từ `review_stats.md`, đối chiếu `traps_key.json` (sau khi đã export) để mô tả bẫy lọt.
  - **Part 5:** Claude kiểm tra mask, chạy thêm SAM với nhiều điểm âm + box cho img_013 (`r11`), điền mục 5 và các dòng r07–r11.
  - **Part 6:** Claude soi 6 ảnh QC và điền `qc_review.csv` (cả hai vòng), viết `calibration_decisions.csv`, `guideline_v1.1_calibrated.md`, mục 6–8.
  - **Cách kiểm tra:** tôi đối chiếu từng nhận xét với ảnh trong CVAT, tự sửa các lỗi QC trong CVAT (export `qc_fixed.json`), chạy lại script để xác nhận số liệu. Claude cũng phát hiện ra tôi đã dán nhầm chuỗi `--thr` ví dụ khi merge (mục 1).
- Gold có bị nhiễm không? **Không.** `gold_manual.json` được gán tay xong (29 phút / 4 ảnh) trước khi chạy pre-label và trước khi dùng Claude; Claude không đọc hay sửa nhãn gold.
- Máy / GPU / CVAT: Windows 11, **chỉ CPU** (mọi lần chạy `prelabel.py` / SAM ở `device = cpu`); CVAT chạy bằng Docker trên `localhost:8080`. Vướng mắc: chạy script sai thư mục (đứng ở `day08\` thay vì thư mục repo) nên báo không tìm thấy file; Python hệ thống không có numpy, phải kích hoạt `.venv` của repo.
- Dùng cache `data/cache/`? **Không**, tự chạy `prelabel.py` cho cả 36 ảnh ở `--conf 0.05` (`r05` YOLO26m 1.94 s/ảnh, `r06` YOLOE 0.37 s/ảnh trên CPU).
- Lỗi / vướng mắc khi dùng script và công cụ:
  - `qc_sample.py --force` **ghi đè phiếu `qc_review.csv` đã điền** của vòng trước (phải lưu lại thành `qc_review_round1.csv`).
  - Trong CVAT, thuộc tính `conf` là kiểu chữ nên Filters không có toán tử `<`. "Starts with" trên thuộc tính bị bỏ qua mà không báo lỗi; chỉ `==` chạy được. Không lọc được `conf < 0.4` như tài liệu gợi ý.
- Một điều tôi làm khác lần sau:
  - Chép **đúng** chuỗi `--thr` từ `threshold_sweep.md` (lần này dán nhầm chuỗi ví dụ, làm motorbike recall trên gold chỉ còn 9/14).
  - Duyệt theo **danh sách box** thay vì theo vùng mắt nhìn, để không lọt box ma ngoài mặt đường.
