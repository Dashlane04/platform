# OTTv2 bot protocol v2

Encoding UTF-8, một JSON object + LF mỗi frame. Tối đa 65.536 byte gồm LF. Không duplicate key, NaN/Infinity, trailing JSON/frame dư hoặc field phản hồi ngoài schema. Bot nhận EOF thì thoát. Stderr là log có giới hạn. V1 starter trên demo cũ chưa có handshake; dùng ZIP v2 từ backend.

## Init → ready (2 giây)

```json
{"protocol_version":2,"type":"init","match_id":"opaque-job-id","payload":{"side":"red","engine":"ottv2-v1","profile":{"match_ms":60000,"grace_ms":3000}},"timing":{"clock_id":"worker-instance-id","started_mono_ns":"123000000000","deadline_mono_ns":"125000000000","budget_ms":2000}}
```

`profile` thực tế có thêm giới hạn RAM/build/output/job trong `/config`. Đây là handshake, chưa phải lượt suy nghĩ. Bot trả chính xác:

Web controller còn gửi `payload.match = {"kind":"benchmark|match","adjudication":{...}}` để bot biết chế độ và phiên bản xử lý kết quả. Observation có `payload.adjudication` tương ứng. Đây là trường bổ sung của input; bot không copy vào response. Report/live v2 có thể kết thúc với `result.winner:null` (hòa hoặc cả hai bot lỗi startup); xem [MATCH-POLICY.md](MATCH-POLICY.md).

```json
{"protocol_version":2,"type":"ready","match_id":"opaque-job-id"}
```

## Observation → action

```json
{"protocol_version":2,"type":"observation","match_id":"opaque-job-id","turn_id":"1","state_version":0,"payload":{"state":{"board":[],"to_move":"red","ply":0,"result":null},"legal_moves":[{"from":"a2","to":"a3"}],"counts":{"red":{"rock":3,"paper":3,"scissors":3},"blue":{"rock":3,"paper":3,"scissors":3}},"bases":{"red":"a1","blue":"i9"}},"timing":{"clock_id":"worker-instance-id","started_mono_ns":"200000000000","deadline_mono_ns":"215000000000","budget_ms":15000}}
```

`board: []` ở ví dụ viết tắt; frame thật luôn là array 9 hàng × 9 cột. Ô trống `null`, quân `{"side":"red|blue","kind":"rock|paper|scissors"}`. `legal_moves` được engine tính cho bên tới lượt, không phải gợi ý chiến thuật.

```json
{"protocol_version":2,"type":"action","match_id":"opaque-job-id","turn_id":"1","state_version":0,"payload":{"from":"a2","to":"a3"}}
```

Hoặc `"payload":{"resign":true}`. Không chấp nhận `resign:1`. Mỗi observation có đúng một response. `turn_id` là **string opaque**, không suy ra từ số ply; benchmark có nhiều observation độc lập, state_version có thể lặp nhưng turn_id luôn mới. `state_version` là integer không âm, copy đúng kiểu và giá trị.

Engine kiểm tra lại legal move; không tin legal claim của bot. Sai match/turn/version → `stale_action`; field sai/JSON lỗi/response dư → `protocol_error`. Phản hồi phải được worker đọc đủ LF trước deadline; gửi một phần hoặc không flush vẫn hết giờ.

## Vòng đời và tài nguyên

Một process sống xuyên trận/benchmark, không reset mỗi lượt. Sau ready hoặc action, worker SIGSTOP và chờ xác nhận process đã dừng; SIGCONT khi có observation tiếp. Không chạy suy nghĩ khi đối thủ đi. Fork/thread/new session/network/ptrace/kill syscall bị chặn; C++ v1 single-thread. Không có API HTTP dành cho bot và không có credential server trong bot environment.

`timing.*_mono_ns` là string để không mất chính xác trong JS; chỉ có ý nghĩa trong `clock_id` đó. Bot có thể dùng `std::chrono::steady_clock` để tự giới hạn nội bộ theo **budget tương đối** và chừa dư, không dùng UTC hoặc tự báo elapsed để xin thêm giờ.

Admin có thể pause/resume scheduler giải; chỉ ngừng ghép trận mới, không pause clock của bot đang đấu. Disconnect người xem không pause bot. Hỏng worker/controller không tiếp tục clock cũ; job aborted/failed. Replay là event log, không chạy lại bot.

Đồng hồ cả trận: Bullet 60.000ms hoặc đấu thường 300.000ms **mỗi bot**, không cộng giây. Observation thêm `payload.clocks` (red/blue, ms); `timing.time_control="match"`, `remaining_ms`, `initial_ms`, `increment_ms=0`. `budget_ms` là quỹ còn lại làm tròn xuống; dùng deadline tuyệt đối do worker cấp. Response/action giữ nguyên v2/v3. Benchmark chia chung quỹ cho 72 mẫu. Request `turn_ms` chỉ hỗ trợ cấu hình cũ; không gửi cùng `match_ms`. Xem [TIMING](TIMING.md).

`profile.turn_ms` vẫn được gửi để bot cũ đọc được init. Khi có `profile.match_ms`, worker chỉ áp dụng quỹ cả trận: `turn_ms` không phải ngân sách được cấp lại mỗi lượt. Bot cũ chỉ trả action nhanh vẫn dùng được; bot tìm kiếm theo toàn bộ turn_ms phải chuyển sang chia timing.budget_ms còn lại cho các nước tiếp theo.
