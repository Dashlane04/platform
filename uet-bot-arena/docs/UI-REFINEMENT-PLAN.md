# Kế hoạch tinh giản UI UET Bot Arena

Ngày: 12/10/2026. Trạng thái: đã áp dụng và kiểm tra trên UI local WSL.

Kết quả kiểm tra: [UI-REFINEMENT-VALIDATION.md](UI-REFINEMENT-VALIDATION.md).

## Mục tiêu

Giữ nhận diện xanh đậm, nền sáng, sidebar và bố cục dashboard hiện tại.
Giao diện tập trung vào giải đấu, bot và trận đấu; giảm trang trí và câu chữ
quảng bá. Toàn bộ màn hình đội, khách và BTC dùng cùng một hệ thống thành phần.

## Những điểm đã thấy trước khi chỉnh sửa

- `live/pages.js`: slogan ở trang chủ, lời chào và tiêu đề đăng nhập/đăng ký;
  emoji bàn tay lớn và nhiều tiêu đề có dấu chấm không cần thiết.
- `live/view.js`: tagline CODE. COMPETE. CONNECT., đoạn giới thiệu nhiều dòng
  ở auth, icon khiên dùng cho mọi ghi chú, icon mũi tên ở hầu hết liên kết.
- `live/app.css`: lime nổi trên hero/auth, họa tiết lưới/khối hình trang trí,
  hover nhấc card và nhiều nền xanh nhạt lặp lại.
- Thành phần KPI, card, badge và bảng có nhiều cấp cỡ chữ/khung riêng;
  một số nhãn và nút rất nhỏ. Cần thống nhất khi tinh giản.
- Bàn cờ đã có SVG quân và lịch sử ăn quân hữu ích; tiếp tục cải thiện độ
  nhất quán, không thay về chữ hoặc emoji.

## Quy tắc thiết kế

### Màu và bề mặt

- Giữ xanh đậm làm màu thương hiệu và nút chính. Giảm lime ở hero/auth;
  dùng nền trắng hoặc xanh nhạt trung tính cho vùng được chọn.
- Màu đỏ/xanh của hai đội, lỗi/cảnh báo và các lớp vùng trên bàn cờ vẫn cần
  phân biệt rõ. Tối giản trang trí không làm mất thông tin.
- Card dùng viền mảnh, shadow rất nhẹ hoặc không có. Bỏ gradient và khối
  hình chỉ để trang trí. Không dùng hiệu ứng nhấc card khi hover.
- Một nhóm thao tác chỉ có một nút chính khi có thể xác định rõ hành động.
  Nút sửa/xem phụ dùng kiểu nhẹ hơn, nút xóa có nhãn và màu riêng.

### Chữ và bố cục

- Giữ font hệ thống đang dùng. Chuẩn hóa cỡ chữ, độ đậm, khoảng cách và
  padding qua token; hạn chế chữ HOA và letter-spacing cho nội dung thường.
- Tiêu đề gọi đúng chức năng; dòng phụ chỉ có khi thêm thông tin hữu ích.
- Các màn hình thao tác ưu tiên form/bảng; giảm card lồng card và banner.
- KPI ưu tiên con số và nhãn, không bắt buộc mỗi chỉ số phải có một icon lớn.
- Nút trên desktop vẫn dễ bấm; mobile có kích thước chạm phù hợp, bảng
  rộng cuộn trong vùng riêng, không gây tràn toàn trang.

### Biểu tượng và ảnh

- Một bộ SVG nét đơn cho navigation, nút và trạng thái, đồng nhất stroke,
  kích thước và căn chỉnh. Không thêm ảnh minh họa AI hay icon 3D.
- Bỏ emoji lớn trên hero. Có thể dùng một bàn cờ nhỏ dựng bằng chính
  component của sản phẩm nếu thật sự có ích; ưu tiên không có minh họa.
- Quân Búa/Lá/Kéo dùng SVG rõ nét, không gradient, không đổ bóng nổi mạnh.
  Giữ tên/tooltip và các dấu hiệu phân biệt ngoài màu sắc.
- Chuẩn hóa icon Khiên, năng lượng và kiểm soát. Có chú giải ngắn ở màn
  experimental; không phủ nhiều hiệu ứng phát sáng lên quân.
- Icon trang trí có thể bỏ. Icon chức năng phải có nhãn hoặc accessible name.

## Nội dung

Viết như một công cụ dành cho sinh viên và BTC: ngắn, rõ, lịch sự. Không dùng
khẩu hiệu, câu hỏi gợi cảm hứng, dấu chấm ngắt câu để tạo nhịp hoặc tiếng Anh
trang trí. Giữ thuật ngữ cần thiết như SDK, benchmark, ruleset, C++17 và JSON;
giải thích ở đúng nơi người dùng cần quyết định.

