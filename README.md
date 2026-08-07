# Báo cáo phiếu mang hàng hóa ra cổng — AMECC

Dashboard HTML một trang, đọc **trực tiếp** dữ liệu từ Google Sheet mỗi lần mở, các biểu đồ
**lọc chéo lẫn nhau như Power BI**.

- **File duy nhất:** `index.html` — mở bằng trình duyệt là chạy, không cần cài đặt, không cần server.
- **Nguồn:** Google Sheet `1Y2PzDgiw4NRXyHt10_gkGWL27YrdA3F1sk-9AoxPZik`
  - tab **`Phiếu xe`** — mỗi dòng là một phiếu mang hàng hóa ra cổng
  - tab **`PX_Dự án`** — bảng nối `phiếu ↔ công trình/dự án` (1 phiếu thuộc nhiều dự án ⇒ nhiều dòng cùng mã phiếu)

Trang gồm: 5 thẻ KPI, module đối soát Base Service vs. thực tế tại cổng, đường xu hướng theo
ngày, biểu đồ đơn vị đề nghị, bảng loại hàng hóa, bảng công trình/dự án và bảng chi tiết phiếu.

## Điều kiện bắt buộc

Sheet phải mở quyền xem: **Chia sẻ → Bất kỳ ai có đường liên kết → Người xem**.

Trang gọi Google Visualization API bằng thẻ `<script>` (JSONP) nên không vướng CORS —
chạy được cả khi mở file trực tiếp từ máy (`file://`), gửi qua Zalo/email, hay đưa lên
GitHub Pages / SharePoint.

## Lọc chéo kiểu Power BI

Bốn ô chọn ở đầu trang (kỳ / đơn vị / công trình / trạng thái) và các visual **dùng chung một
bộ lọc**. Bấm vào một cột, một dòng bảng hay một điểm trên đường xu hướng là đặt thêm một lát
cắt; bấm lại đúng chỗ đó để bỏ.

| Bấm vào | Lọc theo |
|---|---|
| Cột trên *Số phiếu ra theo đơn vị đề nghị* | Đơn vị (đồng bộ luôn ô chọn Đơn vị) |
| Dòng bảng *Top loại hàng hóa ra* | Loại hàng hóa |
| Dòng bảng *Top công trình / dự án* | Công trình (đồng bộ luôn ô chọn Công trình) |
| Điểm trên *Xu hướng số phiếu theo ngày* | Một ngày cụ thể |

Các lát cắt đang bật hiện thành chip ở thanh **"Đang lọc chéo"** ngay dưới tiêu đề, kèm nút ✕
cho từng chip và nút xóa tất cả.

Mỗi visual tính số liệu bằng **tất cả bộ lọc TRỪ chiều của chính nó** — đúng cách Power BI xử
lý. Bấm một đơn vị thì biểu đồ đơn vị vẫn hiện đủ các đơn vị (cột được chọn sáng, còn lại mờ
đi) để còn so sánh được, trong khi KPI, xu hướng, hai bảng và donut đối soát đều đã lọc theo
đơn vị đó. Cột `Khác` trên biểu đồ đơn vị là nhóm gộp nên không bấm lọc được.

## Cấu hình — đầu thẻ `<script>`

| Hằng số | Ý nghĩa |
|---|---|
| `SHEET_ID` | ID sheet, lấy từ URL giữa `/d/` và `/edit` |
| `MAIN_SHEET` | Tên tab chính — mặc định `Phiếu xe`, phải khớp **chính xác** tên hiển thị dưới đáy Google Sheet |
| `MAIN_GID` | Đặt `gid` nếu muốn đọc theo gid thay vì theo tên tab; để trống `''` thì đọc theo `MAIN_SHEET` |
| `PROJECT_TAB.sheet` | Tên tab nối dự án — mặc định `PX_Dự án` |
| `AUTO_REFRESH_MINUTES` | Tự tải lại sau N phút (mặc định 10, đặt `0` để tắt) |
| `DATE_FIELD` | `'d'` xếp phiếu theo `ngay_tao` · `'gra'` theo `gio_de_nghi_mang_ra` |
| `DETAIL_LIMIT` | Số dòng tối đa dựng ra ở bảng chi tiết (mặc định 400) |
| `COLUMN_OVERRIDE` | Tên cột chỉ định tay; để trống `''` thì luôn tự dò |

Nút **⟳ Làm mới** ở góc phải tiêu đề tải lại ngay, cạnh đó là thời điểm cập nhật gần nhất và
số phiếu / số dòng nối đọc được.

## Ghép cột

Trang tự dò cột theo **tên tiêu đề** (bỏ dấu, không phân biệt hoa thường), nên thêm/bớt/đổi
thứ tự cột trong sheet vẫn chạy. `COLUMN_OVERRIDE` chỉ là gợi ý: tên nào khớp thì dùng luôn,
không khớp thì rơi về tự dò.

