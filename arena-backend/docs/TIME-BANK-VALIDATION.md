# Đồng hồ cả trận — kiểm chứng local 2026-10-12

Mỗi bot có quỹ riêng: Bullet 60.000ms hoặc đấu thường 300.000ms, mặc định 5 phút. Không increment. Chỉ trừ từ lúc worker bắt đầu serialize observation đến lúc nhận đủ LF của action; build, ready, engine, đối thủ và SSE không trừ quỹ. Worker dùng nanosecond monotonic, giữ số dư xuyên các lượt. Hết quỹ: không nhận action muộn, server đại diện đối thủ tuyên bố sau tối thiểu 3 giây. Benchmark 72 mẫu chia chung một quỹ.

Request mới dùng `match_ms`. Response/action JSON Lines v2/v3 giữ nguyên; observation thêm `payload.clocks` và timing ngân sách còn lại. `profile.turn_ms` giữ cho parser cũ, không cấp lại mỗi lượt khi có match_ms. Bot cũ tìm kiếm theo toàn bộ thời gian mỗi lượt cần đổi cách phân bổ; bot mẫu và ZIP minimax đã cập nhật.

Các profile, image và kết quả giải cũ giữ nguyên. API vẫn nhận turn_ms cho cấu hình cũ; không cho gửi đồng thời hai chế độ. Tạo giải mới để chuyển từ clock cũ sang quỹ cả trận. Benchmark để dự giải phải khớp profile/image đã ghim, nên benchmark cũ không được tự chuyển thành đạt chuẩn mới.

## Kết quả

- 98 kiểm thử backend pass, gồm schema 1/5 phút, clock xuyên lượt, đối thủ đứng yên, timeout, replay snapshot và benchmark dùng chung quỹ.
- 22 kiểm thử UI pass; kiểm tra cú pháp JavaScript pass.
- `tests/time_bank_integration.py`: C++ thật / Docker, 8 checks pass. Quỹ riêng 450ms chỉ dùng trong kiểm thử nội bộ để ép timeout nhanh, API không cho chọn giá trị này. Mỗi action ngủ 160ms; Đỏ hết quỹ ở lượt thứ ba, board dừng tại 4 ply, Xanh còn khoảng 129ms. Kiểm tra trừ chính xác từ receipt nanosecond và chờ claim bằng monotonic. Trận và benchmark 72 mẫu ở cả quỹ 60s/300s pass.
- `tests/experimental_integration.py`: API thật, hai đội nộp SDK v3, benchmark, chốt giải hai lượt, guest SSE/replay, tính điểm và sai action/capability đều pass. Bản ghi có clocks ở từng state.
- `tests/integration.py`: regression cấu hình cũ, sandbox/off-turn freeze, hard deadline 15s + claim 3s pass.
- `tests/web_integration.py`: đăng nhập, nộp source, benchmark C++, trận luyện tập và SSE resume pass.
- `tests/bot_guide_check.py`: mẫu trong hướng dẫn build và đạt 72 mẫu với quỹ mới; input tải xuống khớp engine và action schema.
- `tests/experimental_minimax.py`: ZIP build, hai preset × 72 mẫu, các thế chiến thuật và duel pass; tìm kiếm đạt depth 3.

## UI và local

Kiểm tra trình duyệt trên bản sao dữ liệu test, không sửa tài khoản/giải local. Form luyện tập có đúng Bullet 1 phút / Đấu thường 5 phút, mặc định 5 phút. Replay đầu trận hiển thị 5:00.0 mỗi bên; cuối trận hiển thị 4:59.8 / 4:59.9. Tua replay giữ nguyên 81 ô và view ID; số dư đứng yên ở snapshot đã chọn. Console không có error/warning. Ảnh: `test-results/time-bank-ui.png`.

API local đã restart êm sau khi xác nhận không có job hoặc giải đang chạy. `/platform` trả match_options_ms=[60000,300000]; `/health/ready` trả ready=true với image clock-v2 `sha256:950ac21d0a0fb35ef8ec038748a87766f8b5a0660c841d92090ae4b5fc719183`.

Giới hạn kiểm chứng: timeout mới được ép bằng quỹ ngắn nội bộ, không chờ tiêu hết trọn 5 phút; các bài quỹ 60s/300s dùng bot nhanh và hoàn tất trước khi hết giờ. Kiểm thử này tập trung cơ chế clock và tương thích, không phải bài tải/soak mới.
