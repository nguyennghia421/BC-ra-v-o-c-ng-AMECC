# Báo cáo phiếu mang hàng hóa ra cổng — AMECC

Dashboard HTML một trang, đọc **trực tiếp** dữ liệu từ Google Sheet mỗi lần mở.

- **File duy nhất:** `index.html` — mở bằng trình duyệt là chạy, không cần cài đặt, không cần server.
- **Nguồn:** Google Sheet `1Y2PzDgiw4NRXyHt10_gkGWL27YrdA3F1sk-9AoxPZik`, tab `gid=0`.

Trang gồm: 4 thẻ KPI (tổng phiếu, hoàn thành, đang xử lý, từ chối/hủy), module đối soát
Base Service vs. thực tế tại cổng, và bộ lọc theo tuần.

## Điều kiện bắt buộc

Sheet phải mở quyền xem: **Chia sẻ → Bất kỳ ai có đường liên kết → Người xem**.

Trang gọi Google Visualization API bằng thẻ `<script>` (JSONP) nên không vướng CORS —
chạy được cả khi mở file trực tiếp từ máy (`file://`), gửi qua Zalo/email, hay đưa lên
GitHub Pages / SharePoint.

## Cấu hình — đầu thẻ `<script>`

| Hằng số | Ý nghĩa |
|---|---|
| `DATA_MODE` | `'live'` đọc Google Sheet · `'mock'` dùng dữ liệu minh họa để demo không cần mạng |
| `SHEET_ID` | ID sheet, lấy từ URL giữa `/d/` và `/edit` |
| `SHEET_GID` | `gid=` trên URL của đúng tab chứa dữ liệu |
| `AUTO_REFRESH_MINUTES` | Tự tải lại sau N phút (mặc định 10, đặt `0` để tắt) |
| `DATE_FIELD` | `'d'` xếp phiếu theo `ngay_tao` · `'gra'` theo `gio_de_nghi_mang_ra` |
| `COLUMN_OVERRIDE` | Tên cột chỉ định tay; để trống `''` thì luôn tự dò |
| `USE_CHILD_TABS` | Đọc thêm tab `Công trình dự án` / `Hạng mục`. Mặc định `false` |

Ở chế độ `mock`, trang hiện nhãn **"● DỮ LIỆU MINH HỌA (DEMO)"** dưới tiêu đề để không ai
nhầm với số thật.

## Ghép cột

Trang tự dò cột theo **tên tiêu đề** (bỏ dấu, không phân biệt hoa thường), nên thêm/bớt/đổi
thứ tự cột trong sheet vẫn chạy. `COLUMN_OVERRIDE` chỉ là gợi ý: tên nào khớp thì dùng luôn,
không khớp thì rơi về tự dò.

Bốn trường thực sự ảnh hưởng tới con số trên trang:

| Trường | Bắt buộc | Từ khóa tiêu đề nhận diện được |
|---|---|---|
| Ngày | ✅ | `ngay_tao`, ngày ra, ngày lập, ngày đề nghị, thời gian, ngày… |
| Trạng thái | | `trang_thai`, tình trạng, status, kết quả |
| Đơn vị | | `don_vi_de_nghi`, đơn vị, bộ phận, phòng ban… |
| Công trình | | `cong_trinh_du_an`, công trình, dự án, mã CT… |

Các cột còn lại (`loai_hang_hoa`, `phuong_tien`, `ly_do`, `bien_kiem_soat`,
`nguoi_dieu_khien`, `hang_muc`, `dia_diem_den`, `phan_loai_hang_hoa`,
`quan_ly_phu_trach`, `gio_de_nghi_mang_ra/vao`) vẫn được đọc và giữ trong bộ nhớ, nhưng bố
cục hiện tại **chưa hiển thị** — sẵn sàng khi thêm bảng chi tiết hoặc biểu đồ trở lại.

**Kiểm tra ghép cột:** mở trang → **F12** → tab **Console** → mở nhóm
`[AMECC] Ghép cột Google Sheet → báo cáo`. Bảng ở đó cho biết trường nào đang lấy từ cột nào.

Nếu sheet có dòng tiêu đề gộp ô phía trên bảng, trang tự thử `headers=1..4` để tìm đúng dòng
tiêu đề.

**Ngày** đọc được: ô Date/DateTime của Google Sheet, `dd/mm/yyyy`, `dd-mm-yy`, `yyyy-mm-dd`,
và dạng kèm giờ như `09:00 30/07/2026`. Dòng không có ngày hợp lệ bị bỏ qua, số dòng bỏ qua
được cảnh báo trong Console.

**Trạng thái** tự quy về 3 nhóm để tính KPI:

- *Hoàn thành* ← hoàn thành, hoàn tất, xong, đã duyệt, đã phê duyệt, đã ra cổng, done, approved…
- *Từ chối/Hủy* ← từ chối, không duyệt, hủy, reject, cancel…
- *Đang xử lý* ← còn lại (chờ phê duyệt, đang xử lý, ô trống…)

## Bộ lọc tuần

Dùng **tuần lịch ISO-8601 thật** — thứ Hai là ngày đầu tuần, đúng như lịch treo tường, nên
một tuần có thể vắt qua hai tháng. Nhãn ghi rõ khoảng ngày, ví dụ `Tuần 30 · 20/07 – 26/07`.
Chỉ những tuần thực sự có phiếu mới xuất hiện. Lần đầu mở trang tự chọn tuần gần nhất có dữ liệu.

## Ô nhập số thực tế tại cổng

Sheet không có số bảo vệ đếm thực tế nên ô này nhập tay, và **chỉ sống trong phiên xem hiện
tại** — không ghi vào `localStorage` hay server, tải lại trang là mất. Đây là ô xem thử để
ước lượng nhanh mức chênh lệch, không phải nơi lưu số liệu chính thức.

Khi chưa nhập, ở chế độ `live` trang lấy thực tế = số phiếu trên Base (chênh lệch 0) chứ
không bịa số. Chỉ chế độ `mock` mới sinh một số lệch mẫu để minh họa cách đối soát hoạt động.

## Tab con nối theo `phieu_id` (đang tắt)

Code đọc hai tab `Công trình dự án` và `Hạng mục` vẫn còn nguyên, cho phép một phiếu thuộc
nhiều công trình / hạng mục. Bố cục hiện tại không có biểu đồ công trình hay bảng chi tiết
nên `USE_CHILD_TABS = false` để bớt 2 lượt gọi mạng mỗi lần tải.

Bật lại thành `true` nếu thêm các panel đó. Khi bật: phiếu có dòng ở tab con thì lấy danh
sách từ tab con, không có thì lùi về cột tương ứng ở tab chính; tab con lỗi hoặc rỗng thì
báo cáo vẫn chạy bằng tab chính. Tên tab trong `CHILD_TABS` phải khớp **chính xác** với tên
hiển thị dưới đáy Google Sheet.
