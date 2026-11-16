# Báo cáo cá nhân — Trần Tuấn Đăng

## 1. Thông tin thành viên

- Họ tên: Trần Tuấn Đăng
- MSSV: 2A202603618
- Email: 26ai.dangtt@vinuni.edu.vn
- GitHub: https://github.com/TunDangg

## 2. Phần việc phụ trách

Single agent: Triển khai shape B (`src/lab8/shapes/single.py`), vòng lặp ReAct có giới hạn bước, công cụ chỉ đọc và kiểm thử độ bền (durability / crash).

## 3. Đóng góp cụ thể

- Triển khai `src/lab8/shapes/single.py`:
  - Khởi tạo vòng lặp ReAct qua `lab8.runtime.run_react` với `max_steps=8` và chế độ `worker=True`.
  - Thiết kế hàm `decide(seen)` với các quyết định đọc dữ liệu (`lookup_order`, `compute_compensation`).
  - Đảm bảo tính toàn vẹn nghiệp vụ: bỏ qua câu chữ trong ticket người dùng (chống prompt injection và bẫy khai khống ngày trễ), chỉ sử dụng trường dữ liệu `delay_days` từ database order row.
  - Chuyển toàn bộ quyền ghi cho host: gọi `durability.issue_refund` với quote đầy đủ, host chịu trách nhiệm kiểm tra approval, idempotency và khấu trừ trần theo đơn.
  - Hỗ trợ kiểm thử độ bền: truyền `crash_at` vào host, bắt và flush trace khi gặp `InjectedCrash` trước khi re-raise để phục vụ kịch bản crash test.
- Xây dựng bộ kiểm thử `src/tests/test_single_shape.py`:
  - Kiểm tra Shape B vượt qua toàn bộ các bẫy public (T01–T10, T13, T14).
  - Kiểm tra tính kháng lỗi đối với điểm yếu của Swarm ở seed 55 trên T02 (vẫn giữ quyết định từ chối chính xác).
  - Kiểm tra agent luôn hoàn thành trong giới hạn $\le 8$ bước và ghi nhận chi phí budget hợp lệ.
- Commit kỹ thuật: `98fa598` (`feat(shape-b): implement single ReAct agent and unit tests`).

## 4. Bằng chứng

- Test cá nhân: `python -m pytest -q tests/test_single_shape.py` (14 passed).
- Ma trận student cho Shape B: `python -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"` (62/62 passed).
- Cổng host: `python -m pytest -q` (71 passed).

## 5. Công cụ AI đã dùng

- Google Antigravity: hỗ trợ tìm hiểu cấu trúc repo, phân tích luồng ReAct, gợi ý thiết kế closure state cho hàm `decide` và xây dựng unit tests.
- Tự kiểm chứng: Chạy toàn bộ ma trận test student, kiểm tra trace contract và tính an toàn idempotency/crash trên SQLite.

## 6. Tự đánh giá

Đã hoàn thành 100% mục tiêu của Shape B; vượt qua toàn bộ 62 test của bộ student matrix; code an toàn, không rò rỉ PII/secret và sẵn sàng tích hợp vào benchmark chung của nhóm.

