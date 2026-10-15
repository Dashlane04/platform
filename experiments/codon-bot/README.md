# Python → native bot experiment

Repo public giữ source và các báo cáo Markdown. `results/`, compiler và binary đo trên máy phát triển được ignore; chạy lại các lệnh bên dưới để tạo dữ liệu cho máy của bạn. Runtime Codon chưa được bật trong backend thi đấu chính.

Thử nghiệm nội bộ, tách khỏi bản UI và source hosting. Mục tiêu: xác minh một bot viết bằng cú pháp Python có thể được build trước thành executable native, chạy liên tục qua stdin/stdout và cho kết quả giống CPython; đo tương quan với C++ cùng thuật toán.

## Phạm vi

- Bàn nối 5 kích thước 9×9 minh họa, **không phải engine hoặc luật OTTv2**.
- Greedy: đánh giá tối đa 12 nước ứng viên.
- Minimax/negamax với alpha-beta: sâu 3, tối đa 8 nhánh mỗi nút.
- Flat Monte Carlo: 128 rollout, tối đa 24 nước mỗi rollout; **chưa phải MCTS có cây UCT**.
- Python và Codon chạy **chính xác cùng file `bot.py`**, không thêm decorator, C extension hoặc cầu nối CPython.
- C++ là bản chuyển cùng thuật toán, dữ liệu `vector<int64_t>`; chưa tối ưu riêng bằng bitboard/SIMD.
- Chỉ chạy source thử do chúng mình viết. Runner đo này chưa phải sandbox để nhận code không tin cậy.

## Chạy lại

Cần Linux x86_64/Ubuntu WSL, Python 3, g++, curl, tar, libgfortran5. Codon v0.20.3 được cố định phiên bản và kiểm tra SHA-256 theo bản phát hành chính thức.

```bash
bash bootstrap.sh
python3 experiment.py
python3 json_experiment.py
python3 write_report.py
```

Trên Windows, từ thư mục workspace:

```powershell
wsl --distribution Ubuntu --exec bash experiments/codon-bot/bootstrap.sh
wsl --distribution Ubuntu --exec python3 experiments/codon-bot/experiment.py
wsl --distribution Ubuntu --exec python3 experiments/codon-bot/json_experiment.py
wsl --distribution Ubuntu --exec python3 experiments/codon-bot/write_report.py
```

Compiler được giải nén vào `.tools/`, binary và log nằm trong `build/`; không thay PATH, cài gói hệ thống hay chỉnh cấu hình WSL. Lần chạy đầy đủ sẽ build lại và ghi lại kết quả đo.

## Thiết kế đo

1. Build `bot.py` bằng Codon với `-release -numerics=py`; C++ bằng g++ với `-O3 -march=native`.
2. Cố định runner và các tiến trình con trên cùng một CPU logic. Mỗi lần chỉ một bot tính toán.
3. Kiểm tra 29 vị trí × 3 thuật toán × 3 runtime; so sánh cả nước đi, điểm và số nút đã thăm. Có tình huống thắng ngay, bắt buộc chặn, bàn trống, bàn chỉ còn một ô và bàn hòa đã đầy.
4. Đo 24 vị trí ngẫu nhiên cố định × 5 lần lặp, đảo thứ tự runtime và vị trí để giảm thiên lệch thứ tự. Bot được giữ sống suốt phiên; warm-up bằng lượt kiểm tra đúng sai trước đó.
5. Ghi thời gian tính toán bên trong bot và thời gian round-trip qua pipe do runner đo. Đo riêng 200 PING và 20 lần khởi động mỗi runtime sau khi cache đã ấm.
6. Ghi peak RSS từ `/proc`, executable đang chạy, liên kết ELF và các probe về JSON, list trộn kiểu, số nguyên lớn.

Không coi thời gian khởi động đã làm ấm là cold start. Không suy ra thông lượng server hay độ công bằng giữa ngôn ngữ từ benchmark laptop WSL này. Giới hạn nhánh và rollout cố định nhằm so sánh cùng công việc, không đánh giá chất lượng chiến thuật.

## Kết quả và source

- `REPORT.md`: kết luận, số liệu và hướng tích hợp.
- `results/results.json`: môi trường, compiler, correctness, timing, bộ nhớ, liên kết và probe.
- `results/samples.json`: từng mẫu đo, đủ để phân tích lại.
- `results/fixtures.json`: các bàn cờ dùng trong thử nghiệm.
- `bot.py`, `bot.cpp`: hai bản source để đối chiếu.
- `experiment.py`: bộ đo, kiểm tra và điều phối tiến trình.
- `bot_json.py`, `json_experiment.py`: adapter JSON-lines chạy trên cả CPython và Codon, cùng kiểm tra đúng sai và latency thực.
- `write_report.py`: tạo lại báo cáo từ các số liệu đã đo.
- `library_experiment.py`, `LIBRARIES.md`: các ví dụ thư viện thuật toán được build/run và đối chiếu CPython, ghi rõ API thiếu trong Codon 0.20.3. Chạy `python3 library_experiment.py` rồi `python3 write_library_report.py`.
- `preflight.py`: prototype worker local với latency/p95/max, CPU tổng, peak RSS, timeout và hash; C++ là mặc định: `python3 preflight.py --algorithm minimax --budget-ms 100`. Dùng `--runtime codon` để thử Codon. Chỉ chạy bot nội bộ tin cậy, chưa phải sandbox.
- `BENCHMARK-CONTRACT.md`: pipeline và contract API dự kiến cho đội thi; các endpoint chưa triển khai.
- `export_benchmark.py`: tạo report JSON có provenance và module dữ liệu cho UI từ số đo thực nghiệm.

Tài liệu đối chiếu: [Codon build](https://docs.exaloop.io/start/usage/), [khác biệt với Python](https://docs.exaloop.io/language/overview/), [bản phát hành v0.20.3](https://github.com/exaloop/codon/releases/tag/v0.20.3).
