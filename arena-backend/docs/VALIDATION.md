# Kết quả kiểm thử local

## UI + API cùng origin v0.4 — 10/10/2026, 22:08 (Asia/Bangkok)

**52/52 kiểm thử tự động**, **10 worker + 6 web API + 5 competition integration checks** pass; `make integration` kết thúc exit 0. JavaScript cả demo cũ và 5 module UI mới qua `npm.cmd run check`. OpenAPI v0.4.0 có **48 paths**, gồm platform/rules/admin dashboard mới. Kiểm thêm UI/asset HTTP 200, CSP self-only, rules text/plain, dashboard RBAC, không expose backend/source/data directory qua static path, summary benchmark không trả report riêng.

Browser QA chạy Uvicorn thật cổng 4182, DB và tài khoản tổng hợp tách riêng; worker Docker thật cùng image `sha256:e596445bf49b00b6788c45879bfdedd2a7a4e4227a8c203b91a56da73468af8f`. Không sửa tài khoản của data directory mặc định hoặc publish Site hosted.

- UI đăng ký 3 thành viên, thêm/bỏ người thứ 4 giữ các trường đã điền, gửi lên API nhận ID hồ sơ. Admin đăng nhập/xem chi tiết hồ sơ; đội đăng nhập mật khẩu tạm thấy màn đổi bắt buộc. Approval và password rotation của tài khoản QA thực hiện qua HTTP API; API/unit/integration kiểm cả credential/reset/session, không coi thao tác đổi password đã được browser tự thực hiện.
- UI nộp SDK ZIP C++17 thật, tạo benchmark, nhận progress/report đủ **72 mẫu**, tải JSON và đọc lại tệp: `passed: true`, 72 samples. UI hiển thị median **0,316 ms**, P95 **0,505 ms**, max **0,797 ms**, CPU container **123,471 ms**, peak memory **8,50 MiB**, startup **3,965 ms**, build **4.131,161 ms**. Đây là số đo starter trên máy local, gồm trusted supervisor trong CPU/RAM; không phải cam kết hiệu năng hoặc benchmark hardware chuẩn.
- UI chọn benchmark đủ điều kiện cho đội thứ nhất. Đội thứ hai được chuẩn bị qua API và benchmark Docker thật. Admin chốt snapshot hai đội, sinh lịch và bắt đầu giải bằng UI. Cả hai lượt hoàn tất sau **30 ply**, Xanh chiếm căn cứ; standings UI hiển thị hai đội **cùng hạng 1**, mỗi đội 2 trận/1 thắng/1 thua.
- Mở live trước khi bắt đầu giải: board mount ID trước/sau giống nhau, luôn **81 cell**, nhận **31 state snapshots** từ ban đầu đến final. Tua về nước 0 vẫn giữ mount ID. Replay phát tới nước 6, dừng, về mới nhất và xoay bàn hoạt động; reload trang khôi phục session và event log.
- UI hồ sơ đội hiển thị thành viên/contact đã duyệt; UI luyện tập tạo trận giữa cùng version, nhận đủ 31 snapshots/kết quả thật. Báo cáo benchmark có CPU/RAM/startup/build và tải JSON thành công. Console được kiểm không có JS/CSP error.
- Desktop viewport 1440×900 và mobile 390×844 (content width 375px sau scrollbar): không tràn ngang; mobile mở/đóng menu giữ nguyên URL trận. Đã sửa vùng backdrop để click đóng không trúng link bên dưới. Ảnh local (ignored): `test-results/ui-arena-desktop.png`, `test-results/ui-arena-mobile.png`.

Hướng dẫn dùng bản nối API: [UI.md](UI.md). Vẫn giữ phạm vi local WSL; Site hosted cũ là demo. Chưa phải kiểm tải production hoặc audit cô lập source đối địch trên host nhiều người dùng.

