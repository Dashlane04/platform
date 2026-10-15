# Runtime policy · C++ mặc định, Codon tùy chọn

- **C++17 mặc định** cho phiên bản mới, starter và hướng dẫn. C++ là đường chính để vận hành giải.
- **Python + Codon 0.20.3 thử nghiệm**: chỉ khi đội chủ động chọn và xác nhận phạm vi hỗ trợ. Cho phép cú pháp/API đã kiểm tra; build/benchmark là điều kiện trước khi chốt. Kết quả lab không bảo đảm mọi bot gần C++.
- Không tự đổi sang CPython khi Codon build lỗi; báo đúng log để đội sửa hoặc chuyển sang source C++.
- Quay lại C++ nghĩa là nộp source C++ trong **phiên bản mới**, build/test lại và chốt theo thể lệ. Không chuyển ngôn ngữ tự động hoặc đổi runtime giữa trận.

## Frontend hiện có

`dist/runtime-config.js` chứa `LANGUAGE_POLICY`: `defaultRuntime: 'cpp17'`, `codonEnabled: true`. Đặt `codonEnabled: false` rồi xuất bản để tắt Codon cho lần nộp mới; UI và handler chỉ nhận các runtime đang mở. Đây chưa phải quyền hạn hoặc policy enforce ở server.

Mỗi submission demo mới lưu runtime, runtimePolicy và codonConsent. Metadata cũ thiếu runtime được hiển thị `Python 3 · bản cũ` vì UI trước đây chỉ hỗ trợ Python. Không đổi nhãn/binary của lịch sử thành C++.

Đổi runtime trong form giữ tên phiên bản và ghi chú, bỏ file đang chọn để đội chọn lại đúng source. Xác nhận Codon không được tự tick. Lưu thành công đưa form về C++ mặc định, không đổi bản đang active hoặc đã khóa.

Các ZIP starter C++/Codon là skeleton protocol đã kiểm tra local; engine, deployment worker và SDK thi chính thức vẫn cần hoàn thiện. Gói Python CPython cũ giữ tại `assets/bot-starter.zip` để không phá liên kết cũ, không dùng làm mặc định.

## Contract cần enforce khi có backend

Server lưu runtime/compiler image digest/build flags/dependency lock/source hash trong artifact bất biến, không tin runtime metadata của client. Backend kiểm tra đội đủ điều kiện dùng Codon và policy giải hiện tại; checkbox frontend không thay thế việc này. Binary dùng trong match phải đúng artifact đã chốt.

Tắt Codon cho submissions mới không sửa job/artifact hoặc trận đã chốt. Nếu ban tổ chức đổi policy cả giải, cần công bố trước deadline và cho đội thời gian nộp C++, không đổi ngầm giữa vòng.

C++ và Codon dùng cùng observation/action, engine, sandbox và resource profile đã công bố. Compiler version và flags C++ production chưa chốt; không suy ra từ compiler laptop. Benchmark báo runtime/source/suite/hardware/limits để so sánh cùng điều kiện. CPython trong biểu đồ lab chỉ là baseline nghiên cứu, không phải lựa chọn nộp mới.
