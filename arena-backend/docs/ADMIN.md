# Điều hành UET Bot Arena

Bản cập nhật v0.6: [thể lệ xử lý kết quả v2](MATCH-POLICY.md), hàng đợi ưu tiên official, cột Hòa trong bảng xếp hạng/CSV và chẩn đoán startup. Giải đã tồn tại giữ thể lệ v1; tạo giải mới để dùng v2. Máy local hiện dùng **http://127.0.0.1:41810/** do Windows dành riêng cổng 4181.

Bản local v0.6: mở **http://127.0.0.1:4181/** và đăng nhập tài khoản có role `admin`. Tài khoản đội không thấy menu quản trị và bị API từ chối truy cập các endpoint admin. Tài khoản admin hiện có tiếp tục dùng được sau migration; không có mật khẩu admin mặc định. Nếu cài mới, dừng controller rồi dùng:

```bash
.venv/bin/python -m arena_api bootstrap-admin --username organizer
make serve
```

CLI hỏi mật khẩu ẩn. Không tạo admin bằng form đội thi. Nếu controller đã chạy thì không chạy bootstrap/migrate offline cùng data directory. `make serve` tự áp dụng migration còn thiếu.

## Các màn hình

| Màn hình | Thao tác |
|---|---|
| `#/home` | Dashboard: đội/hồ sơ/giải, trạng thái worker, queue, job theo trạng thái, hoạt động 7 ngày, tỷ lệ benchmark đạt và P95 cao nhất 24 giờ, nhóm mã lỗi. |
| `#/admin` | Tạo đội kèm tài khoản; xem/sửa tên, email, điện thoại và 3–4 thành viên; tìm kiếm, phân trang, CSV; khóa/mở khóa, cấp lại mật khẩu, đánh dấu đã phát; xóa mềm/khôi phục. |
| `#/admin?tab=registrations` | Xem hồ sơ chờ/đã duyệt/từ chối; duyệt và cấp tài khoản hoặc từ chối. |
| `#/admin?tab=recoveries` | Cấp lại mật khẩu hoặc đánh dấu yêu cầu đã xử lý mà không đổi mật khẩu. Cấp lại mật khẩu tự đóng các yêu cầu của tài khoản. |
| `#/competitions` | Tạo/sửa/xóa mềm/khôi phục giải; mở đăng ký, loại entry trước khi chốt, chốt bot/sinh lịch hai lượt, bắt đầu/tạm dừng/tiếp tục. |
| `#/operations` | Tất cả job: tìm kiếm/lọc đội, trạng thái và loại; mở report/log, tải JSON. Tab source xem metadata/manifest của các phiên bản đã nộp. |
| `#/schedule`, `#/live`, `#/standings` | Xem lịch/live/replay/bảng thắng thua. Với trận lỗi, xem từng attempt và chạy lại từ giải; giữ attempt cũ. CSV bảng xếp hạng. |
| `#/audit` | Thời gian, actor ID, thao tác và đối tượng. Không ghi mật khẩu hoặc nội dung source. |

Dashboard và danh sách vận hành chỉ lấy dữ liệu khi mở trang, thực hiện thao tác hoặc nhấn Cập nhật dữ liệu / Làm mới. Không có timer tải dữ liệu định kỳ. Cập nhật thủ công đối chiếu DOM trong main, giữ shell/sidebar, focus và vị trí cuộn. Probe Docker được cache 10 giây; mất Docker hiển thị trạng thái cần kiểm tra và API từ chối tạo job, không tự chạy source trên host.

Tỷ lệ đạt: benchmark tạo trong 24 giờ gần nhất đã kết thúc; mẫu số gồm completed/failed/aborted. P95 hiển thị là **giá trị cao nhất trong các P95 của benchmark hoàn tất** tạo trong cùng khoảng; không phải percentile gộp toàn bộ lượt. Biểu đồ tính job theo ngày tạo ở Asia/Bangkok; không đếm số lượt bot. Không có mẫu thì hiện `—`.

## Quy trình một giải

