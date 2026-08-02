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

## Ghép cột

Đã khóa sẵn theo tab **"Phiếu ra vào cổng"** (18 cột). Bảng ghép:

| Trường trên báo cáo | Cột sheet | Dùng ở đâu |
|---|---|---|
| Ngày | `ngay_tao` | trục thời gian, bộ lọc tháng/tuần, KPI |
| Đơn vị | `don_vi_de_nghi` | KPI, biểu đồ Top đơn vị, bảng đối soát |
| Công trình | `cong_trinh_du_an` | bộ lọc, biểu đồ Top công trình |
| Trạng thái | `trang_thai` | KPI Hoàn thành / Đang xử lý / Từ chối |
| Phương tiện | `phuong_tien` | bảng chi tiết |
| Loại hàng hóa | `loai_hang_hoa` | bảng chi tiết |
| Lý do | `ly_do` | tìm kiếm, tooltip |

Các cột phụ — có thì dùng, không có thì tự ẩn:

| Cột sheet | Dùng ở đâu |
|---|---|
| `khoi_hien_tai` | bộ lọc **Khối** |
| `phan_loai_hang_hoa` | biểu đồ tròn "Theo loại hàng hóa" (thay `loai_hang_hoa` vì đây là danh mục chuẩn, không phải chữ tự do) |
| `bien_kiem_soat` | cột **Biển KS** trong bảng chi tiết |
| `gio_de_nghi_mang_ra` / `_mang_vao` | cột **Giờ ra → vào** trong bảng chi tiết |
| `phieu_id`, `ten_phieu`, `hang_muc`, `dia_diem_den`, `nguoi_dieu_khien`, `quan_ly_phu_trach` | ô tìm kiếm + tooltip khi rê chuột vào dòng |

Ba tab còn lại (`Công trình dự án`, `Hạng mục`, `Chi tiết hàng hóa`) là bảng con nối theo
`phieu_id`, hiện **chưa dùng** — báo cáo lấy đủ dữ liệu từ tab chính.

Bấm nút **"Cột dữ liệu"** trên trang để xem đang ghép cột nào với cột nào. Nếu sai, sửa
`COLUMN_OVERRIDE`; để trống `''` thì trang tự dò theo từ khóa.

### Đổi mốc thời gian

Hằng số `DATE_FIELD` quyết định phiếu được xếp vào ngày nào:

- `'d'` *(đang dùng)* → `ngay_tao`, ngày lập phiếu
- `'gra'` → `gio_de_nghi_mang_ra`, giờ đề nghị mang hàng ra cổng

Nếu sheet có dòng tiêu đề gộp ô phía trên bảng, trang tự thử lần lượt `headers=1..4` để tìm
đúng dòng tiêu đề.

**Ngày** đọc được các định dạng: ô kiểu Date/DateTime của Google Sheet, `dd/mm/yyyy`,
`dd-mm-yy`, `yyyy-mm-dd`, và dạng có kèm giờ như `09:00 30/07/2026`. Dòng không có ngày hợp
lệ bị bỏ qua và được đếm hiển thị trên thanh trạng thái.

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
