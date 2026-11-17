# Báo cáo Day 08 — 2A202603202 - Nguyễn Thanh Thân

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | car = 0.5; các lớp còn lại 0.3 (mặc định) | r01: YOLO26 nhận đủ person / motorbike / bicycle / car; r02: một ngưỡng chung 0.5 làm sót 76 box thật (img_007 sót 6 motorbike, img_015 sót 6 person) nên chọn ngưỡng theo lớp. `threshold_sweep`: car đạt recall 90% (9/10) ở 0.5 với precision cao nhất (69%); person / motorbike / bicycle không ngưỡng nào ≤ 0.5 đạt recall 90% nên giữ mặc định 0.3 |
| three_wheeler | YOLOE-26s-seg, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | r01: YOLO26 không có lớp xe ba bánh, gọi tuk-tuk là car hoặc bỏ qua (img_007, img_032); r03: YOLOE gõ chữ tìm được xe ba bánh (img_002, img_011, img_032). `threshold_sweep`: 0.15 cho recall 7/7 = 100%, precision 78% trên gold |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (589 box)
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`). Máy không có GPU (chạy CPU, ~5.3 s/ảnh ở r04); SAM 3 là model gated ~3.5 GB, cần GPU, và `transformers` trên máy bị Application Control chặn. Kết quả: mask liền khối khi click đúng vật (img_004: 13.2% ảnh, 1 polygon, 2 click), nhưng với xe có người lái ngồi chồng (img_013) phải tới 12 click và vẫn ra 2 polygon.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Xe ba bánh bị YOLO26 gọi `car` (img_004, r04) hoặc không nhận ra (img_007, img_032, r01); tăng ngưỡng 0.25 → 0.5 (r02) không sửa được mà còn sót thêm 76 box thật. Sửa bằng YOLOE prompt chữ cho `three_wheeler` (r03), nhưng YOLOE vẫn sót tuk-tuk ở img_007.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai (đối chiếu `pre_traps` với `batch_reviewed`, đã bỏ box bẫy): img_024 car 0.81 bị xoá; img_003 three_wheeler 0.80 bị xoá và three_wheeler 0.61 đổi thành truck; img_013 three_wheeler 0.63 đổi thành bus. Điểm thấp mà đúng: 35 box conf < 0.35 được giữ nguyên (IoU ≥ 0.9), phần lớn là three_wheeler 0.16–0.29 (img_003, img_010, img_023, img_022, img_005, img_008, img_018). Điểm `conf` không phải xác suất đúng.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng chọn và chấm trên cùng 4 ảnh gold (105–107 vật). person / motorbike / bicycle không ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 78% / 71% / 60%) nên dùng mặc định 0.3; bus (0 vật) và truck (1 vật) quá ít mẫu nên không chọn ngưỡng riêng (`qc/threshold_sweep.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 462 | 462/505 = 91.5% | 505 box máy |
| edited | 15 | 15/505 = 3.0% | 505 box máy |
| deleted | 28 | 28/505 = 5.5% | 505 box máy |
| added | 23 (7 là đổi lớp → sót thật ≈ 16) | 23/500 = 4.6% | 500 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Số liệu: three_wheeler 17 accepted / 8 edited / 18 deleted trên 43 box máy; car 10 added; truck 3 deleted / 2 added. three_wheeler tệ nhất theo tỉ lệ xoá (18/43 = 42%): ngưỡng YOLOE 0.15 rất thấp (chọn để đạt recall 100% trên gold) nên sinh nhiều box thừa, và YOLOE còn nhầm lớp (img_013 three_wheeler → bus, img_003 → truck). car bị thêm 10 box vì YOLO26 lẫn xe ba bánh với car, và ngưỡng car 0.5 cao hơn mặc định.
- So với gold: precision / recall của **máy** 61/71 = 85.9% (KTC 95%: 76–92%) / 61/105 = 58.1% (KTC 95%: 49–67%) (mẫu số: 105 đối tượng gold, 4 ảnh)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 4/12 = 33.3% (KTC 95%: 14–61%); loại nào lọt: loose 0/3, missing 0/3, wrong_class 1/3 bắt được (extra 3/3 bắt được); ảnh lọt: img_013, img_026, img_033 (loose), img_022, img_029, img_035 (missing), img_025, img_034 (wrong_class); vì sao (bạn đã bỏ bước nào của checklist)? Tôi bỏ bước **ẩn nhãn máy, quét ảnh trước** để tìm vật bị sót, nên cả 3 bẫy missing đều lọt; tôi cũng không phóng to kiểm mép box nên 3 bẫy loose đều lọt. Bẫy extra bắt được 3/3 vì box thừa nổi rõ trên ảnh, còn lỗi thiếu box thì không có gì hiện ra để nhìn.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 462/505 = 91.5% nhưng bắt bẫy chỉ 4/12 = 33%, với tốc độ khoảng 1 phút/ảnh: phần lớn box được chấp nhận vì trông hợp lý chứ không phải vì đã kiểm. Đây là thiên kiến tự động hóa; accepted cao không có nghĩa máy tốt, vì recall của máy trên gold chỉ 61/105 = 58%.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 38/4 = 9.5 phút/ảnh vs duyệt 25/24 ≈ 1.0 phút/ảnh. Thiên lệch: ảnh gold và batch khác nhau nên không phải do quen cảnh, nhưng gán gold trước giúp tôi đã quen guideline và giao diện CVAT, nên tốc độ duyệt có phần nhanh hơn thực tế; đồng thời tốc độ 1 phút/ảnh là quá nhanh so với khuyến nghị ~3 phút/ảnh.
- Kết luận: **có, nhưng có điều kiện**, dựa trên thời gian giảm từ 9.5 xuống 1.0 phút/ảnh và accepted 91.5%; tuy vậy recall của máy trên gold chỉ 58% nên chỉ đáng dùng khi người duyệt vẫn chủ động tìm vật bị sót. Cẩn thận nhất với three_wheeler (18/43 box bị xoá) và car (10 box phải thêm).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Duyệt tối thiểu **3 phút/ảnh**, kiểm bằng `time_log.csv` (25 phút cho 24 ảnh là không đạt).
  2. **Trước khi bật nhãn máy**, đếm số vật mỗi lớp trong ảnh rồi so với số box máy; lệch thì tìm vật bị sót. Kiểm bằng bẫy missing (mục tiêu 3/3).
  3. Phóng to kiểm **từng box three_wheeler và car** (lớp và mép box); sau mỗi 10 ảnh kiểm lại ngẫu nhiên 1 ảnh. Mục tiêu lô sau: bắt bẫy ≥ 10/12, loose ≥ 2/3.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái) | SAM 2.1 | v1: 3 (2 pos + 1 neg); v2: 5; v3: 8 (2 pos + 6 neg); v4: 12 (6 pos + 6 neg) | v1 gồm người lái; v3 đã loại hết người | v1 45.4% ảnh gồm người lái; v3 13.4%, 2 polygon, thân xe khuyết nhiều; v4 34.0%, 2 polygon, lấy lại phần lớn thân xe (v2: chưa ghi kết quả) | tuktuk_img013.jpg, _v2, _v3, _v4 |
