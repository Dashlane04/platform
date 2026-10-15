# Thử nghiệm Python → bot native bằng Codon

**Kết luận: khả thi để xây MVP có starter kit/SDK và phạm vi Python được quy định rõ.** Cùng một source Python đã build thành executable ELF native, chạy xuyên suốt phiên bằng stdin/stdout và cho kết quả khớp CPython. Hiệu năng trên các thuật toán thử gần bản C++ tham chiếu. Chưa thể kết luận cho mọi code Python hoặc engine OTTv2 thật.

## 1. Môi trường và cách đo

- Thời điểm: 2026-10-10T03:48:19Z.
- CPU: 13th Gen Intel(R) Core(TM) i5-13420H.
- OS: Linux-6.18.40.1-microsoft-standard-WSL2-x86_64-with-glibc2.39.
- CPython: 3.12.3; Codon: 0.20.3; g++ (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0.
- Cố định trên CPU logic [1]; một bot tính tại một thời điểm, không bật đa luồng trong thuật toán.
- Cùng file `bot.py` cho CPython và Codon. C++ chuyển cùng thuật toán, cùng kiểu số nguyên 64-bit, chưa tối ưu riêng bằng bitboard/SIMD.
- Codon: `-release -numerics=py`; C++: `-O3 -DNDEBUG -std=c++17 -march=native`. Không dùng fast-math hoặc tắt kiểm tra exception cho Codon.
- Đo sau warm-up: 24 vị trí cố định × 5 lần lặp = 120 mẫu mỗi thuật toán/runtime. Đảo thứ tự công việc và runtime. Bộ đo giữ bot sống, không tính compile/startup vào thời gian chọn nước.
- Bài toán là nối 5 trên bàn 9×9 minh họa, **không phải OTTv2**. Monte Carlo là rollout phẳng, **không phải MCTS có cây UCT**.
- Source và binary nằm trên filesystem Windows mount vào WSL. Các số đo là thăm dò trên laptop, chưa phải dự báo thông lượng server Linux sản xuất.

## 2. Đúng sai và chạy native

- 87 trường hợp = 29 vị trí × 3 thuật toán; 261 phản hồi từ 3 runtime đều khớp **nước đi, điểm đánh giá và số nút đã thăm**.
- Tình huống gồm bàn trống, thắng ngay, bắt buộc chặn, một ô trống và hòa đầy bàn; tất cả nước trả về hợp lệ. Protocol sai cấu trúc và board sai kích thước trả lỗi trong cả ba runtime.
- Các kết quả còn được đối chiếu lại trong toàn bộ 1,080 request đo thời gian.
- `/proc/<pid>/exe` của bot Codon là `build/bot-codon`, định dạng ELF x86-64. Chạy executable trực tiếp, không gọi `python bot.py` hoặc `codon run` khi thi đấu.
- `ldd` và map thư viện của tiến trình Codon không có `libpython`. Source không dùng cầu nối `from python import`.
- Binary vẫn dùng **runtime Codon (`libcodonrt.so`) và các shared library**. Đây không phải binary tĩnh hoàn toàn; cần đóng gói runtime vào image của worker.

## 3. Thời gian tính toán

Median tính toán bên trong bot, chưa gồm parse/serialize và giao tiếp. Cột tăng tốc là tỷ lệ giữa các median, không phải cam kết cho mọi vị trí.

| Thuật toán | CPython (ms) | Codon native (ms) | C++ (ms) | CPython / Codon | Codon / C++ |
|---|---:|---:|---:|---:|---:|
| Greedy | 0.4060 | 0.0220 | 0.0190 | 18.45× | 1.16× |
| Minimax sâu 3, tối đa 8 nhánh | 7.9050 | 0.2900 | 0.2275 | 27.26× | 1.27× |
| Monte Carlo 128 rollout | 5.6670 | 0.3035 | 0.2725 | 18.67× | 1.11× |

Codon nhanh hơn CPython khoảng 18–27× về phần tính toán trong bộ đo này. C++ vẫn nhanh hơn: Codon mất thêm khoảng 11–27% thời gian so với bản C++ cùng thuật toán. Điều này chưa chứng minh ngang mọi C++ được tối ưu chuyên biệt.

## 4. Giao tiếp, khởi động và bộ nhớ

Median round-trip do runner đo, gồm gửi trạng thái text, parse/serialize và nhận hành động:

| Thuật toán | CPython (ms) | Codon (ms) | C++ (ms) |
|---|---:|---:|---:|
| Greedy | 0.4492 | 0.0422 | 0.0418 |
| Minimax sâu 3, tối đa 8 nhánh | 7.9933 | 0.3172 | 0.2721 |
| Monte Carlo 128 rollout | 5.7466 | 0.3412 | 0.2989 |

| Chỉ số | CPython | Codon | C++ |
|---|---:|---:|---:|
| Khởi động đến READY, median 20 lần (ms) | 35.4022 | 82.6012 | 2.1906 |
| PING/PONG, median 200 lần (ms) | 0.0095 | 0.0043 | 0.0044 |
| Peak RSS sau workload (MiB) | 11.2070 | 8.9141 | 3.5195 |

Build bot Codon mất 20.33s; C++ mất 1.29s trong lần chạy này, cùng được pin CPU. Đây là chi phí trả khi nộp/build source, không nên lặp lại mỗi lượt.

**Khởi động Codon chậm hơn CPython trong môi trường đo này.** Cache đã được làm ấm; đây không phải cold start. Vì library và binary trên mount Windows, không suy ra startup của server Linux từ các giá trị này. Dù vậy, thiết kế giữ một tiến trình mỗi bot trong suốt trận giúp tránh trả lại startup mỗi lượt.

## 5. Tương thích Python

| Probe thực chạy | Kết quả | Ý nghĩa |
|---|---|---|
| `json.loads` + `json.dumps` | Build và output khớp CPython | JSON cơ bản dùng được trong bản Codon đã thử |
| `values=[1]; values.append("hello")` | CPython chạy; Codon báo lỗi kiểu khi compile | Không thể nhận mọi Python hợp lệ |
| `x=2**62; print(x*4)` | CPython: `18446744073709551616`; Codon: `0` | `-numerics=py` vẫn không biến `int` thành số nguyên không giới hạn |

Số nguyên lớn đặc biệt đáng chú ý nếu dùng **bitboard 81 ô**: một `int` 64-bit không đủ. Starter kit nên dùng list 81 ô hoặc biểu diễn nhiều word có quy tắc rõ. Không được mặc định code chạy đúng trên CPython sẽ giữ mọi ngữ nghĩa sau compile.

Module JSON native của Codon 0.20.3 trả kiểu `JSON` thay vì dict/list Python thông thường; direct iteration và một số hook khác với CPython. SDK cần chuyển trạng thái sang cấu trúc đồng nhất trước khi gọi hàm chiến thuật. Probe parse/dump thành công chưa chứng minh mọi cách sử dụng `json` đều tương thích.

## 6. Kiểm tra JSON thật với bot

`bot_json.py` nhận mỗi trạng thái bằng một dòng JSON, gọi đúng thuật toán trong `bot.py`, rồi trả hành động bằng JSON. Cùng adapter chạy được trên CPython và Codon; dùng indexing và `int(...)` để chuyển board sang list số nguyên.

- Native JSON wrapper build thành công trong 23.86s.
- 84 trường hợp × 2 runtime = 168 phản hồi khớp kết quả protocol text. Bàn đã đầy được bỏ qua ở kiểm tra wrapper tối giản vì match runner nên dừng trước khi hỏi nước tiếp theo.
- Hash core `bot.py` vẫn khớp bản benchmark chính. Chưa đổi thuật toán để thích nghi JSON.
- Bot JSON native không load `libpython`; không dùng Python interoperability.
- Đo thêm 24 vị trí × 3 lần lặp mỗi thuật toán/runtime với JSON thật:

| Thuật toán | CPython JSON round-trip median (ms) | Codon JSON round-trip median (ms) |
|---|---:|---:|
| Greedy | 0.4167 | 0.0347 |
| Minimax sâu 3, tối đa 8 nhánh | 7.2360 | 0.3071 |
| Monte Carlo 128 rollout | 5.2306 | 0.3417 |

Wrapper chỉ xử lý input đúng schema; chưa phải parser/SDK đã gia cố cho người dùng công khai.

## 7. Đề xuất cho UET Bot Arena

Có thể tiếp tục hướng **viết chiến thuật bằng Python trong phạm vi hỗ trợ → build Codon sau khi nộp → chạy native khi đấu**. Cần làm SDK mẫu và chốt profile ngôn ngữ trước khi công bố cho sinh viên.

Pipeline đề xuất:

```text
Nộp source → build job riêng → kiểm tra tương thích/protocol
          → lưu binary + phiên bản runtime cố định
          → worker trận đấu mở mỗi bot một lần
          → JSON observation / JSON action qua stdin/stdout
          → engine xử lý luật → sự kiện live cho UI
```

Trước giải thật còn phải thử engine OTTv2 thật, bot sinh viên đa dạng, ngân sách CPU/RAM, sandbox cho source không tin cậy, quá tải khi nhiều trận và hành vi crash/timeout. Native compilation không thay thế sandbox.

Có thể giữ C++ là lựa chọn trực tiếp của nền tảng; thí nghiệm này chưa giải quyết công bằng khi nhiều ngôn ngữ thi chung.

## 8. Tái lập và dữ liệu

- Xem `README.md` để chạy lại. Compiler được pin Codon v0.20.3 và xác minh checksum bản phát hành.
- `results/results.json`: môi trường, correctness, timing, liên kết, bộ nhớ, probes.
- `results/samples.json`: toàn bộ 1,080 mẫu benchmark chính.
- `results/json-results.json`, `results/json-samples.json`: JSON adapter và mẫu đo bổ sung.
- `results/fixtures.json`: dữ liệu bàn cờ; `build/*-build.log`: log compiler.
- `bot.py` và `bot.cpp`: source để đối chiếu; `experiment.py`, `json_experiment.py`: bộ đo.

Tài liệu chính thức: [Codon build](https://docs.exaloop.io/start/usage/), [khác biệt với Python](https://docs.exaloop.io/language/overview/), [bản phát hành v0.20.3](https://github.com/exaloop/codon/releases/tag/v0.20.3).
