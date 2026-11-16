# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-TenNhom-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Bùi Quang Hiếu | 2A202603639 | 26ai.hieubq@vinuni.edu.vn | https://github.com/hieuwwww | Workflow + integration |
|   2 |        |      |       |        | Single |
|   3 |        |      |       |        | Orchestrator + handoff |
|   4 |        |      |       |        | Ledger + evaluation |

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
| | Single | | |
| | Orchestrator + handoff | | |
| | Ledger + evaluation | | |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-
