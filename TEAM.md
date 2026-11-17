# Team

## Thông tin nhóm

- Tên nhóm:Loveless
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-Loveless-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:https://github.com/hieuwwww/K5-L3A-DAY08-Loveless-From-Workflows-to-Agents

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Bùi Quang Hiếu | 2A202603639 | 26ai.hieubq@vinuni.edu.vn | https://github.com/hieuwwww | Workflow + integration |
|   2 | Trần Tuấn Đăng | 2A202603618 | 26ai.dangtt@vinuni.edu.vn | https://github.com/TunDangg | Single |
|   3 | Nguyễn Đức Trọng | 2A202603644 | 25ai.trongnd@vinuni.edu.vn | https://github.com/fishiousrend05 | Orchestrator + handoff |
|   4 | Phạm Quỳnh Anh | 2A202603671 | 26ai.anhpq3@vinuni.edu.vn | https://github.com/phamquynhanh-17 | Ledger + evaluation |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Phân công triển khai (nhóm 4 người)

| Vai | Phần việc | Bằng chứng cần bàn giao |
| --- | --- | --- |
| Workflow + integration | Hoàn thiện `src/lab8/shapes/workflow.py`; nối luồng với host, tích hợp kết quả ba shape và xử lý lỗi khi ghép bài. | Test workflow T01–T14, smoke và kiểm tra tích hợp sau khi ghép. |
| Single | Hoàn thiện `src/lab8/shapes/single.py`: vòng ReAct tối đa 8 bước, tool chỉ đọc, quote từ order và ghi refund qua host. | Test single T01–T14, gồm retry/crash. |
| Orchestrator + handoff | Hoàn thiện `src/lab8/shapes/orchestrator.py`: handoff qua allowlist, investigator chỉ đọc, trace đã lọc dữ liệu. | Test orchestrator T01–T14 và test handoff/secret. |
| Ledger + evaluation | Kiểm tra idempotency, approval, trần hoàn tiền và crash qua test mới trong `src/tests/`; chạy benchmark/crash suite; tổng hợp ADR, research card và báo cáo theo số liệu thật. | Test ledger/evaluation đã thêm hoặc sửa, `src/reports/benchmark.json`, `src/reports/crash.json` và báo cáo. |

Không sửa `src/lab8/durability.py`, `src/lab8/policy.py` hay các file thước đo. Mỗi người tự điền thông tin và commit/test thực tế ở bảng bên dưới; mỗi người tạo báo cáo riêng từ `report/team/MSSV_HoTen.md`.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Bùi Quang Hiếu | Workflow + integration | `6b1faff` | `src/tests/test_workflow_integration.py` |
| Trần Tuấn Đăng | Single | `98fa598` | `src/tests/test_single_shape.py` |
| Nguyễn Đức Trọng | Orchestrator + handoff | `998a79b` | `src/tests/test_handoff_secret.py` |
| Phạm Quỳnh Anh | Ledger + evaluation | `c68eab8` | `src/tests/test_ledger_evaluation.py` |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603618_TranTuanDang

### Phần việc phụ trách

- Triển khai Shape B (`src/lab8/shapes/single.py` — Single ReAct agent).

### Đóng góp cụ thể

- Triển khai `src/lab8/shapes/single.py`: vòng lặp ReAct tối đa 8 bước (`max_steps=8`) qua `lab8.runtime.run_react` với quyền `worker=True`, chỉ dùng tool đọc (`lookup_order`, `compute_compensation`).
- Đảm bảo ranh giới an toàn: bỏ qua số ngày trễ ghi trong ticket (chống prompt injection và gian lận ngày trễ), luôn dùng thông tin từ database order row (`delay_days`).
- Tính quote và gọi `durability.issue_refund` với quote đầy đủ để host tự quản lý trần đơn và khấu trừ lần hoàn trước.
- Durability & Crash Safety: truyền tham số `crash_at` vào host và không bắt ngoại lệ `InjectedCrash` (cho phép hệ thống kiểm thử crash và khôi phục an toàn).
- Thêm bộ test `src/tests/test_single_shape.py` xác minh 14 case bẫy public, khắc phục điểm yếu của Swarm ở seed 55, và kiểm tra giới hạn bước/budget.
- Commit kỹ thuật: `98fa598` (`feat(shape-b): implement single ReAct agent and unit tests`).