| img_004 (xe ba bánh) | SAM 2.1 | lần nhầm: 1 pos + 1 neg; lần đúng: 1 pos + 1 neg | chưa kiểm lại bằng mắt | lần nhầm: click dương rơi vào người đang đứng nên mask ra người (2.8% ảnh); lần đúng: 13.2% ảnh, 1 polygon | motorbike_img004.jpg (mask người, ví dụ lỗi), threewheeler_img004.jpg |
| img_008 (xe ba bánh) | SAM 2.1 | 1 pos | chưa kiểm lại bằng mắt | 0.9% ảnh, 1 polygon (chưa xác nhận đủ xe hay chỉ một phần; chỉ 1 click nên dễ chỉ lấy một bộ phận) | threewheeler_img008.jpg |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Mục 4.2.4 (hai vật chạm nhau mỗi vật một polygon, không chồng) giúp quyết định loại người lái khỏi mask tuk-tuk ở img_013. Mục 4.2 **chưa nói** phải làm gì khi tách người ra làm mask xe bị chia thành 2 polygon hoặc bị khuyết (img_013 v3, v4). SAM không biết quy tắc guideline: mask đi theo điểm click, không theo lớp (img_004: click nhầm vào người đứng nên mask ra người).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01, CAL-04 `loi_may`; CAL-02 `cau_hoi_mo`; CAL-03 `can_review`); quan trọng nhất (dẫn `CAL-xx`): **CAL-01** (xe ba bánh bị gọi car / bị sót) vì lỗi lặp ở nhiều ảnh (img_004, img_007, img_032), cả YOLO26 lẫn YOLOE đều mắc, và không sửa được bằng ngưỡng.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở số 2 (xe máy dày gần camera, CAL-02); xe 3 bánh cabin kín là `three_wheeler` + quét riêng xe ba bánh (CAL-01); người mờ (CAL-03); mask xe không gồm người lái (CAL-04).
- Nguyên nhân gốc: cả ba loại. **Máy**: tuk-tuk bị gọi car, ngưỡng YOLOE 0.15 sinh nhiều box thừa (CAL-01, CAL-04). **Guideline mơ hồ**: xe máy dày đặc gần camera, người mờ (CAL-02, CAL-03), mũ/ô chưa có quy tắc. **Người duyệt**: bẫy missing/loose lọt do duyệt quá nhanh, bỏ bước quét ảnh trước.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42: img_003, img_006, img_014, img_031, img_035; 102 đối tượng); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 3/102 = 2.9% · theo **ảnh**: 2/5 = 40.0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: thừa 2 (img_003: thừa nhãn three_wheeler; img_035: thừa nhãn person); sai vị trí/kích thước 1 (img_003: box lỏng).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 2/5 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 77%; với mẫu 5 ảnh không thể chứng minh ngưỡng 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script kiểm được tên file, lớp hợp lệ, box nằm trong ảnh, đủ ảnh, accepted/edited/deleted/added, bắt bẫy, tỉ lệ lỗi QC. Script **không** kiểm được box có ôm sát vật thật không, lớp có đúng thực tế không, vật mà cả tôi lẫn máy cùng bỏ sót (ngoài gold và bẫy), và tôi có thật sự theo checklist hay không.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đã chốt các ca mơ hồ vào v1.1 và dữ liệu không hỏng; nhưng ảnh lỗi 2/5 = 40% > ngưỡng 2% (cận trên 95%: 77%) và bắt bẫy 4/12 = 33% < 80%, nên lô duyệt chưa đáng tin → **Rework**.
Nếu Rework / Hold, việc cần làm tiếp: duyệt lại 24 ảnh batch theo 3 quy tắc ở mục 4 (quét ảnh trước, ≥ 3 phút/ảnh, phóng to kiểm three_wheeler và car); sửa lỗi QC ở img_003 và img_035; export `submission/labels/qc_fixed.json`; chạy lại `build_release.py` và `qc_sample.py --force`, ghi vòng sau vào báo cáo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude (Cowork) để hướng dẫn cài đặt/sửa lỗi môi trường, giải thích guideline, chạy các script (threshold_sweep, merge, inject_traps, review_stats, build_release, qc_sample), chép `runs.csv` sang `model_log.csv` và ghi lại quan sát của tôi, nháp `calibration_decisions.csv` / guideline v1.1 từ các quy tắc tôi đã áp dụng khi gán nhãn. Claude không xem ảnh. Tôi kiểm tra bằng cách đối chiếu với ảnh trong `viz/`, `masks/` và các file output. Phần phân tích và kết luận ở mục 3–8 (thiên kiến, 3 quy tắc, nguyên nhân gốc, quyết định Rework) do Claude gợi ý dựa trên số liệu của tôi; tôi đã đọc, đối chiếu với trải nghiệm duyệt thực tế và đồng ý.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không chạy model và không xem pre-label của ảnh gold trước khi export `gold_manual.json` (các lệnh model ở Part 2 chạy sau, trên ảnh pilot). Trong lúc gán gold có hỏi AI (Claude) về **quy tắc** guideline (motorbike có tính người ngồi không, người ngồi trên xe ba gác có gán không), nhưng không gửi ảnh và không hỏi nên gán gì cho ảnh cụ thể.
- **Ca khó khi gán gold (`can_review`):** `img_001`: tick `can_review` cho box `person` vì không nhận rõ đặc điểm con người (không chắc có phải người thật / đủ để gán `person` hay không).
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows, Python 3.13, torch CPU, không GPU, CVAT Docker. Application Control của Windows chặn `pip.exe` (dùng `python -m pip`) và file `regex/_regex.pyd` bản mới (cài lại `regex==2025.10.22` để YOLOE/CLIP và transformers chạy được); PyYAML phải cài bằng `--only-binary`.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy toàn bộ trên CPU.
- Lỗi / bug script đã gặp (nếu có): `work/runs.csv` ghi trùng `run_id` r04 cho 3 lần chạy (có thể do các lần chạy YOLOE bị lỗi trước đó); trong `model_log.csv` đánh lại r01–r04, tham số giữ nguyên.
- Một điều bạn làm khác lần sau: không duyệt nhanh hơn 3 phút/ảnh và luôn quét ảnh tìm vật bị sót trước khi xem nhãn máy.
