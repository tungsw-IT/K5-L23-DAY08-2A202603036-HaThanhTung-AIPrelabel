# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 621; đối tượng sau duyệt: 621; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 463 | 463/621 = 74.6% | 621 box máy |
| edited (sửa) | 64 | 64/621 = 10.3% | 621 box máy |
| deleted (xoá) | 76 | 76/621 = 12.2% | 621 box máy |
| added (vẽ thêm) | 93 | 93/621 = 15.0% | 621 đối tượng sau duyệt |

Trong số deleted + added, **14 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 79.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 249 | 12 | 8 | 39 |
| bicycle | 6 | 0 | 2 | 7 |
| motorbike | 152 | 43 | 27 | 25 |
| car | 46 | 2 | 6 | 7 |
| bus | 2 | 0 | 1 | 5 |
| truck | 1 | 0 | 5 | 4 |
| three_wheeler | 7 | 7 | 27 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 70 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 50 | 28 | 20 | 64.1% (KTC 95%: 53–74%) | 71.4% (KTC 95%: 60–81%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - wrong_class: 3/3
  - extra: 3/3
  - loose: 2/3
  - lọt: img_034.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
