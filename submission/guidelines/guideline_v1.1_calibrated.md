# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603300 - Nguyễn Hữu Đăng

> **Chỉ viết phần thay đổi so với v1.0**. Chốt các câu hỏi mở đã gặp và bổ sung quy tắc từ lỗi máy, bẫy lọt và kiểm định QC. Mỗi ca khó đều gắn với ảnh ví dụ (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Nếu **có bàn đạp** (dáng xe đạp) → `bicycle`; nếu **không bàn đạp** (dáng xe tay ga) → `motorbike`. Nếu bị che phần bàn đạp, chọn lớp tin nhất và tick `can_review`. | `img_001.jpg`, `img_024.jpg` | `CAL-01` |
| 4 | Vật bị che khuất / xén mép (ngưỡng ≥ 20% diện tích, không rõ hình) | Giữ ngưỡng ≥ 20% diện tích nhìn thấy; nếu chỉ thấy chữ/mảnh nhỏ không nhận ra hình dáng xe (như ca `#11` khi QC) thì **không vẽ box**. | `img_003.jpg`, `img_019.jpg` | `CAL-03`, `CAL-04` |

*(Các câu hỏi mở còn lại như hàng nhô ra, kính chắn gió khi tách mask toàn lô sẽ chốt thêm ở vòng `v1.2` nếu mở rộng sang bài toán segmentation toàn tập).*

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_002.jpg`, `img_004.jpg`, `img_013.jpg` | Xe tuk-tuk / xích lô (`three_wheeler`) | `yolo26-m` bỏ sót hoặc gọi nhầm thành `car`/`motorbike`/`truck`; `sam2.1_b.pt` cắt mask giữ nguyên nhãn sai `car` | Không chỉnh `--conf` để sửa sai lớp; dùng `yoloe-s` (`--classes three_wheeler`) ưu tiên trước `yolo26-m` khi merge và kiểm lại bằng mắt | Mục 2 (`three_wheeler`): cảnh báo lỗi hệ thống của detector COCO khi review (`CAL-02`) |
| `img_001.jpg`, `img_019.jpg` | Box `#8 bicycle` cao `17.5 px` (`< 20 px`); xe xén mép ở `img_019.jpg` | `yolo26-m` gán xe xén mép thành `truck` | Vật đơn lẻ `< 20 px` không gán box riêng (chỉ gom `ignore_region` khi có cụm ≥ 5 vật sát nhau); xe xén mép dựa vào dáng đầu xe/thùng để phân biệt `car` vs `truck` | Mục 4.1.2 & 4.1.6 (`CAL-03`) |
| `img_013.jpg`, `img_023.jpg`, `img_025.jpg`, `img_027.jpg` | Minibus 16 chỗ (`#142`) bị để `car`; lọt bẫy `loose` và `missing` khi review | Pre-label có box lỏng hoặc thiếu box | Phân biệt rõ: **van/minivan** → `car`, **minibus 12–16 chỗ** → `bus`. Khi review bắt buộc ẩn nhãn (con mắt) quét ảnh trước và soi 4 cạnh viền | Mục 2 (`car` vs `bus`) & Quy trình review (`CAL-04`) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc phân biệt xe đạp điện vs xe máy điện; chuẩn hóa `van` (`car`) vs `minibus` (`bus`); siết ngưỡng `20 px`, ca không rõ hình (`#11`) và quy trình chống thiên kiến khi review `three_wheeler` / `loose` | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` |