| Hiện tại | Đề xuất |
|---|---|
| Sẵn sàng cho nước tiếp theo? | Đăng nhập |
| Bắt đầu cùng đồng đội. | Đăng ký đội |
| Hẹn gặp đội bạn tại đấu trường. | Đã gửi hồ sơ đăng ký |
| Đấu trường của những ý tưởng. | UET Bot Arena |
| Một ý tưởng mới. Một nước đi hay. | Thi đấu bot OTTv2 |
| Chào [đội], sẵn sàng thi đấu? | Tổng quan |
| Chốt bot. Đổi màu. Thi đấu công bằng. | Bỏ dòng này; thông tin thể thức đặt trong chi tiết giải |
| Bắt đầu từ một chiến thuật. | Tài liệu & thể lệ |
| CODE. COMPETE. CONNECT. | Bỏ tagline |
| Xem job lỗi | Xem tác vụ lỗi |

Nhãn phiên bản giao thức cần tách rõ luật OTTv2 và SDK v2/v3 để tránh nhầm.
Một số thuật ngữ vận hành có thể có chú thích tiếng Anh trong chi tiết kỹ thuật.
Lỗi theo cấu trúc: việc gì xảy ra → nguyên nhân đã biết → cách xử lý.
Không đoán nguyên nhân khi log chưa đủ. Thông tin SDK/ruleset/deadline vẫn giữ
đầy đủ, nhưng chuyển phần diễn giải dài vào mục chi tiết hoặc tài liệu.

## Phạm vi từng màn hình

1. Shell: đơn giản hóa brand/sidebar/footer; giảm thông tin lặp; navigation
   active rõ, một mức icon. Giữ trạng thái kết nối ở mức nhỏ và dễ hiểu.
2. Đăng nhập/đăng ký: giữ xanh đậm làm nhận diện, bỏ đoạn slogan lớn và hình
   trang trí; ưu tiên tiêu đề, form và thông tin cấp tài khoản.
3. Trang chủ/Tổng quan: thông tin giải, việc cần làm và hoạt động gần đây;
   guest thấy giải công khai và nút xem trận, không cần hero quảng cáo lớn.
4. Nộp bot/Benchmark/Chi tiết tác vụ: form và kết quả rõ ràng, nhãn SDK/luật
   dễ nhận biết; hint thường dạng text, chỉ lỗi/cảnh báo dùng khối nổi bật.
5. Giải/Lịch/BXH: giữ top 10 trong một biểu đồ ngang chung, phần còn lại là
   bảng; giảm lưới và màu phụ. Giữ nguyên công thức 3–1–0, phân phần kết quả,
   đồng hạng và dữ liệu. Thua 0 điểm không được vẽ thành độ dài điểm dương.
6. Live/Replay: bàn cờ là trọng tâm; đồng hồ, lượt, số quân và diễn biến đứng
   sau. Làm gọn viền/badge, chuẩn hóa icon ăn quân/kỹ năng, không che bàn.
7. BTC: dashboard metrics và bảng quản lý đồng bộ với UI đội; ưu tiên hàng
   đợi, lỗi cần xử lý và thao tác. Experimental vẫn gắn nhãn rõ, không dùng
   màu tím/neon hoặc phong cách riêng tách khỏi hệ thống.
8. Tài liệu: tiêu đề, đoạn văn, danh sách, code và bảng có phân cấp rõ;
   thuật ngữ và tên nút khớp chính xác với UI.

## Thứ tự thực hiện

### Đợt 1 — Thành phần chung và câu chữ

Chốt token màu/chữ/khoảng cách, chuẩn hóa button/form/table/badge/note/icon.
Áp dụng trước cho shell và Tổng quan để có màn hình đại diện. Rà toàn bộ
text, cả lỗi, trạng thái rỗng, modal và thông báo thành công.

### Đợt 2 — Toàn bộ màn hình

Lần lượt: auth → nộp/benchmark → giải/BXH → live/replay → BTC/experimental
→ tài liệu. Mỗi nhóm dùng lại thành phần chung; tránh một lớp CSS override
mới chồng lên toàn bộ giao diện cũ.

### Đợt 3 — Kiểm tra hoàn thiện

- Desktop và mobile, dùng tên đội dài, nhiều hàng, điểm 0 và trạng thái rỗng.
- Loading/error/success/disabled, focus bàn phím, nhãn screen reader và độ
  tương phản. Không dùng màu làm dấu hiệu duy nhất.
- Guest, đội và BTC; đăng nhập, nộp ZIP, chọn SDK/ruleset, benchmark, vào
  giải, live và replay. Giữ patch bàn cờ/SSE, không đưa reload định kỳ trở lại.
- Kiểm tra syntax và các test UI liên quan; điều chỉnh test text khi wording
  thay đổi, giữ nguyên các kiểm tra hành vi. Chụp ảnh trước/sau màn hình đại diện.

## Điều kiện hoàn tất

- Nhận diện xanh đậm và cấu trúc điều hướng vẫn quen thuộc.
- Không còn slogan/quote trang trí, emoji hero hoặc hình minh họa thừa.
- Icon đồng nhất; màu nổi phục vụ trạng thái hoặc dữ liệu.
- Các màn hình và trạng thái phụ dùng cùng component và cách diễn đạt.
- Số liệu, hành vi và cập nhật live giữ đúng; không đổi API, luật thi hay DB.
- Hoàn thiện bản local trên WSL trước theo phạm vi hiện tại.
