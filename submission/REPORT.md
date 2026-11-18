# Báo cáo Day 08 — 2A202603036 - Hà Thanh Tùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao |
|---|---|---:|---|
| person | YOLO26-m (r01, r02) | 0.30 | Giữ mặc định. Recall tốt nhất khi quét là 79% trên 34 vật gold, dưới mục tiêu 90%. |
| bicycle | YOLO26-m (r01, r02) | 0.30 | Giữ mặc định. Recall tốt nhất là 67% trên 9 vật gold. |
| motorbike | YOLO26-m (r01, r02) | 0.10 | Đạt recall 91% trên 11 vật gold. Precision 40%, nên cần rà box thừa khi duyệt. |
| car | YOLO26-m (r01, r02) | 0.30 | Giữ mặc định. Recall tốt nhất là 88% trên 8 vật gold. |
| bus | YOLO26-m (r01, r02) | 0.30 | Giữ mặc định vì chỉ có 1 vật gold, dưới mức tối thiểu 5. |
| truck | YOLO26-m (r01, r02) | 0.30 | Giữ mặc định vì có 0 vật gold. |
| three_wheeler | YOLOE-s (r03) | 0.15 | Đạt recall 100% trên 7 vật gold và precision 78%. |

Nguồn số liệu là `submission/qc/threshold_sweep.csv`. Ngưỡng được chọn và đánh giá trên cùng 4 ảnh gold, nên precision và recall ở ngưỡng đã chọn có thể lạc quan. Mẫu nhỏ cũng khiến các ngưỡng chỉ là ước lượng ban đầu.

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```powershell
  .\.venv\Scripts\python.exe scripts\merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Mình chọn SAM 2.1 Base (`sam2.1_b.pt`) vì chạy được trên CPU của máy và mask người áo tím ở mép phải `img_004.jpg` bám khá sát thân và chân. Lượt `r04` tạo mask cho 16/16 box trên 2 ảnh, trung bình 5,15 giây/ảnh cho cả YOLO26 và SAM
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Trong `img_004.jpg`, YOLO26 gọi tuk-tuk màu xanh lá là `car`. Đổi ngưỡng chỉ quyết định giữ hay loại box đó. Mình dùng YOLOE để đề xuất box `three_wheeler`, rồi kiểm tra và sửa nhãn trong CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Trong `img_032.jpg`, box `person` có điểm 0.26 bao người mặc áo sọc bị che một phần cạnh người đội mũ trắng. Box xuất hiện ở lượt `r01` và biến mất khi tăng ngưỡng lên 0.50 ở lượt `r02`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Sweep chỉ dùng 4 ảnh gold. Vì ngưỡng được chọn và chấm trên cùng 4 ảnh, precision và recall đo trên gold có thể lạc quan. Các ngưỡng này là ước lượng ban đầu từ một mẫu nhỏ.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 463 | 463/621 = 74,6% | 621 box máy |
| edited | 64 | 64/621 = 10,3% | 621 box máy |
| deleted | 76 | 76/621 = 12,2% | 621 box máy |
| added | 93 | 93/621 = 15,0% | 621 đối tượng sau duyệt

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler`, với 27 box bị xóa, 7 box giữ nguyên và 7 box được chỉnh. Một lỗi khác của nhóm xe ba bánh là nhầm lớp. Ví dụ, trong `img_004.jpg`, YOLO26 gán chiếc tuk-tuk xanh lá thành `car` ở lượt `r04`.
- So với gold: precision / recall của **máy** Trên 4 ảnh, máy có 50 phát hiện đúng, 28 phát hiện sai và bỏ sót 20 vật. Precision là **50/78 = 64,1%** trên 78 box máy. Recall là **50/70 = 71,4%** trên 70 đối tượng gold được tính. Vì ngưỡng được chọn và đánh giá trên cùng 4 ảnh, các số đo này có thể lạc quan.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 (91,7%); loại nào lọt: Còn lọt một bẫy box lỏng ở `img_034.jpg`, Bước mình cần làm kỹ hơn là kiểm tra từng cạnh của box có ôm sát phần nhìn thấy của vật hay không.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Mình giữ nguyên **463/621 box máy (74,6%)** và bắt được **11/12 bẫy (91,7%)**. Kết quả cho thấy mình giữ phần lớn box máy và phát hiện được hầu hết bẫy, nhưng vẫn cần rà kỹ độ ôm sát của box ở những vùng nhiều xe chồng lên nhau.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 18/4 = 4,5 phút/ảnh và duyệt khoảng 3 phút/ảnh (ước tính 72 phút cho 24 ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh):  Vì mình đã gán gold trước, mình quen với các loại cảnh và quy tắc gán nhãn hơn khi sang lượt duyệt. Hai tập ảnh cũng có thể khác nhau về độ khó, nên chênh lệch thời gian chỉ là ước lượng.
- Kết luận: Có, mình sẽ dùng pre-label để hỗ trợ gán nhãn `person` và `motorbike`, rồi kiểm tra lại trong CVAT. Sau lượt rà cuối, mình giữ nguyên **463/621 box máy (74,6%)**, nhưng vẫn phải vẽ thêm **93/621 đối tượng sau duyệt (15,0%)**, trong đó có cả trường hợp đổi lớp. Với `three_wheeler`, mình sẽ rà từng đề xuất vì có **27 box bị xóa** và chỉ **7 box được giữ nguyên**. Mức tiết kiệm thời gian cần được xác nhận bằng số phút duyệt thực tế.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
   1. Ẩn toàn bộ box máy và tự quét từng ảnh từ trái sang phải trước khi bật nhãn lại. Làm đủ 24/24 ảnh.
  2. Trước khi chuyển ảnh, kiểm đủ bốn mục theo thứ tự: vật bị bỏ sót, box thừa, sai lớp và box lỏng. Với box bị che khuất, zoom để kiểm từng cạnh.
  3. Lưu và export toàn bộ nhãn trước khi mở `traps_key.json`. Ghi kết quả rà lại sau khi xem thống kê thành một lượt riêng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` — tuk-tuk | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Dính một phần người lái dù đã đặt điểm âm. Không thấy kiện hàng rõ để đánh giá. | Mask phủ phần lớn mui, thân và bánh xe nhưng lấn sang người lái và người phía sau nhìn qua khoang xe. | `tuktuk_img013.jpg` |
| `img_032.jpg` — xích lô giữa ảnh | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Người đứng cạnh xe nằm ngoài mask. Không thấy hàng hóa rõ để đánh giá. | Mask bắt được ghế, khung trước và hai bánh lớn. Một số khe ở khung gầm bị tô kín. | `cyclo_img032.jpg` |
| `img_032.jpg` — người áo xanh bị che | SAM 2.1 Base (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Không áp dụng vì vật cần lấy mask là người. | Mask bắt được đầu và phần vai nhô lên sau xích lô. Còn thiếu một phần vai sát người áo trắng. | `person_occluded_img032.jpg` |

- **Đối chiếu guideline mục 4.2**. Quy tắc bám theo phần nhìn thấy giúp kiểm biên của người áo xanh bị che. Quy tắc tách hai vật chạm nhau giúp phát hiện phần mask tuk-tuk lấn sang người lái. Quy tắc bỏ các lỗ lớn giúp rà khoảng trống trong khung xích lô. Mục 4.2 có thể bổ sung ví dụ cụ thể về đường biên giữa thân tuk-tuk và người lái khi hai phần chồng lên nhau trong ảnh.
- *Interactor SAM trong CVAT (tùy chọn)*. Mình chạy cả ba ca offline bằng `scripts/sam2_points.py`. Chưa thử Interactor trong CVAT nên chưa có kết quả để so sánh.


## 6. Calibration (Part 6)

- Số quyết định: 3  (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`):  Trong `img_004.jpg`, YOLO26 gán tuk-tuk xanh có mui thành `car`. Khi xác định xe là tuk-tuk, sửa nhãn thành `three_wheeler`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Với xe có mui bị gán `car`, kiểm tra hình dáng thân xe và số bánh nhìn thấy, rồi đánh dấu `can_review` nếu vẫn chưa chắc (`CAL-01`). Với box chồng nhau trong ảnh đêm, giữ box riêng cho từng vật nhận diện được và chỉ xóa box trùng cùng một vật (`CAL-02`). Với người lái bị che, giữ ngưỡng nhìn thấy ít nhất 20% và đánh dấu `can_review` nếu khó ước lượng. Khi tạo mask xe, tách phần người lái nhìn thấy khỏi mask thân xe (`CAL-03`).
- Nguyên nhân gốc:  `CAL-01` là lỗi phân lớp của máy. `CAL-02` và `CAL-03` là các tình huống guideline cần hướng dẫn cụ thể hơn để áp dụng nhất quán.

