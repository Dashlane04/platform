# Kiểm tra UI sau khi tinh giản

Ngày: 12/10/2026. Bản local: http://127.0.0.1:41810/.

## Thay đổi

- Giữ sidebar, nền sáng và xanh đậm. Dùng font hệ thống, một bộ token cho
  chữ, viền, nút và form. UI local không còn phụ thuộc CSS của bản demo.
- Bỏ slogan, emoji hero, gradient, hiệu ứng nhấc card và bóng nổi trên quân.
  Thống nhất SVG cho điều hướng, quân, căn cứ, năng lượng và kiểm soát.
- Viết lại tiêu đề, ghi chú, nhãn bảng và trạng thái bằng câu chữ chức năng.
  Giữ các thuật ngữ cần thiết như SDK, benchmark, JSON và protocol.
- Áp dụng cho khách, đăng nhập/đăng ký, đội thi, BTC, experimental, tài liệu,
  biểu đồ điểm và phòng thi đấu/replay. Dữ liệu đội, giải, luật và API không đổi.

## Kiểm tra tự động

- `npm run check`: đạt kiểm tra cú pháp JavaScript.
- `node --test tests/board.test.mjs tests/client.test.mjs tests/guest.test.mjs tests/follow.test.mjs tests/standings.test.mjs tests/experimental.test.mjs`:
  **20/20 đạt**. Chỉ cập nhật các assertion về câu chữ đã thay đổi;
  giữ kiểm tra phân quyền, lựa chọn SDK/ruleset, điểm số và diễn biến trận.
- Fixture trình duyệt `tests/refresh-preview.py`: đạt.
  Tổng quan và Bảng xếp hạng mỗi màn để yên 12 giây: không request mới,
  không mutation DOM, giữ main/shell và focus. Cập nhật thủ công thêm hàng,
  đổi trạng thái, giữ nguyên hàng cũ và focus; thứ tự mới `job-c, job-b, job-a`.

## Kiểm tra trình duyệt

Desktop dùng viewport mặc định 1280px; mobile 375 × 812px.

| Màn hình | Kết quả |
|---|---|
| Đăng nhập | Form, nhãn và nút rõ ở desktop/mobile; đăng nhập BTC và đội test thành công |
| Đăng ký | Thêm thành viên thứ tư hoạt động; không gửi hồ sơ thật |
| Đội thi | Tổng quan, nộp ZIP, benchmark, kết quả và tài liệu không tràn ngang trên mobile |
| Benchmark | Bot v3 vẫn yêu cầu chọn luật rõ ràng; ngân sách 30 giây được giữ sau khi để yên |
| BTC | Dashboard, bảng đội, hộp sửa đội và vận hành hiển thị đúng; đóng modal trả focus về nút Sửa |
| Experimental | Đổi ô vùng dùng SVG; xem thử trên server và thực hiện một nước thành công |
| Bảng điểm 13 đội | Một graph chứa đúng 10 đội; 3 đội còn lại ở bảng; graph cuộn trong vùng riêng trên mobile |
| Replay chính thức | Tải 107 trạng thái; chuyển từ nước 106 về 0 rồi 1 vẫn giữ cùng view ID và đủ 81 ô |
| Replay experimental | SVG quân/căn cứ/vùng, số quân và tài nguyên hiển thị rõ trên mobile; không tràn trang |
| Console | Không có lỗi hoặc cảnh báo trong hai tab kiểm tra ứng dụng/API |

BTC và đội thi được kiểm tra bằng bản sao dữ liệu integration test trên
`localhost:41820`, worker tắt, cookie tách khỏi `127.0.0.1:41810`.
Không sửa tài khoản, giải hoặc bộ luật người dùng đang sử dụng.
Helper: `arena-backend/tests/ui_refinement_preview.py`.
Các preview tạm được dừng sau khi kiểm tra.

## Ảnh lưu

Trong `arena-backend/test-results/`:

- `ui-before-standings.png`, `ui-after-standings.png`
- `ui-after-auth.png`, `ui-after-register.png`, `ui-after-admin.png`
- `ui-after-replay.png`
- `ui-after-mobile-auth.png`, `ui-after-mobile-standings.png`, `ui-after-mobile-replay.png`

Đây là kiểm tra hồi quy UI, không phải lần kiểm thử tải backend mới hoặc
chứng nhận accessibility đầy đủ. Backend và bản hosted chưa được triển khai lại.


## Điều chỉnh nền và chiều sâu — 2026-10-12

Theo yêu cầu mới, UI giữ màu xanh trầm và icon tối giản, thêm nền sage chuyển nhẹ, lưới bàn cờ mờ, header bán trong suốt và bóng đổ mỏng cho card. Trang đăng nhập có nền xanh đậm cùng họa tiết bàn cờ; form nằm trong card trắng. Chỉ sửa app.css; áp dụng ngay cho UI local.

Đã xem trang luyện tập desktop 1280px và mobile 375px, menu mở/đóng, đăng nhập desktop/mobile. Không tràn ngang, console không có lỗi/cảnh báo; kiểm tra JavaScript và 22 kiểm thử UI pass. Ảnh lưu tại arena-backend/test-results/ui-background-practice.png và ui-background-mobile.png. Kiểm tra dùng bản sao dữ liệu test, không sửa dữ liệu giải hay tài khoản local.
