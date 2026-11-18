# Thông tin học viên

- Họ và tên: Hà Thanh Tùng
- MSHV: 2A202603036
- Lớp: K5-L23
- Tài khoản GitHub: tungsw-IT
- Repo bài nộp: https://github.com/tungsw-IT/K5-L23-DAY08-2A202603036-HaThanhTung-AIPrelabel
- Commit chốt bài: 9ffc883055c97f1493bd442ad97ad295a19e3dcb (commit chứa các sản phẩm của lab).
- Máy: Windows, NVIDIA RTX 3060 Laptop 6 GB, PyTorch bản CPU.
- CVAT Docker: v2.81.0.

## Kết quả tóm tắt

- Máy: accepted 463/621 (74,6%), edited 64/621 (10,3%), deleted 76/621 (12,2%). Added 93/621 đối tượng sau duyệt (15,0%).
- So với gold: precision 50/78 = 64,1%, recall 50/70 = 71,4%.
- Bắt bẫy sau rà lại: 11/12 = 91,7%.
- Thời gian: gán tay 18/4 = 4,5 phút/ảnh. Duyệt khoảng 3 phút/ảnh, tương đương khoảng 72 phút cho 24 ảnh.
- Lỗi QC: 5/5 ảnh (100%) và 6/119 lỗi theo đối tượng (5,0%).
- **Quyết định: Rework.** Tỷ lệ ảnh có lỗi vượt 2% và sai vị trí/kích thước lặp lại ở 5/6 lỗi.