Cuối vòng đã xác minh/restart tiến trình v0.3 của đúng workspace (đối chiếu inode và lock), giữ data directory/tài khoản có sẵn, không có job đang chạy. `make serve` v0.4 trên cổng **4181** trả UI HTTP 200, health version `0.4.0`, `ready: true`; mở trang chủ trong browser thành công. Server QA tạm 4182 được dừng; backend local 4181 giữ chạy để tiếp tục sử dụng.

## Chạy nối tiếp bằng Makefile trên WSL

Đã tái kiểm tra nguyên lệnh `make integration` sau khi thêm `cd` theo đường dẫn tuyệt đối trước từng recipe. Docker build, 10 worker checks, 6 web API checks và 5 competition checks chạy nối tiếp thành công, **make exit 0**. Không còn lỗi khởi động Python `getpath` / `failed to make path absolute` tại bước chuyển sang `web_integration.py` trong vòng này. Worker image của vòng chạy: `sha256:e596445bf49b00b6788c45879bfdedd2a7a4e4227a8c203b91a56da73468af8f`. Các report `test-results/*-latest.json` và `latest.json` được cập nhật bởi bộ thử.

## Giải chính thức v0.3

Vòng HTTP/Docker giải cuối cùng hoàn tất **2026-10-10 14:20:08 UTC**. Đã qua **48 kiểm thử tự động**, **10 kiểm thử Docker worker**, **6 kiểm thử HTTP tài khoản/luyện tập** và **5 kiểm thử tích hợp giải**. `pip check` không phát hiện dependency conflict; adapter JavaScript qua `node --check`; OpenAPI v0.3.0 có 45 paths. Không còn container `arena-*` sau các bộ thử.

Luồng giải dùng Uvicorn HTTP và Docker thật, không mock execution: hai đội được duyệt/đổi mật khẩu, nộp C++ và benchmark đủ 72 mẫu/đội; admin mở/chốt giải; hai fixture tự chạy với đúng owner/binary đã chốt. Cả hai lượt kết thúc sau 30 ply, Xanh thắng do chiếm căn cứ; mỗi đội thắng một trận, cùng hạng nhất. Board SSE/replay khớp final state, hỗ trợ cursor qua reconnect; đội không đọc được raw official job/report hoặc diagnostic của đối thủ. Lặp GET standings không cộng điểm lại; fixture đã hoàn tất không được retry.

Các kiểm thử tự động kiểm thêm: lịch ba đội có lượt nghỉ; lịch 32 đội đủ 992 cặp có thứ tự, mỗi đội Đỏ 31/Xanh 31; queue đầy không mất lịch; lỗi SQL khi chốt rollback cả lịch/trạng thái/idempotency; source/binary/runtime khác bị từ chối; báo cáo sai provenance không tính điểm; restart dừng giải và đưa attempt dở vào `attention`; retry giữ lịch sử và completion cũ không ghi đè; migration schema 1→2 giữ tài khoản/hash mật khẩu và phiên cũ.

Worker regression vẫn kiểm deadline thật 15s, không nhận action muộn; khoảng chờ tuyên bố đo được **3.014 ms**. Vòng worker hoàn tất 14:16:00 UTC; vòng web API 14:17:47 UTC. Image giữ nguyên `sha256:83492f21c80902d0329829ef4ed2cf809238ae9be874377f56103a9c51c60665`. Reports (ignored): `test-results/competition-latest.json`, `web-api-latest.json`, `latest.json`.

Đây là kiểm thử local trên WSL2/Docker, chưa phải kiểm tải 32 đội chạy bot thật hoặc audit production. Lịch 32 đội dùng record tổng hợp trong unit test; giải Docker thật dùng hai đội. Chưa nối UI hosted, chưa triển khai Internet hoặc distributed workers. Hướng dẫn: [COMPETITIONS.md](COMPETITIONS.md).

## Khung web API v0.2

