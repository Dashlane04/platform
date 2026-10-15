# Kiểm chứng experimental · 12/10/2026

Chạy trên Ubuntu WSL2 + Docker Linux, DB/tài khoản thử riêng. Dữ liệu thật vẫn
giữ 3 tài khoản; công tắc experimental mặc định tắt. Có backup SQLite trước
khi cập nhật controller. UI/API local phục vụ tại `http://127.0.0.1:41810`.

| Phép kiểm | Kết quả |
|---|---|
| Backend unit/API | 93/93 pass |
| UI syntax + Node tests | 18/18 pass |
| C++ ↔ Python simulator | 707 thế, khớp legal actions/ID và next state |
| Native C++ experimental | 2 đội × 72 mẫu, ready/hash/capability v3 hợp lệ |
| Tương thích classic | Starter mở rộng đạt 72 mẫu protocol v2 |
| Giải chính thức | 2 lượt đổi màu, thắng control_target ở ply 17, mỗi đội 3đ |
| Luật bất biến | Chặn benchmark khác luật; tắt/ẩn luật vẫn hoàn tất giải đã chốt |
| Bot lỗi | Worker thật từ chối thiếu capability và action ID lạ |
| Kỹ năng | Luyện tập thật dùng Dash/Khiên; replay có trace và tài nguyên |
| UI trình duyệt | Tạo/validate/publish/chọn luật; preview trừ năng lượng; Khiên hiện trên quân; 81 ô cập nhật; mobile không tràn ngang |

Các cơ chế engine được kiểm riêng: chặn đường, filter loại/ID, chi phí/cooldown,
hết hạn và phá Khiên, cap năng lượng, ưu tiên điều kiện thắng, lặp thế với thời
hạn tương đối, action sai không sửa state, schema/giới hạn frame, CSRF/RBAC,
revision/idempotency, published/clone/archive, hash SDK/engine và qualification.

## Tải đọc, preview và restart

| Nhóm | Yêu cầu | Đồng thời | HTTP 200 | P95 |
|---|---:|---:|---:|---:|
| Public views + experimental replay | 240 | 12 | 240 | 528.6 ms |
| Completed replay SSE + competition SSE | 120 | 24 | 120 | 596.6 ms |
| Preview ruleset có mẫu đi rộng | 96 | 8 | 96 | 443.9 ms |

Không lỗi HTTP/khác payload/rò dữ liệu worker. SIGKILL chỉ controller thử khi
idle, khởi động lại giữ nguyên ruleset/replay/BXH; reconnect cursor không lặp
event. Đây là loopback trên máy local với tài nguyên dùng chung, không có bot
đang thi đấu trong phép đo tải. Chưa chứng nhận sức chứa live hoặc cân bằng luật.

Docker Desktop gặp lại lỗi socket `dockerInference` sau khi môi trường local
dừng; đã khôi phục bằng đổi tên hai thư mục runtime theo cách được người dùng
cho phép, giữ image/volume. Backend được chạy nền bằng `scripts/start-local.ps1`.

## Chạy lại

```bash
make test
make experimental-test
make experimental-load-test
```

Chi tiết máy chạy nằm trong `test-results/experimental-parity-latest.json`,
`experimental-integration-latest.json` và `experimental-load-latest.json`.
Ảnh kiểm tra: `experimental-editor-ui.png`, `experimental-replay-ui.png`.
Thư mục test-results được gitignore; mỗi lượt load-test giữ bộ dữ liệu seed
riêng để có thể kiểm tra replay bằng tay.
