# Theo dõi thai kỳ

Trang web HTML thuần (không cần build, không thư viện ngoài) để theo dõi các chỉ số
siêu âm của em bé qua từng mốc khám. Mỗi chỉ số một biểu đồ riêng (vì đơn vị khác
nhau: g, mm, lần/phút…), dùng chung trục thời gian, có tooltip tổng hợp và bảng số liệu.

## Chạy

```bash
cd baby_monitor
python3 -m http.server 8000
# mở http://localhost:8000
```

Cần chạy qua web server để trang tự liệt kê file trong `data/`. Nếu mở trực tiếp
file `index.html` (giao thức `file://`), trang sẽ hiện nút **Chọn thư mục data…**
để bạn chọn thư mục bằng tay.

## Đưa lên GitHub Pages

Push repo (public) lên GitHub rồi bật Pages (Settings → Pages → deploy từ branch).
Trang tự nhận diện `owner/repo` từ URL `*.github.io` và hỏi GitHub API danh sách
file trong `data/` — thêm chỉ số mới chỉ cần push file `.txt`, không phải khai báo gì.

Hai trường hợp cần chỉnh tay (sửa 2 hằng số ở đầu `<script>` trong `index.html`):

- **Custom domain** (không phải `*.github.io`): điền `GITHUB_REPO = "owner/repo"`.
- **Trang không nằm ở gốc repo** (ví dụ deploy từ `docs/`): sửa `DATA_PATH`
  thành đường dẫn thật trong repo, ví dụ `"docs/data"`.

Lưu ý: GitHub API đọc branch mặc định của repo. Nếu Pages deploy từ branch khác
(ví dụ `gh-pages`) thì danh sách có thể lệch — khi đó dùng `data/manifest.txt`
(xem bên dưới) làm nguồn danh sách thay thế.

## Thêm chỉ số mới

Tạo một file `.txt` trong thư mục `data/`, các cột cách nhau bằng **tab**:

```
Cân nặng ước tính	g
16	146
20	331
22	478
```

- **Dòng đầu**: tên chỉ số `<tab>` đơn vị đo.
- **Các dòng sau**: mốc khám `<tab>` giá trị đo được.
- Mốc khám là **tuần thai** (`12`, `12.5`, `12w3d`, `tuần 12`) hoặc **ngày khám**
  (`13/08/2026`, `2026-08-13`) — dùng thống nhất một kiểu cho tất cả các file.
- Tuần nào chưa đo chỉ số nào thì bỏ qua, không cần điền — biểu đồ tự xử lý điểm thiếu.
- Số thập phân viết `12.5` hoặc `12,5` đều được.

Thêm file xong bấm **Tải lại dữ liệu** (hoặc F5). Không cần khai báo gì thêm —
trang tự quét thư mục `data/`.

Nếu server của bạn không bật autoindex (trang không tự thấy file mới), tạo file
`data/manifest.txt` liệt kê mỗi dòng một tên file:

```
can-nang.txt
nhip-tim.txt
```

## Tính năng

- Tick bật/tắt từng chỉ số (trạng thái được nhớ lại giữa các lần mở).
- Di chuột lên biểu đồ: crosshair đồng bộ mọi panel + tooltip hiện tất cả chỉ số
  tại mốc đó.
- Bảng số liệu đầy đủ ở cuối trang (mốc nào thiếu chỉ số nào hiện "–").
- Tự theo giao diện sáng/tối của hệ điều hành.
