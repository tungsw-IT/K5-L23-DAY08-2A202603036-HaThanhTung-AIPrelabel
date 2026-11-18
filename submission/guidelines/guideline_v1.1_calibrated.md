# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603036 - Hà Thanh Tùng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Xử lý người lái nhìn thấy một phần qua khoang tuk-tuk thế nào? | Giữ tiêu chí thấy ít nhất 20% của v1.0. Ước lượng từ phần đầu, thân và chi nhìn thấy. Nếu sát ngưỡng và khó quyết định, tick `can_review`. Tách phần người lái nhìn thấy khỏi mask xe. | [img_013](../../data/images/img_013.jpg) | CAL-03 |
| 6 | Xử lý box chồng nhau trong ảnh đêm thế nào? | Kiểm tra box theo từng vật. Giữ box `person` và `motorbike` riêng khi nhận ra cả người lẫn xe. Chỉ xóa box dư nếu hai box cùng chỉ một vật. | [img_024](../../data/images/img_024.jpg) | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| [img_004](../../data/images/img_004.jpg) | Tuk-tuk xanh có mui | YOLO26-m ở lượt `r04` gán `car`. | **CAL-01.** Bổ sung vào bước duyệt sai lớp. Với box `car` trên xe có mui, kiểm tra dạng thân xe và số bánh nhìn thấy. Đổi thành `three_wheeler` khi nhận ra tuk-tuk. Nếu chưa đủ căn cứ để phân biệt, tick `can_review`. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản ban đầu của giảng viên | — |
| v1.1 | Bổ sung cách rà tuk-tuk bị nhầm lớp, box chồng nhau trong ảnh đêm và người lái bị che. | CAL-01, CAL-02, CAL-03 |