1. Duyệt hồ sơ hoặc thêm đội trực tiếp. Sao chép mật khẩu tạm ngay khi được trả về; phát qua kênh của BTC rồi đánh dấu đã phát. Đội bắt buộc đổi mật khẩu khi đăng nhập đầu tiên.
2. Tạo giải, chọn ngân sách 15/30/60/300 giây và tối đa 2–32 đội. Có thể đổi thời gian khi còn bản nháp chưa có entry. Sau khi mở đăng ký, chỉ sửa tên và sức chứa trước khi chốt; sức chứa phải đủ entry hiện có.
3. Các đội nộp ZIP và chạy benchmark 72 mẫu đúng cấu hình giải. Mỗi đội tự chọn source/benchmark của mình để tham gia. Admin không sửa binary hoặc giả lập chứng nhận benchmark.
4. Kiểm tra entry rồi chốt. Từ lúc chốt, không cho sửa cấu hình hoặc xóa giải. Bắt đầu để scheduler chạy; tạm dừng chỉ dừng ghép trận tiếp, job đã nhận tiếp tục hoàn tất.
5. Trận lỗi hạ tầng chuyển cần xử lý, chưa tính điểm. Xem attempt và retry; chỉ attempt hợp lệ được tính đúng một lần. Restart controller tạm dừng giải và không tự retry.
6. Xuất bảng xếp hạng. Có thể xóa mềm giải đã hoàn tất; lịch, replay và kết quả vẫn giữ.

## Xóa, khôi phục và sửa đồng thời

- Xóa đội: khóa tài khoản, thu hồi mọi session, ẩn khỏi danh sách đang quản lý. Chặn nếu đội có job chưa kết thúc hoặc entry trong giải chưa hoàn tất. Gỡ entry khi giải còn mở, hoặc hoàn tất giải đã chốt trước.
- Khôi phục đội: giữ tài khoản khóa; admin kiểm tra rồi chủ động mở khóa. Mật khẩu cũ vẫn dùng nếu chưa được cấp lại. Tên và username của đội đã xóa vẫn được giữ, không tái sử dụng nhầm lịch sử.
- Xóa giải: cho phép draft/open chưa có entry, hoặc completed. Bản đã xóa không xuất hiện trong danh sách giải của đội; lịch sử hoàn tất vẫn truy cập qua liên kết cũ. Khôi phục giữ trạng thái trước khi xóa.
- Không xóa source, job, attempt hoặc kết quả thi đấu. Không có thao tác gán thắng/thua thủ công.
- Sửa/xóa/khôi phục gửi `expected_revision`. Nếu admin khác đã thay đổi dữ liệu, API trả `409 admin_conflict`; làm mới rồi xem lại trước khi gửi. Thao tác giải còn có `Idempotency-Key` để retry không lặp.
- Mật khẩu chỉ trả trong response cấp mới/cấp lại, lưu hash Argon2id; không có API đọc lại. Các mutation admin yêu cầu session, role và CSRF.

## API mới

`/api/v1/admin/teams` (POST); `/teams/{id}` (GET); `/teams/{id}/edit`, `/archive`, `/restore` (POST); `/metrics`, `/jobs`, `/jobs/{id}`, `/submissions` (GET); `/recoveries/{id}/resolve` (POST). Sửa/xóa/khôi phục giải: `/api/v1/admin/competitions/{id}/edit|archive|restore` (POST). Contract đầy đủ ở `/docs` và `docs/openapi.json`.

API danh sách job/submission hỗ trợ `limit` (tối đa 100) và `offset`. Job còn có filter `status`, `kind`, `team_id`. UI local tải metadata phân trang từ API rồi lọc, hiển thị 20 mục/trang; report đầy đủ chỉ tải khi mở chi tiết. Bản local phù hợp quy mô giải đầu tiên; chưa bổ sung hạ tầng giám sát nhiều worker hoặc phân vai BTC chi tiết.

## Kiểm tra riêng, không ảnh hưởng dữ liệu thật

```bash
make test
npm.cmd --prefix ../uet-bot-arena run check # PowerShell; trên Linux dùng npm
make integration # Cần Docker/image uet-arena-worker:v1
.venv/bin/python tests/admin_preview.py
```

Fixture cuối in URL cổng tạm, dùng DB tạm, tài khoản `organizer` / `preview-only-admin-password`. Có 23 đội, hồ sơ chờ và job **dữ liệu giả có nhãn**, không chạy worker; chỉ phục vụ kiểm tra browser. Dừng bằng Ctrl+C để dọn DB tạm. Không dùng fixture làm server vận hành.
