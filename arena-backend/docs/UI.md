# UI + API local trên WSL — v0.4

UI thực chạy tại **http://127.0.0.1:4181/**, cùng origin với FastAPI. Không cần chạy một frontend dev server riêng. Giữ hai checkout `arena-backend/` và `uet-bot-arena/` cạnh nhau; API phục vụ `uet-bot-arena/dist/live/index.html` và các ES module trong `dist/live/`. File `dist/index.html` cũ và domain hosted vẫn là demo.

Khách không cần tài khoản để xem giải công khai, lịch đấu, BXH, Arena chính thức và replay. Xem [GUEST-SPECTATOR.md](GUEST-SPECTATOR.md) cho quyền truy cập, cách chia sẻ link và kiểm thử tải hai luồng SSE/người xem.

## Khởi động

Trang **Tài liệu & thể lệ** (`/#/guide`) có hướng dẫn viết bot C++17 chia thành 9 mục mở rộng, kèm tải Markdown, `bot.cpp` và input JSON Lines đầy đủ. Hướng dẫn dùng giao thức worker hiện có (init/ready, observation/action qua stdin/stdout). Mẫu trong hướng dẫn và file tải xuống được kiểm tra cùng nhau bằng `tests/bot_guide_check.py` trên Docker, dùng DB tạm riêng.

Mở Ubuntu WSL, vào thư mục backend:

```bash
cd ~/platform/arena-backend
# Các bước setup/image chỉ cần nếu chưa làm ở lần trước:
make setup
docker build -t uet-arena-worker:clock-v2 -t uet-arena-worker:experimental-clock-v2 .
.venv/bin/python -m arena_api migrate
.venv/bin/python -m arena_api bootstrap-admin --username organizer
# CLI hỏi mật khẩu ẩn, không có mật khẩu mặc định.
make serve
```

Nếu đã có tài khoản admin, bỏ bước bootstrap. Migration tự chạy khi serve, vẫn có CLI để kiểm tra offline. Migrate/bootstrap cần controller dừng. Dữ liệu mặc định ở `.platform-state`; không nhập dữ liệu từ lab cũ `.state` hoặc demo localStorage. Mở đúng `127.0.0.1`, không đổi sang `localhost` khi origin đang cấu hình là `127.0.0.1`.

Kiểm tra worker tại http://127.0.0.1:4181/api/v1/health/ready; `ready: true` mới chạy được source. API reference tại `/docs`. Nếu 4181 đang bận, dùng:

```bash
ARENA_API_ORIGIN=http://127.0.0.1:4182 .venv/bin/python -m arena_api serve --port 4182
```

Nếu đặt UI nơi khác, export `ARENA_UI_ROOT` bằng đường dẫn **Linux tuyệt đối** tới thư mục `dist` chứa `live/`, `styles.css` và `assets/favicon.svg`. Thiếu UI thì `/` chuyển tới `/docs`; API vẫn hoạt động. Đổi port hoặc data directory cần dùng cùng environment cho migrate/bootstrap/serve. `.env.example` chỉ là ví dụ, không tự được nạp.

## Thử trọn luồng bằng hai đội

