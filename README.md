# Báo cáo phiếu mang hàng hóa ra cổng — AMECC

Dashboard HTML đọc **trực tiếp** dữ liệu từ Google Sheet mỗi lần mở trang. Không còn số liệu demo.

- **File duy nhất:** `index.html` — mở bằng trình duyệt là chạy, không cần cài đặt, không cần server.
- **Nguồn:** Google Sheet `1kI8W1hJGHu-lwvm0hlwjqHWBIBuqlaA_XDdRHp4x1LE`, tab `gid=0`.

## Điều kiện bắt buộc

Sheet phải mở quyền xem công khai: **Chia sẻ → Bất kỳ ai có đường liên kết → Người xem**.

Trang gọi Google Visualization API bằng thẻ `<script>` (JSONP) nên không vướng CORS —
chạy được cả khi mở file trực tiếp từ máy (`file://`), gửi qua Zalo/email, hay đưa lên
GitHub Pages / SharePoint.

## Cấu hình

Mở `index.html`, sửa phần đầu thẻ `<script>`:

| Hằng số | Ý nghĩa |
|---|---|
| `SHEET_ID` | ID sheet, lấy từ URL giữa `/d/` và `/edit` |
| `SHEET_GID` | `gid=` trên URL của đúng tab chứa dữ liệu |
| `AUTO_REFRESH_MINUTES` | Tự tải lại sau N phút (mặc định 10, đặt `0` để tắt) |
| `COLUMN_OVERRIDE` | Chỉ định tay tên cột nếu tự nhận diện sai |

## Ghép cột tự động

Trang tự dò cột theo **tên tiêu đề** (không phân biệt hoa thường và dấu), nên thêm/bớt/đổi
thứ tự cột trong sheet vẫn chạy đúng. Các trường cần có:

| Trường | Từ khóa tiêu đề nhận diện được |
|---|---|
| Ngày *(bắt buộc)* | ngày mang ra, ngày lập, ngày đề nghị, thời gian, ngày… |
| Đơn vị | đơn vị đề nghị, đơn vị, bộ phận, phòng ban… |
| Công trình | mã công trình, công trình, dự án, mã CT… |
| Trạng thái | trạng thái, tình trạng, kết quả… |
| Phương tiện | phương tiện, biển số, loại xe… |
| Loại hàng hóa | loại hàng hóa, tên hàng, vật tư, sản phẩm… |
| Lý do | lý do, mục đích, nội dung, diễn giải, ghi chú… |

Bấm nút **"Cột dữ liệu"** trên trang để xem đang ghép cột nào với cột nào. Nếu sai, điền tên
cột chính xác vào `COLUMN_OVERRIDE`.

Nếu sheet có dòng tiêu đề gộp ô phía trên bảng, trang tự thử lần lượt `headers=1..4` để tìm
đúng dòng tiêu đề.

**Ngày** đọc được các định dạng: ô kiểu Date của Google Sheet, `dd/mm/yyyy`, `dd-mm-yy`,
`yyyy-mm-dd`. Dòng không có ngày hợp lệ bị bỏ qua và được đếm hiển thị trên thanh trạng thái.

**Trạng thái** giữ nguyên chữ trong sheet ở bảng chi tiết, đồng thời tự quy về 3 nhóm để
tính KPI:

- *Hoàn thành* ← hoàn thành, hoàn tất, xong, đã duyệt, đã phê duyệt, đã ra cổng, done, approved…
- *Từ chối/Hủy* ← từ chối, không duyệt, hủy, reject, cancel…
- *Đang xử lý* ← còn lại (chờ phê duyệt, đang xử lý, ô trống…)

## Ô nhập số thực tế tại cổng

Sheet không có số bảo vệ đếm thực tế, nên hai ô này vẫn nhập tay:

- Ô lớn ở module **Đối soát Base Service vs. Thực tế tại cổng**
- Cột **Thực tế (nhập tay)** trong bảng đối soát theo đơn vị

Giá trị nhập được lưu trong `localStorage` của trình duyệt theo từng kỳ báo cáo nên không mất
khi tải lại trang. Khi **chưa nhập**, trang lấy thực tế = số phiếu trên Base (chênh lệch 0)
thay vì bịa ra con số ước lượng như bản demo.

## Bộ lọc

Tháng, tuần, đơn vị, công trình, trạng thái đều **sinh động từ dữ liệu thật** trong sheet —
không hardcode "Tháng 7". Mặc định mở ở tháng gần nhất có dữ liệu. Ô tìm kiếm bảng chi tiết
bỏ qua dấu tiếng Việt (gõ `thep cuon` ra `Thép cuộn`).
