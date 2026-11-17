# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603202 - Nguyễn Thanh Thân

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ / chạy dày đặc **gần camera**: từng xe hay `ignore_region`? | Gán **từng xe** khi còn tách được ranh giới và còn thấy ≥ 20% xe. `ignore_region` chỉ dùng cho cụm ≥ 5 vật cao < 20 px không tách được (giữ mục 4.1.6). | img_006, img_021, img_028 | CAL-02 |

Các câu hỏi mở 1, 3, 4, 5, 6, 7: **chưa chốt** vì trong lô này chưa gặp ca đủ rõ để làm ví dụ.

## 2. Ca khó mới (từ lỗi của máy, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_004, img_007, img_032 | Xe ba bánh có cabin / mui kín trông giống ô tô | YOLO26 gán `car` hoặc không nhận ra (r01, r04); YOLOE sót ở img_007 (r03) | Gán `three_wheeler` theo **số bánh** (3 bánh), không theo hình dáng cabin | Mục 2 (`three_wheeler`) và 4.3: xe 3 bánh có cabin kín vẫn là `three_wheeler`. Khi review phải **tự quét riêng xe ba bánh** vì cả hai model đều sót (CAL-01) |
| img_001 | Người mờ, không nhận rõ đặc điểm con người | — (gold gán tay, tick `can_review` 2 box) | Gán `person` khi nhận ra hình dạng người (đầu + thân) và thấy ≥ 20%; không xác định được là người thì **không gán** | Mục 2 (`person`) và 4.3 (CAL-03) |
| img_013 | Mask tuk-tuk có người lái ngồi chồng lên xe | SAM 2.1: 3 click thì mask gồm cả người lái; 8 click thì bỏ được người nhưng xe khuyết | Mask `three_wheeler` **không gồm** người lái / hành khách | Mục 4.2.4: đặt click dương ở đầu, đuôi, bánh, mui xe trước, sau đó click âm lên người (CAL-04) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở số 2 (xe máy dày gần camera); thêm quy tắc xe 3 bánh cabin kín, người mờ, mask xe có người lái | CAL-01, CAL-02, CAL-03, CAL-04 |