1. Ở `#/register`, gửi đội 3–4 thành viên, nhận mã hồ sơ chờ duyệt. Không tạo password ở màn đăng ký.
2. Admin đăng nhập `#/login`, vào **Tài khoản đội thi** → **Hồ sơ chờ duyệt** → **Xem & duyệt**. Chọn username; để trống password cho server sinh ngẫu nhiên hoặc nhập mật khẩu tạm.
3. Hộp thoại cấp tài khoản chỉ hiển thị credential ở lần đó. Sao chép và phát qua kênh ban tổ chức; **Đã phát cho đội** chỉ ghi nhận, không gửi email. Đóng modal sẽ xóa credential khỏi UI memory. Nếu mất response/mật khẩu, cấp lại ở tab tài khoản.
4. Đăng xuất admin; đội đăng nhập bằng mật khẩu tạm và bắt buộc đổi. Sau đó vào trang chủ, hồ sơ đội, **Nộp bot**. Hồ sơ đã duyệt chỉ đọc; thay đổi thành viên liên hệ ban tổ chức.
5. Tải **starter OTTv2** từ server và nộp nguyên ZIP, version `v1.0.0`. Starter dùng C++17/protocol v2. Không dùng starter demo nối 5 từ UI hosted cũ.
6. Chạy **Test & benchmark** với thời gian phù hợp giải (mặc định 5 phút/đội; Bullet 1 phút). Job có progress 72 mẫu và báo cáo median/P95/max, startup/build, CPU và peak memory container, JSON timing/provenance. Benchmark public smoke không chứng nhận bot xử lý mọi trạng thái.
7. **Luyện tập** ghép hai version của chính đội; có thể chọn cùng version cho hai màu. Xác nhận ủy quyền tuyên bố hết giờ. Mở bàn cờ xem live hoặc tua lại. Trận luyện tập không tính hạng.
8. Thực hiện tương tự với đội thứ hai. Admin tạo giải ở **Giải đấu**, chọn ngân sách lượt/số đội, **Mở đăng ký**. Mỗi đội vào giải và chọn benchmark đạt 72 mẫu với đúng profile/image. Server kiểm tra đầy đủ source/artifact/SDK trước khi nhận entry.
9. Admin **Chốt bot & sinh lịch**: UI liệt kê tên đội/version trước xác nhận. Sau chốt không đổi entry. **Bắt đầu giải** tự dispatch các lượt đổi màu. **Tạm dừng ghép trận** chỉ dừng trận mới, không ngắt attempt đã vào queue.
10. Xem **Trực tiếp**, **Lịch thi đấu**, **Bảng xếp hạng** hoặc replay từ lịch. Chỉ fixture hoàn tất hợp lệ được tính điểm: thắng 3đ, hòa 1đ, thua 0đ; cùng tổng điểm đồng hạng. CSV có cột Điểm và bảo vệ giá trị mở đầu công thức. Trận `attention` có diagnostic/retry riêng cho admin và giữ attempt cũ.

Trang **Bảng xếp hạng** gom tối đa **10 đội đầu tiên** vào một biểu đồ thanh ngang chung, có trục tổng điểm và lưới so sánh. Độ dài thanh theo **điểm**, phân màu phần điểm từ hòa (vàng) và thắng (xanh); thua 0đ, số trận thắng/hòa/thua hiện dưới tên đội. Các đội từ vị trí 11 trở đi hiện trong bảng thường theo thứ tự điểm giảm dần của API. Khi đồng hạng ở ranh giới, biểu đồ vẫn tối đa 10 đội, thứ hạng chung được giữ nguyên trong bảng bên dưới. Tổng điểm và thứ hạng lấy trực tiếp từ API; không tự phân hạng lại bằng số thắng. Có bảng số liệu mở rộng cho top 10, nhãn đầy đủ cho trình đọc màn hình, trạng thái chưa thi đấu và cuộn ngang biểu đồ trên màn hình hẹp. CSV vẫn chứa toàn bộ đội. Không thêm polling hoặc reload định kỳ.

Tài khoản/reset/khóa, yêu cầu recovery và audit nằm trong UI admin. Reset thu hồi session và giữ nguyên trạng thái khóa. Recovery luôn trả lời chung để tránh dò tài khoản. UI không tự phát mật khẩu ra email.

## Live, replay và timing

- Bàn 9×9 gồm **81 DOM cell tồn tại suốt view**. SSE chỉ patch quân, highlight, counts, đồng hồ, kết quả và append diễn biến; không reload trang hoặc dựng lại bàn cờ. B = Búa, Ba = Bao, K = Kéo; a1/i9 là căn cứ.
- Nút về đầu, lùi/tiến, thanh tua, phát/dừng, mới nhất và xoay bàn dùng snapshot engine trong event log. Tua về nước cũ tắt theo dõi mới nhất; chọn **Mới nhất** để theo live lại. Kết quả đầu trang là kết quả trận từ server.
- Private practice dùng `/jobs/{id}/events`. Trận giải dùng feed whitelist `/competitions/{id}/fixtures/{id}/events`, không trả report/build log của đối thủ. Official SSE cursor `attempt_id:seq` tự reset khi retry; JSON backfill phân trang theo `next_after`, kể cả các event bị lọc.
- EventSource nối lại từ Last-Event-ID. Feed đóng khi terminal hoặc auth_expired. Mất mạng giữ bàn/replay hiện tại và thông báo đang nối lại. Rời view đóng feed/timer. Các bảng live/lịch/hạng không tự làm mới; dùng nút Cập nhật dữ liệu / Làm mới để lấy dữ liệu mới.
- Màn live của giải có **Tự chuyển trận** mặc định bật. Luồng SSE `/competitions/{id}/watch` chỉ gửi trạng thái lịch đấu khi đổi, không tải lại trang. Khi trận hiện tại hoàn tất/cần xử lý và trận sau được đưa vào worker, viewer thay tên đội, event feed và nội dung quân tại chỗ; shell và 81 cell được giữ nguyên. URL cập nhật bằng history.replaceState. Nếu các trận kết thúc rất nhanh giữa hai sự kiện, viewer theo đến trận mới nhất đã hoàn tất.
- Tắt Tự chuyển trận hoặc tua/phát replay trong bàn sẽ giữ trận hiện tại; bật lại để theo giải. Nút **Xem lại trận này** mở route Replay cố định; luyện tập và route Replay không mở feed tiến độ giải. Giải tạm dừng, chờ bắt đầu hoặc hoàn tất hiển thị đúng trạng thái. Rời trang đóng cả hai feed và hủy listener của bàn cũ.
- Countdown lấy 3 probe `/time`, chọn RTT thấp nhất, nội suy bằng `performance.now()`, đồng bộ lại mỗi 30s và hiển thị sai số RTT + worker mapping. Clock trình duyệt **chỉ hiển thị**. `t0`, deadline cứng `t1 < D` và khoảng chờ 3s đều do trusted worker/server xử; xem [TIMING.md](TIMING.md).

