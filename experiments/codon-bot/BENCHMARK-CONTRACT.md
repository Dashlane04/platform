# Test & benchmark cho đội thi

Prototype hiện có: `preflight.py` chạy bot nội bộ đã build, trả JSON thật; UI đọc báo cáo lab cũ từ `benchmark-report.json`. **Chưa có API upload, queue hay worker production; UI không thực thi source vừa nộp.** Nền tảng chọn C++17 làm mặc định; Python + Codon 0.20.3 chỉ là lựa chọn opt-in. CPython trong lab là baseline, không phải runtime nộp mới.

## Quy trình cần tích hợp

1. Chọn submission bất biến và runtime (CPython/Codon/C++), tạo job gắn với SHA-256 của source. Ghi compiler version/flags, dependency lock và worker image digest.
2. Build với timeout, trả log lỗi thư viện/kiểu dữ liệu đúng dòng. Build time là metric riêng, không tính vào lượt đấu.
3. Kiểm tra handshake, schema observation/action và tính hợp lệ của nước đi bằng engine OTTv2 đúng phiên bản. Lỗi ở bước này kết thúc job, không hiển thị kết quả “đạt” vì chạy nhanh.
4. Khởi động một tiến trình bot cho toàn phiên như lúc thi thật. Đo startup riêng, warm-up có công bố. Nếu luật giải reset process mỗi lượt thì benchmark cũng phải reset.
5. Chạy cùng bộ trạng thái khai cuộc/trung cuộc/cuối cuộc, phân nhánh lớn, gần hết thời gian. Lặp nhiều lần, xáo thứ tự có seed cố định. Worker benchmark chỉ đo một job tại một thời điểm; không để build hoặc job khác tranh CPU.
6. Trả báo cáo gắn source/runtime/engine/suite/worker/limits; lưu mẫu theo case để đội biết trạng thái nào chậm. Báo cáo tài nguyên tách khỏi điểm đấu thử.

## Số đo tối thiểu

| Số đo | Định nghĩa |
| --- | --- |
| turn_wall_ms median / p95 / max | Runner dùng đồng hồ monotonic: bắt đầu gửi observation → nhận đủ action. Protocol production phải định nghĩa chính xác mốc này; không lấy timer do bot tự khai làm điểm chấm. |
| cpu_total_ms | CPU user+system của cả sandbox. Prototype local chỉ đo tổng CPU của tiến trình con (gồm startup, correctness, benchmark, shutdown); chưa có CPU mỗi lượt. |
| peak_memory_mib | Production dùng cgroup memory.peak của toàn sandbox. VmHWM của riêng bot chỉ là số liệu local, không tương đương cgroup memory. |
| startup_wall_ms / build_seconds | Đo riêng, có số lần lặp và điều kiện cache. |
| timeouts / crashes / protocol_errors / illegal_actions | Ghi case/phase, signal hoặc exit code, stderr giới hạn dung lượng. Không tính timeout thành một mẫu latency hoàn tất. |
| budget_usage_percent | p95/turn_budget × 100 và max/turn_budget × 100; chỉ có khi cùng cấu hình giới hạn thi thật. Khoảng dư là gợi ý trên bộ test công khai, không bảo đảm mọi trạng thái đều kịp. |
| source / image / engine / suite hashes | Tái lập và ngăn so sánh hai bản source hoặc điều kiện đo khác nhau. |

Không chuẩn hóa thời gian Python bằng một hệ số cố định từ benchmark laptop. Cho các runtime cùng tài nguyên hoặc công bố rõ thể lệ trước giải. Có thể chạy một baseline cố định trước/sau job để phát hiện worker bị nhiễu; nếu baseline lệch ngưỡng hiệu chuẩn thì gắn nhãn `noisy` và chạy lại, không tự “sửa” số đo.

## API dự kiến (chưa triển khai)

```http
POST /api/submissions/{id}/benchmarks
Idempotency-Key: <unique-client-key>
{"runtime":"cpp17","profile_id":"ottv2-practice-v1"}
```

Trả `202 {job_id, status: "queued", submission_sha256, profile_id}`. Server kiểm tra đội sở hữu submission, runtime nằm trong allowlist và giới hạn số job. Không nhận command tùy ý từ client.

`GET /api/benchmarks/{job_id}` trả trạng thái `queued → building → validating → benchmarking → completed|failed`, progress theo số case, report nullable và lỗi có phase. SSE `/events` có sequence number và reconnect; UI chỉ cập nhật panel/job liên quan. `GET /report` xuất JSON, `GET /samples.csv` xuất các mẫu; báo cáo thất bại vẫn tải được. `completed` chỉ nghĩa worker hoàn tất phép đo; eligibility thi chính thức phải do server quyết định riêng.

## Worker production

Worker Linux/container tách khỏi static hosting. Build và chạy code không tin cậy trong sandbox: không mạng, không credentials, filesystem giới hạn/read-only, user không đặc quyền, PID/output/CPU/memory/wall limits và thu hồi cả process tree. Giới hạn lúc build cũng cần kiểm soát. Chốt engine/SDK/allowlist và profile tài nguyên trước khi mở upload thực.

## Chạy prototype local

```bash
python3 preflight.py --runtime cpp --algorithm minimax --budget-ms 100
python3 preflight.py --runtime codon --algorithm minimax --budget-ms 100
python3 preflight.py --runtime cpython --algorithm minimax --budget-ms 100
```

100 ms là ngân sách thử nội bộ có thể đổi, không phải giới hạn OTTv2 đã thống nhất. Runner chỉ nhận tên ba runtime nội bộ, dùng bot tin cậy, không có sandbox/memory quota. Kết quả ở `results/preflight-<runtime>-<algorithm>.json`; khi timeout dừng job và ghi đúng số mẫu hoàn tất/bị bỏ qua. Không sử dụng để chạy source do đội upload.
