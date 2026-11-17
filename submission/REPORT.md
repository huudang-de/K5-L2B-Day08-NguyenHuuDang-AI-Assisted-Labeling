# Báo cáo Day 08 — 2A202603300 - Nguyễn Hữu Đăng

> Báo cáo cá nhân tổng hợp từ số liệu thực nghiệm trong `submission/model_log.csv`, `submission/time_log.csv` và `submission/qc/*`. Mọi tỉ lệ đều kèm mẫu số.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| `person` / `bicycle` / `car` / `bus` / `truck` | `yolo26-m` | `0.30` (mặc định) | `r01` (conf 0.25) bắt tốt người/xe phổ biến (`232` box/8 ảnh); `r02` (conf 0.5) giảm nhiễu xuống `140` box nhưng làm mất thêm xe nhỏ. Trên `threshold_sweep.md`, `person`/`bicycle`/`car` chưa đạt recall 90% ở `<= 0.5`, còn `bus` (1 vật) và `truck` (0 vật) quá ít mẫu nên giữ mặc định `0.30`. |
| `motorbike` | `yolo26-m` | `0.25` | Trong `threshold_sweep.md` (11 vật gold), ngưỡng `0.25` là ngưỡng cao nhất `<= 0.5` đạt mục tiêu recall `>= 90%` (đạt `91%`, precision `62%`). |
| `three_wheeler` | `yoloe-s` (`yoloe-26s-seg.pt`, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle`) | `0.30` (mặc định) | `r03` cho thấy YOLOE bắt được tuk-tuk (`img_002`) và xích lô (`img_032`) mà YOLO26 bỏ qua hoặc gán nhầm; trên gold chỉ có 7 vật (recall max `86%`), giữ `0.30` và đặt ưu tiên trước `yolo26-m` khi merge. |

- **Lệnh `merge_prelabels.py` đã dùng:**
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.25" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM đã chọn:** **`sam2.1_b.pt` (SAM 2.1 base)** (có tải thêm `mobile_sam.pt` bản nhẹ để đối chiếu ở Part 5). Lý do: máy có 24 GB RAM (CPU) vẫn chạy được `sam2.1_b.pt` (~13.8s/ảnh ở `r04`), cho 1 polygon liền mạch bám khung tuk-tuk tốt hơn `mobile_sam.pt` (bị tách 2 polygon ở `img_013`) và không cần xin quyền truy cập/GPU như SAM 3.
- **Lỗi không sửa được bằng đổi ngưỡng:** Lỗi **sai/thiếu lớp ngoài COCO** của `yolo26-m`: ở `img_002` bỏ sót tuk-tuk xanh giữa khung hình dù `conf 0.25` hay `0.5`; ở `img_007` nhầm xích lô thành `motorbike` (conf 0.25) rồi biến mất ở conf 0.5; ở `img_004` và `img_013` (`r04`) gọi tuk-tuk là `car`. Cách sửa là dùng open-vocabulary (`yoloe-s`) ưu tiên trước khi merge và kiểm lại bằng mắt khi review.
- **Box điểm cao mà sai / điểm thấp mà đúng:** `yoloe-s` gán nhầm bán tải (`img_002`) và xe xén mép giống ô tô (`img_011`) thành `three_wheeler`; ngược lại ở `img_007` xích lô bị `yoloe-s` bỏ sót dù vẫn bật `conf 0.25`.
- **Giới hạn chọn ngưỡng:** Ngưỡng được quét và chấm trên **chính 4 ảnh gold (60 đối tượng)** nên các con số recall/precision trên gold bị **lạc quan (in-sample)**; riêng `bus` (1 vật) và `truck` (0 vật) quá ít để ước lượng ngưỡng.

## 2. AI error profile (từ `qc/review_stats.md`)

*(Đã loại 19 đối tượng quanh các bẫy nhân tạo; tính trên 24 ảnh batch, 553 box máy, 528 đối tượng sau duyệt).*

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| `accepted` (giữ nguyên) | 491 | 88.8% | 553 box máy |
| `edited` (sửa) | 8 | 1.4% | 553 box máy |
| `deleted` (xoá) | 46 | 8.3% | 553 box máy |
| `added` (vẽ thêm) | 28 | 5.3% | 528 đối tượng sau duyệt |

*(Trong `deleted + added` có 10 trường hợp đổi lớp, tức số vật máy sót thật ≈ `28 - 10 = 18`).*

- **Lớp máy tệ nhất:** **`three_wheeler`** (`accepted = 7`, `edited = 7`, `deleted = 16`, `added = 7`) — tỷ lệ bị xóa và phải vẽ/sửa lại rất cao do YOLOE nhận nhầm xe khác hoặc vẽ box lỏng/trùng, còn YOLO26 thì gán nhầm sang `car`/`truck`. Tiếp theo là `truck` (`accepted = 1`, `deleted = 6`) và `car` (`deleted = 6`, `added = 6`).
- **So với gold (4 ảnh, 56 đối tượng gold hợp lệ):** Máy đạt **Precision = `58.1%` (KTC 95%: 47–69%, 43 TP / 74 pred)** và **Recall = `76.8%` (KTC 95%: 64–86%, 43 TP / 56 gold)**.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy:** **`8/12 = 66.7%`** (KTC 95%: 39–86%, dưới ngưỡng đạt 80%).
  - Bắt được: `extra = 3/3`, `wrong_class = 2/3`, `missing = 2/3`, `loose = 1/3`.
  - **4 bẫy bị lọt:** `img_014.jpg` (`wrong_class`), `img_027.jpg` (`missing`), `img_023.jpg` (`loose`), `img_025.jpg` (`loose`).
- **Nguyên nhân (bước checklist bị bỏ qua):** Khi thấy các box của máy có `accepted` cao (`88.8%`), mắt dễ tin vào khung có sẵn nên lướt nhanh khâu **kiểm tra 4 cạnh viền từng box** (dẫn tới lọt 2/3 bẫy `loose` ở `img_023`, `img_025`) và **ẩn nhãn quét lại vùng rìa/nền xa** (lọt 1 bẫy `missing` ở `img_027` và 1 `wrong_class` ở `img_014`).
- **Ý nghĩa của `accepted` cao (`88.8%`) nhưng bắt bẫy thấp (`66.7%`):** Tỷ lệ giữ nguyên cao không đồng nghĩa chất lượng review tốt; trái lại nó phản ánh **automation bias (thiên kiến tự động hóa)** — người duyệt có xu hướng chấp nhận các box trông "hợp lý" mà bỏ sót lỗi viền lỏng hoặc thiếu đối tượng.

## 4. Máy có đáng dùng không?

- **Thời gian (từ `time_log.csv`):**
  - Gán tay từ đầu (`gold_manual`): `17 phút / 4 ảnh` = **`4.25 phút/ảnh`**.
  - Duyệt có pre-label (`batch_review`): `46 phút / 24 ảnh` = **`1.92 phút/ảnh`** (nhanh hơn **~2.2 lần**).
  - **Thiên lệch cần lưu ý:** Vì đã gán 4 ảnh gold trước nên đã quen bối cảnh đường phố châu Á và quy tắc guideline, giúp tốc độ duyệt batch nhanh hơn một phần do hiệu ứng quen tay.
- **Kết luận:** **Có đáng dùng**, đặc biệt cho các lớp đông và phổ biến như `person`, `motorbike`, `car` (`accepted` lần lượt `267`, `157`, `50`) giúp giảm hơn một nửa thời gian vẽ tay; riêng lớp `three_wheeler` vẫn cần người duyệt kiểm chặt.
- **Ba quy tắc giảm thiên kiến (kiểm tra được):**
  1. **Ẩn toàn bộ nhãn trước khi xem ảnh (100% số ảnh):** Bấm nút con mắt trong CVAT, tự đếm nhẩm tổng số người/xe theo chiều trái→phải trước khi bật lại box để chặn lỗi `missing`.
  2. **Kiểm tra riêng 4 cạnh viền theo lớp rủi ro:** Lọc riêng các box `three_wheeler`, `car`, `truck` bằng thanh *Filters* để kéo sát viền từng box, loại lỗi `loose` và `sai_lop`.
  3. **Nghỉ mắt bắt buộc mỗi 8–10 ảnh (tối thiểu 2 phút):** Tránh mỏi mắt gây bấm `Accept` theo quán tính ở nửa sau của lô 24 ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| `img_013.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | **Còn dính** một phần lưng/quần/chân người lái và nền phía sau | `1` điểm `--neg` trên lưng chưa đủ tách hết người lái (phủ `45.5%` ảnh, `1` polygon) | `tuktuk_img013_sam2.jpg` |
| `img_013.jpg` | `mobile_sam.pt` | 3 (2 pos, 1 neg) | Tách áo/quần người lái sạch hơn, nhưng vẫn **dính mặt và bàn chân** | Mask bị vỡ thành **`2` polygon** (`35.3%` ảnh) và cắt sót một phần khung trước | `tuktuk_img013_sam1.jpg` |
| `img_004.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | **Không dính** người đứng cạnh | Ôm gọn xe tuk-tuk đỏ (`13.4%` ảnh, `1` polygon) nhưng bỏ sót biển `TAXI` trên nóc | `tuktuk_img004_sam2.jpg` |
| `img_017.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | **Không dính** người ngoài | Do đặt điểm `--neg` ngay trên chữ `MR.K` ở mái xe nên mask bị khoét lẹm một mảng trên nóc (`12.1%` ảnh, `1` polygon) | `tuktuk_img017_sam2.jpg` |

- **Đối chiếu guideline mục 4.2:** Quy tắc 4.2.4 (hai vật chạm nhau không chồng lên nhau) giúp xác định rõ phải tách người lái ra khỏi mask xe tuk-tuk. Tuy nhiên với khung xe rỗng có chữ/biển hiệu (`img_017`), cần quy tắc rõ hơn về việc đặt điểm âm ngoài viền mái và xử lý biển `TAXI` trên nóc xe (`img_004`).
- **So sánh chọn phiên bản:** `sam2.1_b.pt` giữ khối polygon liền mạch hơn (`1` polygon), trong khi `mobile_sam.pt` nhẹ và nhanh hơn nhưng dễ đứt gãy viền khung sắt mảnh.

## 6. Calibration (Part 6)

- **Số quyết định:** **`4`** quyết định (`CAL-01` đến `CAL-04` trong `submission/qc/calibration_decisions.csv`); quan trọng nhất là **`CAL-01`** (chốt xe đạp điện có bàn đạp → `bicycle`, xe máy điện không bàn đạp → `motorbike`) và **`CAL-04`** (phân biệt `van` → `car` vs `minibus` → `bus`, cùng quy tắc ẩn nhãn khi review).
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:** Bổ sung phân loại xe đạp điện/xe máy điện, phân biệt van vs minibus, giữ ngưỡng `< 20 px` cho box đơn lẻ (bỏ box không thấy rõ hình như `#11` ở `img_003`), và quy trình kiểm tra riêng lớp `three_wheeler`.
- **Nguyên nhân gốc:** Kết hợp giữa **guideline v1.0 còn câu hỏi mở** (xe đạp điện, ca sát ngưỡng 20 px), **hạn chế của model** (YOLO26 không có lớp `three_wheeler`), và **sai sót chủ quan của người duyệt** (bỏ qua bước soi 4 cạnh viền và sót minibus `#142` ở `img_013`).

## 7. QC và quyết định (Part 6)

- **Mẫu:** **`6` ảnh** (`1` ảnh `can_review` + `5` ảnh ngẫu nhiên, `seed 42`, gồm `img_003`, `006`, `013`, `014`, `022`, `033`), tổng **`116` đối tượng**. *Hạn chế:* tự kiểm tra trên chính lô mình duyệt nên có thể sót cùng điểm mù với lúc review.
- **Lỗi theo đối tượng:** **`2/116 = 1.7%`** · **Lỗi theo ảnh:** **`2/6 = 33.3%`** (vượt xa ngưỡng Release `<= 2%`).
- **Top lỗi:**
  - `thua`: `1` lỗi (`img_003.jpg`, box `#11 three_wheeler` chỉ thấy tên/không thấy rõ hình xe).
  - `sai_lop`: `1` lỗi (`img_013.jpg`, box `#142` xe 16 chỗ/minibus bị gán `car` thay vì `bus`).
- **Giới hạn mẫu nhỏ:** Với `2/6` ảnh lỗi, **cận trên khoảng tin cậy Wilson 95% trong `qc_report_auto.md` lên tới `70%`**. Mẫu 6 ảnh quá nhỏ để chứng minh lô đạt ngưỡng `<= 2%`, chỉ đủ để phát hiện sớm các lỗi lớn/lặp.
- **Script kiểm được / không kiểm được:** `validate_labels.py` và `qc_report.py` kiểm được schema, ảnh thiếu/thừa, chiều cao `< 20 px` và tính tỉ lệ từ phiếu kiểm, nhưng **không** tự kiểm được độ ôm sát viền box, không phát hiện được ca bỏ sót nếu cả máy và người cùng sót, cũng không kiểm chứng được người duyệt có thực hiện đúng checklist 4 bước hay không.

**Quyết định: `Rework`**, ngày `2026-11-17`.
**Lý do (theo cây Hold → Rework → Release):** Không bị `Hold` vì guideline đã được hiệu chỉnh sang `v1.1` và dữ liệu ảnh gốc sạch; tuy nhiên phải chọn **`Rework`** vì đồng thời vi phạm hai điều kiện chất lượng: **(1)** tỉ lệ ảnh có lỗi ở mẫu QC là `2/6 = 33.3% > 2%`, và **(2)** tỉ lệ bắt bẫy chỉ đạt `8/12 = 66.7% < 80%` (lọt 2 bẫy `loose`, 1 `missing`, 1 `wrong_class`).
**Việc cần làm tiếp:** Mở lại lô `batch-review` trên CVAT theo `guideline_v1.1_calibrated.md`, sửa trực tiếp box `#11` (`img_003`), `#142` (`img_013`) và rà lại toàn bộ 24 ảnh với checklist bắt buộc ẩn nhãn + kiểm 4 cạnh viền trước khi chạy lại QC.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Có sử dụng trợ lý AI (Gemini) để giải thích quy trình/guideline, hỗ trợ chạy lệnh kiểm tra, tổng hợp CSV/markdown từ số liệu thực tế; mọi số liệu đều đối chiếu trực tiếp từ output của `validate_labels.py`, `threshold_sweep.py`, `review_stats.py`, `qc_report.py` và ảnh `viz/`/`masks/`/`work/qc/review/`.
- **Gold có bị nhiễm không:** **Không.** 4 ảnh gold được gán tay hoàn toàn trên CVAT ở Part 1 trước khi tải trọng số và trước khi chạy bất kỳ model pre-label nào.
- **Máy / GPU / CVAT dùng:** Windows, CPU (RAM 24 GB, không có GPU), CVAT Docker `v2.74.1` chạy tại `http://localhost:8080`.
- **Dùng cache `data/cache/` thay vì tự chạy?** **Không** — toàn bộ model `yolo26m.pt`, `yoloe-26s-seg.pt`, `sam2.1_b.pt`, `mobile_sam.pt` được tải về `models/` và chạy trực tiếp trên máy.
- **Lỗi / bug gặp phải:** Không có bug script; có gặp tình trạng mạng timeout cổng 443 khi `git push` lên GitHub (xử lý bằng ép IPv4 `git -4` và lưu commit cục bộ trước).
- **Một điều làm khác lần sau:** Ở Part 4 sẽ bật chế độ ẩn nhãn ngay từ đầu cho từng ảnh và kiểm riêng 4 cạnh viền của tất cả box để không bị lọt bẫy `loose` và `missing`.