## Session và dữ liệu

Các trang Tổng quan, bảng xếp hạng, lịch, danh sách trực tiếp, giải đấu và vận hành
không có timer polling hoặc reload định kỳ. Dữ liệu được lấy khi mở trang, thực hiện
thao tác hoặc nhấn **Cập nhật dữ liệu / Làm mới**. Cập nhật thủ công đối chiếu DOM
trong `#main`, giữ shell/sidebar, node có key, focus và vị trí cuộn; bỏ phản hồi đến
muộn sau khi chuyển route. Lỗi mạng giữ dữ liệu hiện có và cho phép thử lại.
Bàn cờ và tiến độ job vẫn cập nhật bằng SSE, không phụ thuộc nút cập nhật bảng.
Quân và chú giải dùng SVG nét Búa/Lá/Kéo. Đồng hồ và replay giữ timer riêng để hiển thị.

Regression fixture riêng, không dùng session/database thật: từ `uet-bot-arena` chạy
`python3 tests/refresh-preview.py`, mở URL in ra và đợi khoảng 30 giây. `#qa-result`
kiểm tra Tổng quan và Bảng xếp hạng không phát sinh request/DOM mutation sau 12 giây
đứng yên; nút cập nhật lấy dữ liệu mới mà giữ main/shell, dòng job có key và focus.
Dừng fixture bằng Ctrl+C.

Cookie HttpOnly/SameSite=Strict, CSRF trong memory, `/auth/me` khôi phục sau reload. Không lưu credential, cookie, CSRF hay mật khẩu trong localStorage/sessionStorage. Server vẫn kiểm role/ownership cho mọi request; ẩn tab không thay thế phân quyền. Form giữ dữ liệu khi API lỗi, có request ID hỗ trợ. Upload/job/giải giữ idempotency key cho retry cùng payload; đổi input sinh key mới.

API chỉ phục vụ các asset UI đã định, không mount data directory/source/SDK làm static root. CSP chỉ cho script/style/connect cùng origin; dùng system font. UI không import dataset/auth giả của demo. C++17 là runtime duy nhất hiện bật; Codon để giai đoạn sau.

## Kiểm tra

```bash
# Ubuntu WSL, tại arena-backend:
make test
make integration
make schema
# Tại uet-bot-arena, Node không cần npm install:
npm run check
```

Trên Windows PowerShell có thể dùng `npm.cmd run check` nếu policy chặn `npm.ps1`. Kết quả kiểm thử và browser QA được ghi tại [VALIDATION.md](VALIDATION.md). Bản này chạy local trên WSL theo lựa chọn hiện tại; chưa publish domain hosted hoặc đưa backend ra Internet. Template nginx đã proxy `/` tới UI nối API cùng origin cho giai đoạn triển khai sau.

Regression chuyển trận: `node tests/follow.test.mjs` tại thư mục UI; fixture browser `python3 tests/follow-preview.py` kiểm tra opt-out/resume, đổi màu/tên, giữ shell/cell, tua tắt tự chuyển và Replay cố định. Fixture chỉ dùng dữ liệu giả, không dùng DB/tài khoản local.
