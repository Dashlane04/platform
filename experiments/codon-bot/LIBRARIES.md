# Kiểm chứng thư viện thuật toán với Codon

Đã compile/run các snippet bằng Codon **0.20.3** (`-release -numerics=py`) và đối chiếu stdout với CPython **3.12.3** trong cùng WSL. Không dùng `from python import`, decorator `@python` hay CPython bridge.

**Mỗi kết quả chỉ áp dụng cho đoạn code được kiểm tra**, không chứng nhận toàn bộ thư viện. Kết quả build fail là giới hạn của bản compiler được cố định, không phải khẳng định mọi bản Codon đều thiếu API đó.

| API / cách dùng | Kết quả | Phạm vi |
| --- | --- | --- |
| heapq · heappush/heappop | Native, cùng output | Dijkstra với heap các tuple (cost, node) |
| collections.deque | Native, cùng output | BFS với append/popleft |
| bisect | Native, cùng output | bisect_left/right và insort |
| itertools | Native, cùng output | combinations, permutations, product(repeat=2) |
| Counter / defaultdict | Native, cùng output | Đếm số nguyên; gom nhóm bằng defaultdict(list) |
| functools.lru_cache | Build fail | Không có API này trong bản thử; dùng cache dict đã kiểm tra |
| math.isqrt | Build fail | Không có isqrt trong bản thử |
| math.comb | Build fail | Không có comb trong bản thử |
| math.gcd / sqrt | Native, cùng output | UCLN và căn bậc hai |
| Memoization bằng dict | Native, cùng output | Fibonacci đệ quy có cache |
| NumPy native | Native, cùng output | Array int, nhân từng phần tử và np.sum |

## Ý nghĩa cho giải

- Đề xuất SDK/allowlist nhỏ, cố định compiler image và phát bộ kiểm tra cùng source cho đội. Cho đội chạy preflight trước hạn nộp để thấy lỗi ở đúng dòng.
- Các thuật toán tự viết bằng list/dict/set cùng kiểu, heap, queue và itertools có thể dùng native qua những API đã kiểm chứng.
- Thư viện PyPI ngoài danh sách này chưa được thử. Codon có cầu nối gọi Python, nhưng phần chạy qua cầu nối vẫn phụ thuộc CPython; không coi toàn bot là native độc lập. NetworkX/SciPy không được chứng nhận native bởi thí nghiệm này.
- NumPy được kiểm tra ở mức array và reduction nhỏ; chưa chứng nhận toàn API hoặc hiệu năng NumPy cho từng bot.
- Đã thấy khác biệt int64 và list trộn kiểu trong thử nghiệm trước: xem REPORT.md. Các bitboard hơn 64 bit cần biểu diễn phù hợp.

## Tái lập

```bash
python3 library_experiment.py
python3 write_library_report.py
```

Code từng snippet, SHA-256, thời gian build, output CPython/native và log lỗi ở `results/library-results.json`.

Tài liệu: [stdlib](https://docs.exaloop.io/libraries/stdlib/), [NumPy](https://docs.exaloop.io/libraries/numpy/), [Python bridge](https://docs.exaloop.io/integrations/python/python-from-codon/).
