# UET Bot Arena — UI

Cài toàn bộ UI + API trên máy mới: [INSTALL.md](../INSTALL.md). Source web chính nằm trong `dist/live/` và được FastAPI phục vụ; `npm run dev` chỉ mở prototype static cũ.

## Bản local nối API thật (v0.6)

v0.6 hiển thị hòa/lỗi startup theo [MATCH-POLICY.md](../arena-backend/docs/MATCH-POLICY.md), cột Hòa/CSV, policy v1/v2 của từng giải và queue ưu tiên trên dashboard. Máy local dùng **http://127.0.0.1:41810/**; cách chọn cổng và origin nằm trong README backend.

Bảng xếp hạng dùng **thắng 3đ, hòa 1đ, thua 0đ**, bằng tổng điểm đồng hạng. Top 10 cùng xuất hiện trên một biểu đồ thanh ngang có trục điểm chung; từ vị trí 11 là bảng thường theo điểm giảm dần. Biểu đồ vẫn tối đa 10 đội khi đồng hạng ở ranh giới, giữ nguyên hạng của API. Có số liệu thắng/hòa/thua, bảng top 10 mở rộng, CSV toàn bộ đội gồm cột Điểm và cuộn ngang biểu đồ trên mobile. Kết quả cũ được tính lại sau migration backend 004, không chạy lại trận. Các trang danh sách vẫn chỉ cập nhật khi mở trang hoặc nhấn Cập nhật dữ liệu.

Chạy `make serve` trong checkout `../arena-backend` trên Ubuntu WSL và mở **http://127.0.0.1:4181/**. FastAPI phục vụ `dist/live/index.html` cùng API, không cần frontend server riêng. Đăng ký/admin cấp tài khoản/đổi password, dashboard, nộp C++, benchmark, luyện tập, giải hai lượt, live/replay và bảng xếp hạng đều lấy dữ liệu thật từ backend. Hướng dẫn đầy đủ: [UI.md](../arena-backend/docs/UI.md). Kiểm tra JavaScript: `npm run check` (PowerShell: `npm.cmd run check`).

Admin local: `#/home` (metrics), `#/admin` (đội/tài khoản), `#/operations` (job/source), `#/competitions` (điều phối giải), `#/audit` (nhật ký). Xem [ADMIN.md](../arena-backend/docs/ADMIN.md).

Domain hosted và `npm run dev` bên dưới vẫn mở prototype cũ, không dùng để chạy bot hoặc đăng nhập backend thật.

## Prototype static cũ (giữ làm tham chiếu thiết kế)

Giao diện tiếng Việt cho nền tảng thi đấu bot sinh viên năm nhất. Không cần cài thư viện ngoài.

## Chạy trên máy

```sh
cd uet-bot-arena
npm run dev
```

Mở http://127.0.0.1:4173. Kiểm tra cú pháp: `npm run check`.

## Các màn hình

- `#/home`: Tổng quan, trận đang diễn ra, lịch tiếp theo, thông báo.
- `#/login`: Đăng nhập bằng mã tài khoản do admin cấp, hiện/ẩn mật khẩu, điền tài khoản mẫu.
- `#/register`: Đăng ký đội 3–4 người; kiểm tra mã sinh viên trùng và gửi hồ sơ chờ duyệt.
- `#/change-password`: Đổi mật khẩu tạm ở lần đăng nhập đầu tiên.
- `#/recover`: Yêu cầu admin cấp lại mật khẩu tạm.
- `#/admin/accounts`: Demo quản trị: duyệt đội, tạo tài khoản/mật khẩu tạm, sao chép/tải phiếu cấp, đánh dấu đã phát, cấp lại và khóa/mở khóa tài khoản.
- `#/live`: Trận trực tiếp, lọc bảng và trạng thái.
- `#/arena/M024`: Bàn 9×9, đồng hồ, lịch sử, sự kiện, góc nhìn và diễn biến demo.
- `#/replay/M020`: Phát, tạm dừng, tua theo từng nước hoặc thanh thời gian.
- `#/team`: Quản lý thông tin đội, thêm thành viên (tối đa 4), lịch sử và phiên bản bot.
- `#/submissions`: Chọn/kéo thả ZIP, kiểm tra extension/signature/size, lưu metadata, chi tiết lịch sử.
- C++17 mặc định cho lần nộp mới; Python + Codon 0.20.3 là lựa chọn thử nghiệm cần đội xác nhận. Runtime được lưu theo phiên bản, lịch sử Python cũ giữ nhãn cũ. Xem `RUNTIME-POLICY.md` để tắt lựa chọn Codon hoặc tích hợp policy phía server.
- `#/benchmark`: Chọn phiên bản cần test (chờ worker), xem báo cáo lab thật, đổi runtime/thuật toán, median/p95/max/RSS, build/startup riêng, so sánh ba runtime, hash bộ test/source, xuất JSON và bảng API Codon đã kiểm chứng.
- `#/standings`: Tìm tên đội, lọc bảng, sắp xếp, tải CSV.
- `#/schedule`: Lọc vòng/trạng thái/bảng, bốc thăm thử vòng 7.
- `#/guide`: Tài liệu, giới hạn của bản demo, bot mẫu tải xuống.

