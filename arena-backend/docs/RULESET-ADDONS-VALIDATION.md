# Kiểm chứng Ruleset add-on · 2026-10-13

Thay đổi nằm ở lớp catalog/pipeline và UI, tách khỏi `arena/experimental.py`, engine tiêu chuẩn và SDK simulator. Không thay byte/hash engine hoặc SDK đã ghim. Không migrate hay sửa definition của giải hiện có.

## Kết quả

- Backend: `python -m unittest discover -s tests -v` — **102 tests pass**. Bao gồm 4 bài pipeline mới, giữ kiểm tra qualification/hash, phân quyền, timing, replay và giải cũ.
- UI: **27 tests pass**, kiểm tra JavaScript `npm run check` pass. Các bài add-on kiểm tra điều hướng đội/BTC, escape nội dung, links theo đúng ruleset, mục tiêu/chi phí và giới hạn 32 bước.
- Sau chỉnh sửa cuối về schema/tài liệu và giữ release_notes khi client cũ không gửi trường này, 4 tests pipeline chạy lại pass. Sau bổ sung nhãn ID/loại quân trong preview và guard link giải khi tắt add-on, 10 tests add-on/experimental cùng syntax check chạy lại pass.
- ZIP starter riêng được nộp thật qua API submissions trong DB tạm, qua verifier source ZIP, nhận SDK v3. Gói tài liệu và starter được tách để contract.json/ruleset.json không gây lỗi upload source.

## QA trình duyệt trên bản sao DB

Server QA riêng `localhost:41820`, worker tắt, dùng bản sao `test-results/experimental-ui-proof-final`. Không thao tác tài khoản/giải ở cổng 41810.

1. BTC đăng nhập, nhập `examples/ruleset-addons/energy-dash.addon.json` và công bố ngay. Thư viện hiện Published và hash đúng definition.
2. Clone thành v2, giảm cost dash 2 → 1, lưu ghi chú rồi Kiểm tra & công bố. Trang chi tiết sinh đúng bảng trước/sau từ parent; bản v1 giữ nguyên.
3. Khi bật công tắc, link tạo giải chọn chính xác ID v2, switch được bật và select bắt buộc. Khi công tắc tắt, form báo không khả dụng; link từ editor chỉ xuất hiện khi đã bật.
4. Đội đăng nhập, thấy thư viện công bố, không thấy import/editor/toggle/release. Mở chi tiết, đọc changelog và thử nước đi. Sau bước 1, bàn chuyển Xanh tới lượt và legal_actions cập nhật tại chỗ.
5. Preview cho biết action ID, loại quân và năng lượng; có observation hiện tại để kiểm tra cooldown/Khiên. Không tạo job.
6. Desktop và mobile 375×812 không tràn ngang; preview mobile hiển thị đúng. Console không có warning/error.

Ảnh: `test-results/addon-team-details.png`, `test-results/addon-team-mobile.png`. Tab QA đóng, viewport khôi phục, server QA dừng và bản sao tạm được dọn sau kiểm tra.

## Phạm vi

Smoke hai màu kiểm tra khả năng mô phỏng cấu hình, không đo cân bằng hay sức mạnh bot. Không chạy lại tải/Docker parity cho thay đổi này vì không đổi primitive/engine/SDK/worker. Thêm primitive mới cần cập nhật các thành phần đó và chạy lại parity/integration. Các primitive hiện có vẫn được bảo vệ bằng suite experimental trong 102 tests.
