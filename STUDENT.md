# Thông tin học viên

- Họ và tên: Nguyễn Thanh Thân
- MSHV: 2A202603202
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: thannt97
- Repo bài nộp (URL): https://github.com/thannt97/K5-L23-DAY08-2A202603202-NguyenThanhThan-AIPrelabel
- Commit chốt bài (hash): <<>>
- Máy: Windows, Python 3.13.17, torch 2.15.0+cpu, không có GPU (chạy CPU)
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 462 / 15 / 28 = 91.5% / 3.0% / 5.5% (mẫu số: 505 box máy); added: 23 = 4.6% (mẫu số: 500 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 61/71 = 85.9% / 61/105 = 58.1%
- Bắt bẫy: 4/12 = 33.3%
- Thời gian: gán tay 38/4 = 9.5 phút/ảnh; duyệt có pre-label 25/24 ≈ 1.0 phút/ảnh
- Lỗi QC: 40.0% theo ảnh (2/5), 2.9% theo đối tượng (3/102)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi QC 2/5 = 40% > 2% và bắt bẫy 4/12 = 33% < 80%, lô duyệt chưa đáng tin.
