# Báo cáo cá nhân — 2A202603644_NguyenDucTrong

## 1. Thông tin thành viên

- Họ tên: Nguyễn Đức Trọng
- MSSV: 2A202603644
- Email: 25ai.trongnd@vinuni.edu.vn
- GitHub: https://github.com/fishiousrend05

## 2. Phần việc phụ trách

- Orchestrator + handoff: Triển khai Shape C (`src/lab8/shapes/orchestrator.py`), thiết kế gói handoff qua allowlist, investigator chỉ có quyền đọc và đảm bảo trace không rò rỉ dữ liệu nhạy cảm.

## 3. Đóng góp cụ thể

- Triển khai `src/lab8/shapes/orchestrator.py`:
  - Đọc thông tin claim qua `get_claim_view`, không đưa nội dung `ticket` thô, `parent_secret` hay raw headers vào trace.
  - Xử lý trường hợp thiếu `order_id` trả về `needs_info` (không suy diễn mã đơn từ câu chữ trong ticket).
  - Xây dựng gói dữ liệu chuyển giao `handoff` qua `lab8.handoff.make_handoff` chỉ chứa các trường trong allowlist và chỉ cấp quyền tool đọc (`INVESTIGATOR_TOOLS = ["lookup_order"]`).
  - Thực thi investigator chỉ-đọc qua `lab8.runtime.run_readonly_investigators`, đảm bảo investigator không có quyền ghi vào sổ cái.
  - Orchestrator tự tính tiền bồi hoàn bằng `compute_compensation` từ dữ liệu đơn hàng đã lọc, không bị ảnh hưởng bởi seed như Swarm (`seed % 5 == 0`).
  - Gọi `durability.issue_refund` với quote đầy đủ, truyền `crash_at` và chỉ flush trace trước khi re-raise `InjectedCrash` để phục vụ kiểm thử độ bền.
- Commit kỹ thuật: `998a79b` (`feat: orchestrator`).

## 4. Bằng chứng

- Ma trận student cho Shape C: `python -m pytest -q -m student -o addopts= -k orchestrator` (62/62 passed).
- Kiểm thử handoff & bảo mật: `python -m pytest -q tests/test_handoff_secret.py` (passed).
- Kết quả benchmark (`src/reports/benchmark.json`): Orchestrator đạt pass rate `1.000`, pass^5 `1.000`, seed variance `0.000000`, cost/success `38.08 VND`.

## 5. Công cụ AI đã dùng

- Sử dụng trợ lý AI để tra cứu contract của `make_handoff`, `run_readonly_investigators` và kiểm tra quy chuẩn Trace Contract v1.
- Tự kiểm chứng bằng cách chạy toàn bộ bộ test student của `orchestrator`, `test_handoff_secret.py` và `lab8.check_submission` (`0 FAIL`).

## 6. Tự đánh giá

- Hoàn thành đầy đủ yêu cầu của Shape C (`orchestrator.py`), đảm bảo ranh giới bảo mật handoff/trace và đạt 100% test passed trên cả bộ test đơn vị và benchmark.
