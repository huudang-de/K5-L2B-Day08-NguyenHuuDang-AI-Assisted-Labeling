# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 553; đối tượng sau duyệt: 528; ngưỡng accepted: IoU >= 0.9
- Đã **loại 19 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 491 | 491/553 = 88.8% | 553 box máy |
| edited (sửa) | 8 | 8/553 = 1.4% | 553 box máy |
| deleted (xoá) | 46 | 46/553 = 8.3% | 553 box máy |
| added (vẽ thêm) | 28 | 28/528 = 5.3% | 528 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 18.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 267 | 1 | 11 | 7 |
| bicycle | 6 | 0 | 2 | 2 |
| motorbike | 157 | 0 | 5 | 2 |
| car | 50 | 0 | 6 | 6 |
| bus | 3 | 0 | 0 | 3 |
| truck | 1 | 0 | 6 | 1 |
| three_wheeler | 7 | 7 | 16 | 7 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 43 | 31 | 13 | 58.1% (KTC 95%: 47–69%) | 76.8% (KTC 95%: 64–86%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - extra: 3/3
  - missing: 2/3
  - loose: 1/3
  - lọt: img_014.jpg (wrong_class), img_027.jpg (missing), img_023.jpg (loose), img_025.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