## Phạm vi

Đây là frontend demo. Đăng nhập so khớp thông tin thử trong bộ nhớ trình duyệt; chưa có xác thực hoặc phân quyền server. Mật khẩu được tạo và đổi chỉ tồn tại trong tab đang mở, không ghi vào localStorage. Sau khi tải lại trang, các tài khoản tự tạo cần được cấp lại mật khẩu từ demo quản trị. Tài khoản mẫu có nút điền thông tin để thử luồng. Hồ sơ demo, yêu cầu đăng ký/cấp lại, metadata tài khoản, thành viên, phiên bản và ghi chú lưu trong localStorage. Không gửi email hay thông tin tài khoản cho đội; nút sao chép/tải phiếu chỉ chuẩn bị thông tin demo để xem trước.

File ZIP chỉ được kiểm tra sơ bộ ở client; không gửi file ra mạng, không giải nén, không kiểm tra anti-cheat hay resource usage. Bàn đấu dùng diễn biến minh họa xác định, không phải engine OTTv2. Các clock và trạng thái là demo; server sẽ quyết định khi tích hợp.

UI dùng ES modules (app.js, pages.js, data.js, ui.js, match-view.js), HTML và CSS. Live và replay cập nhật các ô cờ, đồng hồ, lượt và lịch sử tại chỗ; giữ nguyên DOM của trang, bàn cờ, các ô và nút điều khiển. Lịch sử tự cuộn khi người xem đang ở cuối; giữ vị trí khi đang đọc nước cũ. Có thể triển khai dưới dạng static hoặc chuyển sang React khi tích hợp backend. Google Fonts là tùy chọn: khi không có mạng, dùng Segoe UI/sans-serif. Layout hỗ trợ desktop và mobile, semantic labels, điều hướng bàn phím, focus trap cho modal và reduced motion.

Starter mặc định ở `dist/assets/starter-cpp/`, dùng C++17 và header nlohmann/json v3.11.3 kèm giấy phép MIT. Starter opt-in ở `dist/assets/starter-codon/`, đã build thử bằng Codon 0.20.3 và chạy cùng source bằng CPython. Cả hai chỉ là skeleton protocol, chưa phải engine/SDK OTTv2 chính thức. Gói ZIP được tạo bằng `python scripts/package-starters.py`; kiểm tra giao tiếp nằm ở `../experiments/codon-bot/verify_starters.py`.

## Tích hợp tiếp theo

Benchmark hiện là báo cáo thí nghiệm nội bộ trên laptop WSL cho bot nối 5, không phải số đo bot của đội hoặc server OTTv2. `benchmark-data.js` được tạo từ `../experiments/codon-bot/results/results.json`; `library-data.js` từ probe compile/run thực. Không tạo job giả, không gán số đo lab vào submission được chọn. Nút chạy server bị vô hiệu hóa có giải thích. Prototype runner và hợp đồng API/worker nằm tại `../experiments/codon-bot/preflight.py` và `BENCHMARK-CONTRACT.md`; cần worker sandbox riêng trước khi chạy source của đội.

Thay data.js bằng API client; nối auth/session, membership, submissions, pairing/standings và SSE/WebSocket. Thay boardAt bằng state và event sequence từ server. Giữ credentials ngoài process bot, validate quyền và clock ở server. Chốt schema observation/action, luật di chuyển và pause trước khi sử dụng bot mẫu cho thi thật.
