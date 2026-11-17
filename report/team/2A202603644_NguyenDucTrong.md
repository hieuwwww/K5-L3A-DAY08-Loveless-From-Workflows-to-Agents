# Báo cáo cá nhân — 2A202603644_NguyenDucTrong



## 1. Thông tin thành viên

- Họ tên: NguyenDucTrong
- MSSV: 2A202603644
- Email: 26ai.trongnd4@vinuni.edu.vn
- GitHub: (https://github.com/fishiousrend05)

## 2. Phần việc phụ trách

**Shape C — Orchestrator** (`src/lab8/shapes/orchestrator.py`), kèm phần handoff theo allowlist và trace không chứa secret.

Shape C gồm hai vai: investigator chỉ có tool đọc, và orchestrator là bên tính quote bằng code rồi gọi cổng ghi tiền của host (`durability.issue_refund`). Phần việc của mình là thay stub ở các dòng `TODO CP2` và `TODO CP3` bằng cài đặt đầy đủ, để shape đi qua bộ test student của shape C và giữ cổng host xanh.

## 3. Đóng góp cụ thể

**Commit chính:** `[ĐIỀN: hash commit của orchestrator.py]`

**Cài đặt `run_claim` của shape C**, giữ nguyên chữ ký `run_claim(claim_id, *, seed, store_path, crash_at=None) -> RunResult`. Luồng xử lý:

1. Đọc claim view bằng `get_claim_view`. Claim không tồn tại thì trả `failed`, `proposal_amount = 0`.
2. Thiếu `order_id` thì trả `needs_info`, không gọi tool nào nữa và không lấy mã đơn từ chữ trong ticket.
3. Dựng handoff bằng `make_handoff` với `facts={}` và chỉ cấp tool đọc (`lookup_order`), rồi chạy `run_readonly_investigators`.
4. Đơn không thấy hoặc không thuộc principal thì trả `escalated`, `proposal_amount = 0`, không gọi `issue_refund`.
5. Orchestrator tự quote bằng `compute_compensation` từ order view của investigator. Quote bằng 0 thì trả `denied`, `proposal_amount = 0`, không gọi `issue_refund`.
6. Quote lớn hơn 0 thì gọi `issue_refund` với **quote đầy đủ** và truyền `crash_at`. Kết quả host: `ok` thành `resolved`, `conflict` thành `failed`, còn lại (host từ chối, có audit `policy_denied`) thành `denied`. Cả ba trường hợp đều giữ `proposal_amount` bằng quote.

**Quyết định kỹ thuật:**

- **Investigator chỉ có `lookup_order`.** Quote do orchestrator tự tính nên không cần thêm `lookup_policy` hay `compute_compensation` cho investigator. Cách này giảm số tool call và cost/success. Có thể thêm `lookup_refund_history` vào `INVESTIGATOR_TOOLS` nếu cần, đổi lại thêm một tool call mỗi claim.
- **Truyền quote đầy đủ vào `issue_refund`.** Host tự trừ phần đã hoàn của đơn, nên shape không tự trừ (liên quan T13 và T14).
- **Không đọc ticket để quyết định.** Số ngày trễ lấy từ `delay_days` của order, nên shape C không bị lỗi tin số ngày trong ticket như swarm ở seed chia hết cho 5.
- **Quote bằng 0 thì không gọi host. Quote lớn hơn 0 thì luôn gọi host**, kể cả khi dự đoán host sẽ từ chối, vì chỉ host mới ghi được audit `policy_denied`.
- **`crash_at` được truyền thẳng vào `issue_refund`.** Khi gặp `InjectedCrash`, shape ghi trace rồi raise lại, không nuốt ngoại lệ.
- **Trace chỉ ghi dữ liệu đã qua allowlist:** handoff đã sanitize, claim view tối giản (`claim_id`, `has_order_id`, không có ticket), order view đã lọc field, quote, và kết quả của host. Không ghi claim row, order row hay `parent_secret`.
- Mọi nhánh return đều đi qua `lab8.shapes.common.finish()`, nên `refund_effects`, `committed_audit_count` và `trace_path` luôn lấy từ ledger và trace ghi trong chính lần chạy đó.

**Test:** không thêm test mới. Mình dùng bộ test student có sẵn của shape C (62 test) để kiểm tra.

## 4. Bằng chứng

**Commit:** `feat: orchestrator`

**Kết quả chạy test** (trong thư mục `src/`):

```
python -m pytest -q -m student -o addopts= -k orchestrator
→ 62 passed, 257 deselected, 3 warnings
```

Lần chạy `python -m pytest -q` trước đó cho kết quả `319 passed`. Ba warning `PytestUnknownMarkWarning` về `pytest.mark.student` xuất hiện vì lệnh dùng `-o addopts=` nên cấu hình đăng ký marker không được nạp. Đây chỉ là cảnh báo, không ảnh hưởng kết quả test.

**Chưa điền (làm ở CP4 và CP5):**

- Crash suite: `python -m lab8.crash_test --count 20 --out reports/crash.json` cho `crash_safety` = `[ĐIỀN]`
- Benchmark của shape C (pass rate, pass^5, variance giữa các seed, cost/success): `[ĐIỀN từ reports/benchmark.json]`
- Run id minh họa: `[ĐIỀN, ví dụ R-orchestrator-<claim_id>-s11]`

## 5. Công cụ AI đã dùng

**Claude (Anthropic)**, dùng vào các việc sau:

- Đọc và tóm tắt cấu trúc repo, luật nghiệp vụ (`business_rules.md`), contract (`contracts.md`) và các checkpoint.
- Đề xuất bản nháp `orchestrator.py` dựa trên các file mình cung cấp (`handoff.py`, `durability.py`, `compensation.py`, `tools.py`, `mock_llm.py`, `schemas.py`, `common.py`, `swarm_reference.py`, `trace.py`, `db.py`). Bản nháp bám theo cách gọi trong `swarm_reference.py`.

**Cách mình tự kiểm tra kết quả:**

- Chạy bộ test student của shape C trên máy mình và đạt 62/62 (xem mục 4).
- Đối chiếu code với bảng trạng thái trong `docs/contracts.md` và các luật trong `docs/business_rules.md`.
- `[ĐIỀN thêm nếu đã làm: chạy crash suite, đọc file trace trong thư mục traces/ để chắc không có ticket hay secret, chạy benchmark]`

Lưu ý: file `runtime.py` chưa được đưa cho Claude đọc, nên cách gọi `run_readonly_investigators` dựa trên cách `swarm_reference.py` đang dùng. Các test student đã xác nhận cách gọi này hoạt động.

## 6. Tự đánh giá

**Làm tốt:** shape C qua toàn bộ 62 test student, giữ cổng host xanh, tuân đúng các luật: quote đầy đủ, không đọc ticket để quyết định tiền, chỉ cấp tool đọc cho investigator, trace không chứa dữ liệu nhạy cảm.

**Còn hạn chế / cần làm tiếp:**

- Chưa có số liệu crash suite và benchmark để so sánh C với A, B, D trong ADR (điền ở CP4 và CP5).
- Shape C dùng một investigator và tool đọc rất tối thiểu, nên với bộ claim public nó khó cho kết quả tốt hơn workflow thuần code. Số liệu cost/success và variance sẽ cho thấy điều này, và mình sẽ đưa vào ADR cùng nhóm.
- `[ĐIỀN thêm nhận xét cá nhân: điều bạn học được, khó khăn gặp phải]`