## 7. QC và quyết định (Part 6)

- Mẫu: 5/24 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); với 119 đối tượng. Mình tự kiểm lô đã duyệt, nên hai lượt có thể cùng bỏ qua một loại lỗi.
- Lỗi theo **đối tượng**: 6/119 = 5,0%. **Ảnh có lỗi:** 5/5 = 100%, vượt ngưỡng Release ≤ 2%.
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Sai vị trí hoặc kích thước chiếm 5/6 lỗi. Ví dụ, box `#177` trong `img_014.jpg` lấn lên mặt đường phía trên yên xe. Box thừa chiếm 1/6 lỗi. Trong `img_031.jpg`, box `#551` gán `person` cho chiếc mũ cam trên kệ.

- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`):  Với 5/5 ảnh mẫu có lỗi, cận trên Wilson 95% là 100%. Tỷ lệ 100% mô tả 5 ảnh đã kiểm. Mẫu này quá nhỏ để chứng minh tỷ lệ ảnh lỗi của cả lô ≤ 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật):  Script đếm lỗi đã ghi trong `qc_review.csv` và tính tỷ lệ. Độ ôm sát của box, vật mà cả máy lẫn người duyệt đều bỏ sót và việc thực hiện checklist cần được kiểm tra bằng mắt.

**Quyết định: Rework**, ngày 2026-11-18. Kết quả QC cho thấy lỗi vị trí hoặc kích thước lặp lại ở 5/6 lỗi, và cả 5 ảnh mẫu đều có lỗi nên lô cần được sửa trước khi xét Release.

Nếu Rework / Hold, việc cần làm tiếp: Rà lại độ ôm sát của box trên cả 24 ảnh, sửa các box đã ghi trong phiếu QC và xóa box thừa đã xác nhận. Sau đó tạo lại release candidate và lấy mẫu QC vòng tiếp theo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài.** Chatpt.
- Bộ gold được gán thủ công và kiểm tra trước khi chạy các lượt pre-label trong lab.
- Windows, RTX 3060 Laptop 6 GB, PyTorch bản CPU và CVAT Docker v2.81.0.
- Dự đoán chính được chạy trong repo; lấy kết quả từ các lượt ghi trong work/runs.csv.
- `img_001.jpg` — 2 người ở đằng sau xe ô tô. Đã tick `can_review` vì chưa chắc có nên vẽ box bao cả 2 người đó không.
- Python hệ thống thiếu numpy nên chuyển sang .venv. Export batch ban đầu tên instances_default.json, sau đó đổi thành batch_reviewed.json để chạy thống kê.
- Lần sau sẽ kiểm từng cạnh của box ở các vùng nhiều xe chồng lên nhau trước khi export.


### Nâng cao — So sánh YOLOE, Grounding DINO và OWLv2

Mình chạy ba model trên cùng 8 ảnh pilot với lớp `three_wheeler`. Số box và thời gian CPU lấy từ `work/runs.csv`.

| Model | Ngưỡng | Box đề xuất trên 8 ảnh | Giây/ảnh |
|---|---:|---:|---:|
| YOLOE-s (`r03`) | 0.25 | 8 | 0,30 |
| Grounding DINO (`r07`) | 0.30 | 18 | 8,09 |
| OWLv2 (`r08`) | 0.20 | 44 | 5,72 |

Đối chiếu bằng mắt trên 4 ảnh. Chỉ đếm xe ba bánh nhận diện rõ. Mỗi xe chỉ tính một lần cho mỗi model. Trong mỗi ô, số trước là **xe tìm đúng**, số sau là **box dư, trùng hoặc chưa xác nhận**. Các xe bị che nhiều hoặc ở quá xa cần kiểm thêm trước khi kết luận.

| Ảnh | Xe ba bánh thấy rõ | YOLOE: đúng / dư | Grounding DINO: đúng / dư | OWLv2: đúng / dư |
|---|---:|---:|---:|---:|
| `img_002.jpg` | 1 | 1 / 3 | 1 / 1 | 1 / 2 |
| `img_011.jpg` | 1 | 1 / 0 | 0 / 2 | 1 / 0 |
| `img_016.jpg` | 0 | 0 / 0 | 0 / 4 | 0 / 1 |
| `img_032.jpg` | 4 | 3 / 0 | 2 / 0 | 4 / 9 |
| **Tổng trên 4 ảnh** | **6** | **5 / 3** | **3 / 7** | **6 / 12** |

- Trong `img_011.jpg`, YOLOE và OWLv2 tìm được tuk-tuk xanh sát mép trái. Hai box của Grounding DINO nằm trên các xe máy ở tiền cảnh.
- Trong `img_032.jpg`, OWLv2 tìm thêm chiếc xích lô ghế xanh bị người đội mũ trắng che một phần. Một số box khác của OWLv2 chỉ bao bánh xe hoặc trùng trên cùng một xích lô. Trong `img_016.jpg`, Grounding DINO còn gán nhầm taxi trắng-xanh thành xe ba bánh.
- Mình chọn YOLOE làm nguồn đề xuất chính vì tìm được **5/6 xe thấy rõ** trong 4 ảnh kiểm và chạy **0,30 giây/ảnh** trên CPU. Có thể dùng OWLv2 để tìm thêm xe bị che khi chấp nhận rà nhiều box hơn. Đây là đối chiếu trên 4 ảnh với các ngưỡng riêng của từng model. Cần một tập nhãn độc lập lớn hơn để kết luận về precision và recall.



### Nâng cao — Chuẩn bị pilot, mask batch và gán lại mù

- **Pilot riêng (mục 3).** Đã chuẩn bị 8 ảnh trong `work/tasks/pilot/` và gộp 321 box ở ngưỡng 0.10 vào `work/prelabel/merged_pilot/pre.coco.json`. Chưa có bản duyệt `pilot_reviewed.json`, nên chưa chọn ngưỡng pilot. Khi duyệt, cần tự tìm và thêm cả vật nằm dưới ngưỡng đề xuất.
- **Mask cả tập seg (mục 4).** YOLO26-m → SAM 2.1 Base tạo 74 mask và YOLOE-s → SAM 2.1 Base tạo 22 mask, mỗi lượt chạy trên 8 ảnh. Dùng ngưỡng từ Part 3 cho lượt YOLO26 và ngưỡng 0.25 cho lượt YOLOE. Sau khi gộp ở 0.25, còn 78 mask trên 8 ảnh, lưu ở `prelabel/seg_pre.coco.json`. XML nạp vào CVAT là `work/cvat_import/seg_pre.xml`. Chưa có bản review mask nên chưa tính thống kê segmentation.
- **AI trực tiếp trong CVAT (mục 5).** Đã xác nhận Docker đang chạy các chức năng MobileSAM và YOLOE. Chưa có lượt thao tác trực tiếp trên một task để so sánh với chạy offline.
- **Gán lại mù (mục 6).** Đã chuẩn bị 8 ảnh trong `work/tasks/selfcheck/`. Chưa có bản gán lại thủ công bởi người đã duyệt lô, nên chưa tính độ nhất quán.

### Nâng cao — So sánh độ phân giải YOLO26 trên gold

So sánh YOLO26-m ở `imgsz 1024` và `1280` trên cùng 4 ảnh gold, giữ ngưỡng motorbike 0.10 và các lớp YOLO26 còn lại 0.30. Ghép box cùng lớp với IoU ≥ 0.50 và loại dự đoán trong vùng ignore theo script. Trong phép thử này, vật nhỏ là vật có box cao từ 20 đến dưới 40 px trên ảnh gốc.

| imgsz | Recall vật nhỏ | Precision (6 lớp YOLO26) | Recall (6 lớp YOLO26) |
|---|---:|---:|---:|
| 1024 | 15/29 = 51,7% | 43/75 = 57,3% | 43/63 = 68,3% |
| 1280 | 19/29 = 65,5% | 49/85 = 57,6% | 49/63 = 77,8% |

Ở mức 1280, model tìm đúng thêm 4 vật nhỏ và đưa ra thêm 10 box trên 4 ảnh. Lượt 1280 chạy CPU ở 1,19 giây/ảnh. Các ngưỡng đã dùng từng được chọn trên chính 4 ảnh gold, nên đây là so sánh trong mẫu. Máy có RTX 3060 Laptop, nhưng môi trường PyTorch đang dùng bản CPU. Các lượt sweep riêng cho YOLO26 và YOLOE nằm dưới `work/qc/advanced_*`. Ở ngưỡng 0.15, YOLOE bắt được 7/7 xe ba bánh với 7/9 box đúng trên gold.

Ở cùng imgsz 1280 và cùng các ngưỡng, YOLO26-l tìm đúng 19/29 vật nhỏ (65,5%), precision 49/78 (62,8%), recall 49/63 (77,8%) ở 1,50 giây/ảnh CPU (r12). YOLO26-x tìm đúng 20/29 vật nhỏ (69,0%), precision 51/85 (60,0%), recall 51/63 (81,0%) ở 2,49 giây/ảnh (r13). Với 4 ảnh này, x chỉ thêm một vật nhỏ so với m/l ở 1280.

SAM 2.1 Large dùng lại đúng 3 click trên tuk-tuk trong img_013.jpg. Mask còn 292.883 px (41,9% ảnh), so với 317.255 px (45,4%) của Base. Large loại được phần lớn người lái nhưng vẫn lấn sang người phía sau và một phần gần tay người lái. File kết quả là masks/tuktuk_img013_sam_large.jpg. Diện tích chỉ mô tả đầu ra vì chưa có mask chuẩn cho xe này.

**Các phép thử tùy chọn chưa thực hiện.** Bỏ qua SAM 3. Các mục đo trên pilot đã duyệt, QC polygon, AI trực tiếp trong CVAT và gán lại mù cần các lượt thao tác hoặc bản export riêng. Chưa đo lỗi residual/new trên gold sau 24 giờ. Các lượt thử đã dùng dự đoán chạy tại máy.
