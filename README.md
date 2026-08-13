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
Danh sách file trong `data/` được đọc từ `data/manifest.txt` — file này do GitHub
Action trong repo (`.github/workflows/manifest.yml`) **tự sinh lại mỗi lần push**,
nên thêm chỉ số mới vẫn chỉ cần push file `.txt`, không phải khai báo gì.

Nếu manifest thiếu (ví dụ Action bị tắt), trang sẽ thử tiếp GitHub API (tự nhận
diện `owner/repo` từ URL `*.github.io`; giới hạn 60 request/giờ mỗi IP), rồi tới
danh sách đã lưu của lần tải thành công gần nhất.

Hai trường hợp cần chỉnh tay khi dùng đường API dự phòng (2 hằng số ở đầu
`<script>` trong `index.html`):

- **Custom domain** (không phải `*.github.io`): điền `GITHUB_REPO = "owner/repo"`.
- **Trang không nằm ở gốc repo** (ví dụ deploy từ `docs/`): sửa `DATA_PATH`
  thành đường dẫn thật trong repo, ví dụ `"docs/data"`.

## Nhập số liệu

Toàn bộ số liệu nằm trong một file bảng: `data/kham-thai.csv`. Cách điền dễ
nhất: **mở bằng Excel / Google Sheets / LibreOffice** — file hiện thành lưới ô
thẳng hàng, mỗi lần đi khám thêm đúng **một dòng** rồi lưu lại đúng định dạng
CSV:

```csv
Ngày khám,Tuần thai,Cân nặng ước tính (g),Nhịp tim thai (lần/phút)
28/06/2026,16w1,146,152
25/07/2026,20,331,146
10/08/2026,22w2,478,-
```

- **Dòng đầu (header)**: hai cột đầu là ngày khám và tuần thai, mỗi cột sau là
  một chỉ số dạng `Tên (đơn vị)` — **thêm cột mới là web tự có thêm biểu đồ**.
- Ngày khám viết `28/06/2026` hoặc `2026-06-28`.
- Tuần thai viết `20` (chẵn tuần) hoặc `16w1` (16 tuần 1 ngày); `16w1d`,
  `16.5` cũng được. Biểu đồ vẽ theo tuần thai; ngày khám hiện ở tooltip
  và bảng số liệu.
- Ô nào chưa đo điền `-` (hoặc để trống).
- Trang đọc mọi file `.csv`/`.txt` trong `data/`. CSV nhận cả dấu phẩy lẫn
  chấm phẩy làm phân cách (Excel tiếng Việt hay xuất chấm phẩy); file `.txt`
  tách cột bằng tab như trước. Format cũ mỗi-chỉ-số-một-file vẫn đọc được.
- Số thập phân: trong file phân cách bằng phẩy thì bắt buộc dùng dấu chấm
  (`12.5`); file tab hoặc chấm phẩy thì `12,5` cũng được.

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

Quan trọng hơn: khi đã khai ngày dự sinh, **tuần thai trên biểu đồ được tính lại
từ ngày khám** (dự sinh − 280 ngày = mốc 0 tuần) thay vì lấy cột "Tuần thai"
trong file. Lý do: tuần in trên phiếu siêu âm thường lệch vài ngày giữa các lần
khám vì dự sinh được hiệu chỉnh dần theo quý I, trong khi bảng chuẩn `refs.txt`
lại tra theo tuổi thai thật — lệch mốc thì so với vùng p10–p90 sẽ sai. Cột
"Tuần thai" vẫn được giữ để đối chiếu: chỗ nào lệch, bảng và tooltip ghi thêm
"(phiếu 18w5)". Không khai dự sinh thì trang dùng nguyên cột trong file như trước.

## Khoảng tham chiếu

File `refs.txt` (cạnh `index.html`) chứa đường cong chuẩn của mọi chỉ số, chia
mục bằng dòng `# Tên chỉ số` (tên khớp với header cột trong data, bỏ phần đơn
vị); dưới mỗi mục là các dòng `tuần <tab> p10 <tab> p50 <tab> p90`. Chỉ số nào
có mục trong file này thì biểu đồ có vùng mờ p10–p90 và đường trung vị phía sau
đường đo. Giữa các tuần có trong bảng trang tự nội suy tuyến tính. Đây là bảng
tra cứu tĩnh điền một lần — **không phải cập nhật gì khi đi khám**.

Nguồn số liệu (ghi cả trong đầu file `refs.txt`):

- **Cân nặng ước tính, BPD, FL**: WHO Fetal Growth Charts (Kiserud và cộng sự,
  2017, PLoS Medicine) — percentile p10/p50/p90 theo tuần 14–40.
- **CRL**: chuẩn INTERGROWTH-21st (Papageorghiou và cộng sự, 2014).
- **Nhịp tim thai**: không có bảng percentile chuẩn duy nhất phủ cả thai kỳ —
  mục này là khoảng bình thường tổng hợp từ tài liệu lâm sàng, chỉ mang tính
  định hướng.

> ⚠️ Vùng tham chiếu chỉ để tham khảo giữa hai lần khám. Mọi kết luận về sức
> khỏe của bé luôn theo bác sĩ — bác sĩ đánh giá tổng thể nhiều chỉ số cùng
> bối cảnh lâm sàng, không nhìn một đường trên biểu đồ.

## Tính năng

- Tick bật/tắt từng chỉ số (trạng thái được nhớ lại giữa các lần mở).
- Nút ⤢ ở góc mỗi biểu đồ để phóng to xem riêng chỉ số đó (đóng bằng ✕, phím
  Esc, hoặc bấm ra vùng nền).
- Di chuột lên biểu đồ: crosshair đồng bộ mọi panel + tooltip hiện tất cả chỉ số
  tại mốc đó.
- Bảng số liệu đầy đủ ở cuối trang (mốc nào thiếu chỉ số nào hiện "–").
- Tự theo giao diện sáng/tối của hệ điều hành.