| Trường | Bắt buộc | Từ khóa tiêu đề nhận diện được |
|---|---|---|
| Ngày | ✅ | `ngay_tao`, ngày ra, ngày lập, ngày đề nghị, thời gian, ngày… |
| Mã phiếu | ⚠️ | `phieu_id`, mã phiếu, số phiếu — **cần có để nối được với `PX_Dự án`** |
| Trạng thái | | `trang_thai`, tình trạng, status, kết quả |
| Đơn vị | | `don_vi_de_nghi`, đơn vị, bộ phận, phòng ban… |
| Loại hàng hóa | | `loai_hang_hoa`, tên hàng, vật tư, sản phẩm… |
| Công trình | | `cong_trinh_du_an`, công trình, dự án, mã CT… |
| Phương tiện | | `phuong_tien`, loại xe, số xe |

Các cột còn lại (`ly_do`, `bien_kiem_soat`, `nguoi_dieu_khien`, `hang_muc`, `dia_diem_den`,
`phan_loai_hang_hoa`, `quan_ly_phu_trach`, `gio_de_nghi_mang_ra/vao`) vẫn được đọc và giữ
trong bộ nhớ; ô tìm kiếm ở bảng chi tiết có tra cả biển kiểm soát và người điều khiển.

**Kiểm tra ghép cột:** mở trang → **F12** → tab **Console** → mở nhóm
`[AMECC] Ghép cột Google Sheet → báo cáo`. Bảng ở đó cho biết trường nào đang lấy từ cột nào,
kèm số dòng nối đọc được từ `PX_Dự án` và cảnh báo nếu tab đó lỗi.

Nếu sheet có dòng tiêu đề gộp ô phía trên bảng, trang tự thử `headers=1..4` để tìm đúng dòng
tiêu đề.

**Ngày** đọc được: ô Date/DateTime của Google Sheet, `dd/mm/yyyy`, `dd-mm-yy`, `yyyy-mm-dd`,
và dạng kèm giờ như `09:00 30/07/2026`. Dòng không có ngày hợp lệ bị bỏ qua, số dòng bỏ qua
được cảnh báo trong Console.

**Trạng thái** tự quy về 3 nhóm để tính KPI:

- *Hoàn thành* ← hoàn thành, hoàn tất, xong, đã duyệt, đã phê duyệt, đã ra cổng, done, approved…
- *Từ chối/Hủy* ← từ chối, không duyệt, hủy, reject, cancel…
- *Đang xử lý* ← còn lại (chờ phê duyệt, đang xử lý, ô trống…)

## Tab `PX_Dự án`

Trang tự dò cột mã phiếu (tiêu đề chứa `phiếu`) và cột dự án (`công trình`, `dự án`…), gom
thành `{ mã phiếu: [dự án, …] }` rồi gắn vào từng phiếu.

- Phiếu **có** dòng ở `PX_Dự án` → lấy danh sách dự án từ tab này.
- Phiếu **không có** dòng nào → lùi về cột công trình ngay trên tab `Phiếu xe`.
- Tab `PX_Dự án` lỗi, rỗng, hoặc tab chính không có cột mã phiếu → báo cáo vẫn chạy bằng tab
  chính, lý do ghi ở Console.

Vì một phiếu có thể thuộc nhiều dự án, **bảng Top công trình / dự án đếm theo lượt xuất hiện**:
phiếu thuộc 2 dự án được đếm ở cả 2 dòng, nên dòng TỔNG của bảng này có thể lớn hơn số phiếu ở
KPI. Các KPI, donut đối soát và bảng chi tiết luôn đếm **số phiếu duy nhất**.

## Bộ lọc kỳ

Dùng **tuần lịch ISO-8601 thật** — thứ Hai là ngày đầu tuần, đúng như lịch treo tường, nên
một tuần có thể vắt qua hai tháng. Nhãn ghi rõ khoảng ngày, ví dụ `Tuần 30 · 20/07 – 26/07`.
Chỉ những tuần thực sự có phiếu mới xuất hiện. Lần đầu mở trang tự chọn tuần gần nhất có dữ liệu.

KPI so sánh với **tuần liền trước**, áp dụng cùng các lát cắt đang bật để so đúng "táo với táo".
Chọn kỳ `Toàn bộ` hoặc đang lọc theo 1 ngày cụ thể thì phần so sánh tắt đi.

## Ô nhập số thực tế tại cổng

Sheet không có số bảo vệ đếm thực tế nên ô này nhập tay, và **chỉ sống trong phiên xem hiện
tại** — không ghi vào `localStorage` hay server, tải lại trang là mất. Đây là ô xem thử để
ước lượng nhanh mức chênh lệch, không phải nơi lưu số liệu chính thức.

Khi chưa nhập, trang lấy thực tế = số phiếu trên Base (chênh lệch 0) chứ **không bịa số**.
