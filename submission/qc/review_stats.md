# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 505; đối tượng sau duyệt: 500; ngưỡng accepted: IoU >= 0.9
- Đã **loại 19 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 462 | 462/505 = 91.5% | 505 box máy |
| edited (sửa) | 15 | 15/505 = 3.0% | 505 box máy |
| deleted (xoá) | 28 | 28/505 = 5.5% | 505 box máy |
| added (vẽ thêm) | 23 | 23/500 = 4.6% | 500 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 16.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 264 | 3 | 4 | 0 |
| bicycle | 7 | 0 | 1 | 1 |
| motorbike | 132 | 4 | 0 | 6 |
| car | 36 | 0 | 2 | 10 |
| bus | 3 | 0 | 0 | 3 |
| truck | 3 | 0 | 3 | 2 |
| three_wheeler | 17 | 8 | 18 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 105 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 61 | 10 | 44 | 85.9% (KTC 95%: 76–92%) | 58.1% (KTC 95%: 49–67%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 4/12 = 33.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 0/3
  - extra: 3/3
  - wrong_class: 1/3
  - missing: 0/3
  - lọt: img_026.jpg (loose), img_025.jpg (wrong_class), img_022.jpg (missing), img_035.jpg (missing), img_013.jpg (loose), img_034.jpg (wrong_class), img_029.jpg (missing), img_033.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
