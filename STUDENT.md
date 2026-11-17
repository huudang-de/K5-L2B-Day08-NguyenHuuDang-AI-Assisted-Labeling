# Thông tin học viên

- Họ và tên: Nguyễn Hữu Đăng
- MSHV: 2A202603300
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: huudang-de
- Repo bài nộp (URL): https://github.com/huudang-de/K5-L2B-Day08-NguyenHuuDang-AI-Assisted-Labeling
- Commit chốt bài (hash): 6c52cf8 (Part 5–6; bản rà soát cuối trên nhánh main)
- Máy: Windows, không có GPU (CPU, RAM 24 GB)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 491 / 8 / 46 (mẫu số: 553 box máy); added: 28 (mẫu số: 528 đối tượng)
- So với gold: precision / recall của **máy** 58.1% (43/74) / 76.8% (43/56)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 4.25 phút/ảnh (17 phút / 4 ảnh); duyệt có pre-label 1.92 phút/ảnh (46 phút / 24 ảnh)
- Lỗi QC: 33.3% theo ảnh (2/6), 1.7% theo đối tượng (2/116)
- **Quyết định:** Rework — lý do một dòng: Ảnh có lỗi QC 2/6 (33.3% > 2%) và tỉ lệ bắt bẫy 8/12 (66.7% < 80%), cần rà lại lô theo guideline v1.1.
