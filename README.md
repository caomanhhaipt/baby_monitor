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
28/06/2026	16w1	146
25/07/2026	20	331
10/08/2026	22w2	478
```

- **Dòng đầu**: tên chỉ số `<tab>` đơn vị đo.
- **Các dòng sau**: ngày khám `<tab>` tuần thai `<tab>` giá trị đo được.
- Ngày khám viết `28/06/2026` hoặc `2026-06-28`.
- Tuần thai viết `20` (chẵn tuần) hoặc `16w1` (16 tuần 1 ngày); `16w1d`,
  `16.5` cũng được. Biểu đồ vẽ theo tuần thai; ngày khám hiện ở tooltip
  và bảng số liệu.
- Lần khám nào không đo chỉ số nào thì bỏ qua, không cần điền — biểu đồ tự
  xử lý điểm thiếu.
- Số thập phân viết `12.5` hoặc `12,5` đều được.
- Format 2 cột cũ (`mốc khám <tab> giá trị`) vẫn đọc được.

Thêm file xong bấm **Tải lại dữ liệu** (hoặc F5). Không cần khai báo gì thêm —
trang tự quét thư mục `data/`.

Nếu server của bạn không bật autoindex (trang không tự thấy file mới), tạo file
`data/manifest.txt` liệt kê mỗi dòng một tên file:

```
can-nang.txt
nhip-tim.txt
```

## Ngày dự sinh & vạch "hôm nay"

Khai báo trong `config.txt` (cạnh `index.html`) một trong hai dòng:

```
du-sinh: 12/12/2026
kinh-cuoi: 07/03/2026
```

Trang sẽ hiện dòng "Hôm nay bé được 22w5 tuần · còn N ngày tới dự sinh" và vẽ
vạch **hôm nay** trên các biểu đồ (khi mốc hôm nay ở gần sau lần khám cuối).

## Khoảng tham chiếu

Nếu tồn tại file `refs/<cùng tên file trong data/>`, biểu đồ chỉ số đó sẽ có
vùng mờ p5–p95 và đường trung vị phía sau đường đo. Format mỗi dòng:
`tuần <tab> p5 <tab> p50 <tab> p95` (dòng tiêu đề tùy chọn).

> ⚠️ Số liệu trong `refs/` đi kèm repo là giá trị **xấp xỉ** tổng hợp từ các
> bảng tăng trưởng thông dụng (Hadlock, WHO), chỉ để tham khảo hình dạng đường
> cong. Hãy thay bằng bảng chuẩn mà phòng khám/bác sĩ của bạn sử dụng, và mọi
> kết luận về sức khỏe của bé luôn theo bác sĩ.

## Tính năng

- Tick bật/tắt từng chỉ số (trạng thái được nhớ lại giữa các lần mở).
- Di chuột lên biểu đồ: crosshair đồng bộ mọi panel + tooltip hiện tất cả chỉ số
  tại mốc đó.
- Bảng số liệu đầy đủ ở cuối trang (mốc nào thiếu chỉ số nào hiện "–").
- Tự theo giao diện sáng/tối của hệ điều hành.