### Bằng chứng

- Test: `python -m pytest -q tests/test_single_shape.py` (14/14 passed) và `python -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"` (62/62 passed).

## 2A202603671_PhamQuynhAnh

### Phần việc phụ trách

- Ledger, Durability, Crash Safety & Benchmark Evaluation.

### Đóng góp cụ thể

- Thêm bộ test `src/tests/test_ledger_evaluation.py`: kiểm thử tính idempotent trên khóa `refund:{claim_id}`, chống thanh toán trùng lặp, phát hiện xung đột số tiền (amount conflict) và duy trì trần hoàn tiền đơn hàng.
- Kiểm thử cơ chế phục hồi sau sự cố sập nguồn (Crash safety), đảm bảo giao dịch duy trì đúng một effect trong ledger và một committed audit event.
- Thực thi Crash Test Suite 20 kịch bản đạt chuẩn an toàn 20/20 (`src/reports/crash.json`).
- Thực thi Benchmark toàn diện trên 4 kiến trúc và 5 seed (`src/reports/benchmark.json`), trích xuất số liệu phân tích và hoàn thiện ADR (`decision_adr.md`) và Research Card (`research_card.md`).
- Commit kỹ thuật: `c68eab8` (`feat(eval): add ledger evaluation tests, benchmark and crash reports`).

### Bằng chứng

- Test: `python -m pytest -q tests/test_ledger_evaluation.py` (5 passed).
- Crash safety: `python -m lab8.crash_test --count 20 --out reports/crash.json` (20/20).
- Benchmark: `src/reports/benchmark.json` (Workflow đạt pass rate 1.0, cost 27.86 VND).

## 2A202603639_BuiQuangHieu

### Phần việc phụ trách

- Workflow + integration: triển khai Shape A (`src/lab8/shapes/workflow.py`) và kiểm tra tích hợp với host và bộ test public.

### Đóng góp cụ thể

- Triển khai `src/lab8/shapes/workflow.py`: đọc claim/order có scope, tính quote từ dữ liệu order bằng `compute_compensation`, truyền quote đầy đủ qua cổng host `durability.issue_refund`, trả `RunResult` và trace trên mọi nhánh.
- Thêm `src/tests/test_workflow_integration.py` để kiểm tra claim không tồn tại, trần hoàn tiền theo đơn và trace không lộ dữ liệu nhạy cảm.
- Commit kỹ thuật: `6b1faff` (`feat: implement workflow claim handling and integration tests`).

### Bằng chứng

- Test: `python -m pytest -q tests/test_workflow_integration.py` và `python -m pytest -q -m student -o addopts= -k workflow` (62/62 passed).
- Benchmark: `src/reports/benchmark.json` (Workflow đạt pass rate 1.000, pass^5 1.000, variance 0.000000, cost/success 27.86 VND).

## 2A202603644_NguyenDucTrong

### Phần việc phụ trách

- Orchestrator + handoff: triển khai Shape C (`src/lab8/shapes/orchestrator.py`), handoff qua allowlist, investigator chỉ đọc và trace đã lọc dữ liệu.

### Đóng góp cụ thể

- Triển khai `src/lab8/shapes/orchestrator.py`: đọc claim view, dựng gói handoff an toàn bằng `make_handoff` chỉ cấp tool đọc (`lookup_order`), thực thi investigator qua `run_readonly_investigators`.
- Tính quote từ order view bằng `compute_compensation`, truyền quote đầy đủ và `crash_at` vào `durability.issue_refund`, flush trace trước khi re-raise `InjectedCrash`.
- Commit kỹ thuật: `998a79b` (`feat: orchestrator`).

### Bằng chứng

- Test: `python -m pytest -q -m student -o addopts= -k orchestrator` (62/62 passed) và `python -m pytest -q tests/test_handoff_secret.py`.
- Benchmark: `src/reports/benchmark.json` (Orchestrator đạt pass rate 1.000, pass^5 1.000, variance 0.000000, cost/success 38.08 VND).