Vòng HTTP thật hoàn tất **2026-10-10 13:52:35 UTC**. Đã qua **33 kiểm thử tự động** (32 kiểm thử chung và 1 kiểm thử CLI bootstrap/serve/shutdown), **10 kiểm thử Docker worker** và **6 kiểm thử HTTP/Uvicorn + Docker**.

Luồng đã chạy: đăng ký → admin duyệt/cấp mật khẩu tạm → bắt buộc đổi mật khẩu → nộp ZIP → build C++ → benchmark 72 mẫu → trận OTTv2 → replay/SSE reconnect → admin cấp lại mật khẩu và thu hồi phiên đội. Trận kết thúc sau 30 ply, Xanh thắng do chiếm căn cứ. Kiểm thử riêng xác nhận CSRF/RBAC/ownership, rollback approval trùng username, session persistence/expiry, body limits và không trả password hash.

Benchmark qua web API: median **0,359 ms**, P95 **0,932 ms**, max **1,233 ms**. Đây là starter trên máy thử, không phải bảo đảm hiệu năng cho mọi bot. Report: `test-results/web-api-latest.json`; worker regression: `test-results/latest.json` (đều ignored).

Image của vòng này: `sha256:83492f21c80902d0329829ef4ed2cf809238ae9be874377f56103a9c51c60665`. `pip check` không phát hiện dependency conflict. Schema đã xuất vào [openapi.json](openapi.json). Chi tiết chạy/vận hành: [BACKEND.md](BACKEND.md).

## Vòng worker lab trước đó

Vòng Docker hoàn tất lúc **2026-10-10 13:30:51 UTC**, trên Ubuntu WSL2 + Docker Desktop Linux Engine. Đây là kết quả của starter C++ và bộ trạng thái công khai, chưa phải chứng nhận cho bot của đội hoặc đo tải production.

## Kết quả

- **17/17** kiểm thử engine, protocol, upload, persistence và HTTP API.
- **10/10** kiểm thử tích hợp bằng container thật: build, benchmark, trận OTTv2, replay/SSE reconnect, quyền theo đội, lỗi compiler, action cũ, output limit, dừng bot ngoài lượt và giới hạn sandbox.
- Kiểm thử deadline thực **15 giây**: không nhận action muộn; khoảng chờ tuyên bố hết giờ quan sát được **3.006 ms**, không ghi timeout thành mẫu latency thành công.
- Trận starter đấu chính nó: **30 ply**, Xanh thắng do chiếm căn cứ (`base_capture`). Hai bot dùng hai container riêng.
- Browser: upload ZIP, benchmark, tạo trận, tua về lượt 0 và trở lại live; form đang nhập còn nguyên sau khi trận hoàn tất. Đã kiểm tra bố cục hẹp 458 px và desktop 1280 px, không tràn ngang.

| Benchmark 24 trạng thái × 3 lần | Kết quả |
|---|---:|
| Mẫu hợp lệ | 72 |
| Median wall time/lượt | 0,308 ms |
| P95 wall time/lượt | 0,652 ms |
| Max wall time/lượt | 0,833 ms |

Số đo dao động theo máy, scheduling và trạng thái bot. Đây chưa phải thử nghiệm so sánh C++ với Codon; backend hiện chỉ nhận C++17.

Image đã kiểm thử:

```text
sha256:c253d9c048d69ccc9deea48b2f5d8539dc391611185d769ffc9039adfd5f1e8a
```

Report đầy đủ, gồm hash engine/SDK/artifact, mẫu timing và metrics cgroup: `test-results/latest.json` (ignored). Chạy lại bằng các lệnh trong [README](../README.md); script tích hợp không bỏ qua lỗi Docker.

## Giới hạn

Engine được viết từ văn bản luật OTTv2 đã nhận; chưa đối chiếu với file JS gốc được nhắc trong luật. Console/API đang chạy local, chưa nối auth hoặc worker của web hosted. Các kiểm thử trên không thay thế review bảo mật độc lập trước khi mở upload công khai.
