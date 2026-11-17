# Báo cáo Day 08 — 2A202603202 - Nguyễn Thanh Thân

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | car = 0.5; các lớp còn lại 0.3 (mặc định) | <<vì sao, dẫn r01, r02>> |
| three_wheeler | YOLOE-26s-seg, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | <<vì sao, dẫn r01, r03>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (589 box)
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`). Máy không có GPU (chạy CPU, ~5.3 s/ảnh ở r04); SAM 3 là model gated ~3.5 GB, cần GPU, và `transformers` trên máy bị Application Control chặn. <<bổ sung nhận xét của bạn về chất lượng mask>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Xe ba bánh bị YOLO26 gọi `car` (img_004, r04) hoặc không nhận ra (img_007, img_032, r01); tăng ngưỡng 0.25 → 0.5 (r02) không sửa được mà còn sót thêm 76 box thật. Sửa bằng YOLOE prompt chữ cho `three_wheeler` (r03), nhưng YOLOE vẫn sót tuk-tuk ở img_007. <<bổ sung nếu cần>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng chọn và chấm trên cùng 4 ảnh gold (105–107 vật). person / motorbike / bicycle không ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 78% / 71% / 60%) nên dùng mặc định 0.3; bus (0 vật) và truck (1 vật) quá ít mẫu nên không chọn ngưỡng riêng (`qc/threshold_sweep.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 462 | 462/505 = 91.5% | 505 box máy |
| edited | 15 | 15/505 = 3.0% | 505 box máy |
| deleted | 28 | 28/505 = 5.5% | 505 box máy |
| added | 23 (7 là đổi lớp → sót thật ≈ 16) | 23/500 = 4.6% | 500 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Số liệu: three_wheeler 17 accepted / 8 edited / 18 deleted trên 43 box máy; car 10 added; truck 3 deleted / 2 added. <<nhận xét của bạn: vì sao, dẫn ảnh>>
- So với gold: precision / recall của **máy** 61/71 = 85.9% (KTC 95%: 76–92%) / 61/105 = 58.1% (KTC 95%: 49–67%) (mẫu số: 105 đối tượng gold, 4 ảnh)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 4/12 = 33.3% (KTC 95%: 14–61%); loại nào lọt: loose 0/3, missing 0/3, wrong_class 1/3 bắt được (extra 3/3 bắt được); ảnh lọt: img_013, img_026, img_033 (loose), img_022, img_029, img_035 (missing), img_025, img_034 (wrong_class); vì sao (bạn đã bỏ bước nào của checklist)? <<viết bằng lời bạn>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 38/4 = 9.5 phút/ảnh vs duyệt 25/24 ≈ 1.0 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái) | SAM 2.1 | v1: 3 (2 pos + 1 neg); v2: 5; v3: 8 (2 pos + 6 neg); v4: 12 (6 pos + 6 neg) | v1 gồm người lái; v3 đã loại hết người | v1 45.4% ảnh gồm người lái; v3 13.4%, 2 polygon, thân xe khuyết nhiều; v4 34.0%, 2 polygon <<nhận xét v2, v4>> | tuktuk_img013.jpg, _v2, _v3, _v4 |
| img_004 (xe ba bánh) | SAM 2.1 | lần nhầm: 1 pos + 1 neg; lần đúng: 1 pos + 1 neg | <<>> | lần nhầm: click dương rơi vào người đang đứng nên mask ra người (2.8% ảnh); lần đúng: 13.2% ảnh, 1 polygon | motorbike_img004.jpg (mask người, ví dụ lỗi), threewheeler_img004.jpg |
| img_008 (xe ba bánh) | SAM 2.1 | 1 pos | <<>> | 0.9% ảnh, 1 polygon <<đủ xe hay chỉ một phần?>> | threewheeler_img008.jpg |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01, CAL-04 `loi_may`; CAL-02 `cau_hoi_mo`; CAL-03 `can_review`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở số 2 (xe máy dày gần camera, CAL-02); xe 3 bánh cabin kín là `three_wheeler` + quét riêng xe ba bánh (CAL-01); người mờ (CAL-03); mask xe không gồm người lái (CAL-04).
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42: img_003, img_006, img_014, img_031, img_035; 102 đối tượng); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude (Cowork) để hướng dẫn cài đặt/sửa lỗi môi trường, giải thích guideline, chạy các script (threshold_sweep, merge, inject_traps, review_stats, build_release, qc_sample), chép `runs.csv` sang `model_log.csv` và ghi lại quan sát của tôi, nháp `calibration_decisions.csv` / guideline v1.1 từ các quy tắc tôi đã áp dụng khi gán nhãn. Claude không xem ảnh. Tôi kiểm tra bằng cách đối chiếu với ảnh trong `viz/`, `masks/` và các file output. <<bổ sung / sửa>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có: ghi trung thực (vd. có hỏi AI về quy tắc guideline như người ngồi trên xe, nhưng không gửi ảnh và không chạy model trên ảnh gold)>>
- **Ca khó khi gán gold (`can_review`):** `img_001`: tick `can_review` cho box `person` vì không nhận rõ đặc điểm con người (không chắc có phải người thật / đủ để gán `person` hay không).
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows, Python 3.13, torch CPU, không GPU, CVAT Docker. Application Control của Windows chặn `pip.exe` (dùng `python -m pip`) và file `regex/_regex.pyd` bản mới (cài lại `regex==2025.10.22` để YOLOE/CLIP và transformers chạy được); PyYAML phải cài bằng `--only-binary`.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy toàn bộ trên CPU.
- Lỗi / bug script đã gặp (nếu có): `work/runs.csv` ghi trùng `run_id` r04 cho 3 lần chạy (có thể do các lần chạy YOLOE bị lỗi trước đó); trong `model_log.csv` đánh lại r01–r04, tham số giữ nguyên.
- Một điều bạn làm khác lần sau: <<>>
