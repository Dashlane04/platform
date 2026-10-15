# Khách xem giải và trận đấu

Không cần đăng ký hoặc đăng nhập để mở trang chủ, **Giải đấu**, **Lịch thi đấu**, **Trực tiếp**, **Bảng xếp hạng**, bàn Arena chính thức hoặc replay. Trang chủ có nút “Xem trận không cần đăng nhập” và danh sách giải công khai. Có thể gửi thẳng URL `/#/arena/<competition_id>/<fixture_id>` hoặc `/#/replay/<competition_id>/<fixture_id>`.

Giải trở thành công khai khi BTC mở đăng ký. Giải nháp và giải đã xóa mềm không xuất hiện trong danh sách; URL đã biết cũng trả 404 cho khách/đội. BTC có thể xem và khôi phục giải đã ẩn. Ẩn giải sẽ đóng SSE đang xem bằng event `unavailable`; UI giữ bàn đã xem và thông báo giải/trận không còn công khai. Thông tin đã được người xem tải xuống trước khi ẩn không thể thu hồi.

## Quyền đọc

Các GET sau cho phép truy cập ẩn danh:

- `/api/v1/competitions`
- `/api/v1/competitions/{id}`
- `/api/v1/competitions/{id}/fixtures`
- `/api/v1/competitions/{id}/standings`
- `/api/v1/competitions/{id}/watch` — JSON hoặc SSE theo `Accept`
- `/api/v1/competitions/{id}/fixtures/{fixture_id}/events` — JSON hoặc SSE; hỗ trợ cursor và `Last-Event-ID`

Cookie hợp lệ của BTC cho phép xem thêm giải nháp/đã ẩn; quyền này được kiểm tra lại mỗi vòng SSE. Cookie hết hạn, bị thu hồi hoặc tài khoản phải đổi mật khẩu chỉ có quyền khách trên các GET công khai. Các endpoint riêng vẫn trả 401/403 theo quy tắc cũ.

Khách chỉ thấy tên đội, version bot được chọn, lịch, kết quả, điểm, bàn cờ và clock hiển thị. Event trận được lọc bằng whitelist hiện có; không trả build log, source, stderr, báo cáo benchmark hoặc số đo tài nguyên riêng. Attempt ID trong cursor chỉ định danh luồng replay, không cấp quyền đọc `/jobs/{id}`.

Nộp bot, benchmark, luyện tập, chọn/rút bot dự giải, hồ sơ đội và mọi thao tác BTC vẫn yêu cầu tài khoản tương ứng. Mutation sau đăng nhập vẫn cần CSRF và idempotency key. Không mở CORS hay origin khác; đây là chế độ khách của UI/API cùng origin.

## Cập nhật giao diện

Bàn cờ cập nhật qua SSE tại chỗ. Arena chính thức tiếp tục tự chuyển sang trận kế tiếp; replay cố định trận người xem chọn. Trang chủ, lịch và BXH dùng nút cập nhật thủ công, không thêm reload hoặc polling toàn trang. Menu khách chỉ có các mục công khai và đăng nhập.

## Chạy lại kiểm thử tải

Trong thư mục backend trên WSL, với Docker/image worker sẵn sàng:

```sh
make guest-load-test
```

Script `tests/guest_load.py` tạo database/cổng riêng, nộp và benchmark hai bot C++ thật (72 mẫu/bot), mở giải hai lượt và dùng HTTP/SSE không cookie. Bot thử chỉ chờ 300 ms ở mỗi lượt trận để phép đo có đủ thời gian nhận nhiều nước đi; không đổi engine, ngân sách hay chính sách xử lý kết quả. Không chạm dữ liệu/tài khoản thật, không dừng Docker hoặc xóa image/volume.

Đo các mức 1/8/24 client đọc; 48 client vượt admission và phục hồi; 8 người xem (16 SSE) + 8 client API; 12 người xem (24 SSE) + 4 client API; 16 người xem (32 SSE) vượt giới hạn cùng API và phục hồi. Mỗi người xem Arena mở **hai luồng SSE**: bàn cờ và tiến độ giải. Các luồng được đọc liên tục, kiểm tra nhận nhiều trạng thái bàn cờ.

CLI hiện giới hạn 32 kết nối/task: 16 trang Arena đã có thể chiếm hết số slot, gây 503 cho kết nối mới. Đây là giới hạn hiện tại, không phải chứng nhận hỗ trợ 16 người xem đồng thời ổn định. Không tăng giới hạn chỉ để bài thử vượt qua. P95 và tỷ lệ 200/503 nằm trong `test-results/guest-load-report.md` và JSON cùng tên; throughput ở pha overload gồm cả request bị từ chối.

Phép đo ngắn trên loopback, DB nhỏ, không bao gồm browser rendering, mạng Internet, proxy/TLS hoặc soak nhiều giờ. Xem [LOAD-RESILIENCE.md](LOAD-RESILIENCE.md) cho bộ tiêm sự cố riêng đã có.
