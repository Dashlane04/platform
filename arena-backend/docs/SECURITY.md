# Worker boundary và triển khai

## Cấu hình beta

Docker run cố định: network=none, read-only rootfs, no-new-privileges, cap-drop=ALL; chỉ SETUID/SETGID/KILL cho trusted supervisor để demote/stop/kill bot. Trusted supervisor UID 0, bot/compiler UID/GID 10001, bot không giữ các capability đó. Không mount Docker socket, host home, credentials, SQLite, engine/controller state hoặc artifact đối thủ vào container. Bot run chỉ thấy artifact của nó; build thấy source và SDK ro, thư mục output riêng rw. `/tmp` có size limit.

Build: 1024 MiB cgroup, 1 CPU, 64 PID, 512 MiB tmpfs, wall 30s, log 256 KiB, binary tối đa 16 MiB. Flags do server cố định; không build script, không pip/network. Compiler vẫn xử dữ liệu không tin cậy, vì vậy **build cũng phải cô lập**.

Run: 256 MiB cgroup/no extra swap, 1 CPU, 64 PID toàn container, 32 MiB tmpfs, bot RLIMIT_AS/CPU/NOFILE/FSIZE/CORE. Child libseccomp filter từ chối fork/clone/thread/session/network/ptrace/cross-process signal/io_uring/bpf. Trusted supervisor không dùng filter child vì cần stop/resume bot. Root filesystem/common tools có thể đọc trong container; không chứa secret hoặc bot khác. Bot có thể exec tool khác trong cùng giới hạn nhưng không thêm process/thread, không thoát isolation và vẫn phải trả protocol.

Stderr có thread drain và giới hạn tổng; stdout theo frame; output oversize dừng job/match tương ứng. Worker không fallback host khi Docker unavailable. Source ZIP được kiểm trước ghi, không extractall, không nhận symlink hoặc traversal. Binary hash được xác minh trước reuse.

## Chưa phải production hardening hoàn tất

Container dùng chung kernel với host, không phải VM riêng. Chạy worker trên VM/host riêng không có credentials quan trọng; cân nhắc gVisor/Firecracker hoặc sandbox đã audit cho giải công khai. Không chạy API untrusted upload trên máy cá nhân chứa dữ liệu quan trọng. Trước public cần review seccomp/UID/mounts, cập nhật/pin dependency/image từ registry tin cậy, quét kernel/runtime, audit API và test độc lập.

Control API beta bind loopback. --dev chỉ cho laptop cá nhân; real deployment cần auth/session/role đội và admin, token rotation/revocation, HTTPS same-origin proxy, request/concurrency/rate/body limits tại gateway, durable object storage, quotas/retention/backup, event visibility cho spectators, metrics/logs, host calibration, worker lease/watchdog ngoài controller. Không mở Docker daemon TCP ra mạng.

Worker lab v1 tại port 4180 có sessions RAM 1h; hết hạn/restart mất session. Web API v0.3 tại port 4181 dùng session lưu DB dạng hash, Argon2 password, CSRF và role guards; xem [BACKEND.md](BACKEND.md). SQLite store có controller file lock ở launcher; không scale nhiều process/máy bằng cách dùng chung file. Queue chưa distributed. Launcher lab cũ chỉ join worker 12s khi dừng; web API mới chờ thao tác có giới hạn kết thúc trước khi đóng DB/nhả lease (service timeout 400s). Nếu bị kill cưỡng bức vẫn cần dọn container mồ côi theo tên arena-* sau khi chắc chắn controller cũ đã dừng. Không dùng wildcard xóa Docker resources khác.

Giải v0.3 chỉ scheduler nội bộ được tạo official job với owner rõ ràng cho từng source. API luyện tập vẫn kiểm cả hai bot thuộc đội hiện tại; không nhận cờ `official` hoặc owner từ client. Shared live/replay dùng whitelist board/timing/result, không chuyển nguyên raw worker event/report cho đối thủ. Diagnostic hai bot chỉ admin đọc. Entry chốt đúng binary đã benchmark; thay source/SDK/profile/binary khiến official attempt lỗi không tính điểm. [COMPETITIONS.md](COMPETITIONS.md) mô tả giới hạn scheduler, retry và phục hồi sau restart.

Không có email, upload source lên third-party hay secret team trong report. Hosting Site riêng vẫn là static demo; source chạy native phải qua API/worker Linux ngoài Sites.

## Sự cố Docker Desktop đã xử lý trên máy thử

Ngày 10/10/2026, Desktop 4.71.0 gặp stale AF_UNIX reparse-point sockets `dockerInference`, `docker-secrets-engine/engine.sock` và `userAnalyticsOtlpHttp.sock`. Khi Docker đã dừng, đổi tên **riêng các socket 0-byte đã kiểm tra** sang `.stale-*` qua WSL rồi khởi động lại. Không factory reset, không xóa volume/image/config/secret store. Bản lưu socket giữ nguyên ngoài workspace; không phải secret database.

Đây là khôi phục trạng thái runtime local, không phải yêu cầu chạy Codex bằng Administrator. Không lặp lại rename socket khi Docker đang chạy. Tham khảo báo cáo cùng dạng lỗi tại [Docker desktop-feedback #448](https://github.com/docker/desktop-feedback/issues/448).
