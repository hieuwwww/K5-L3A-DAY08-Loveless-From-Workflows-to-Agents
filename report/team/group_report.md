# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Loveless
- Lớp: L3A
- Repo: `K5-L3A-DAY08-Loveless-From-Workflows-to-Agents`
- Thành viên: xem `TEAM.md` (Bùi Quang Hiếu, Trần Tuấn Đăng, Nguyễn Đức Trọng, Phạm Quỳnh Anh)
- Commit chốt (trùng `report/team/decision_adr.md`): `3fa141f` (mốc tích hợp code và benchmark; sửa báo cáo sau mốc này không đổi code hoặc JSON benchmark)

## 2. Mục tiêu và cách tiếp cận

### 2.1. Mục tiêu bài toán
Hệ thống xử lý khiếu nại hoàn cước vận chuyển (claims) cho sàn LogiX-VN. Mục tiêu của nhóm là thử nghiệm cùng một quy trình nghiệp vụ trên 4 kiến trúc với mức độ tự chủ khác nhau:
- **Shape A (Workflow):** Pipeline xác định thuần Python (deterministic), không gọi LLM suy luận.
- **Shape B (Single ReAct):** 1 tác nhân ReAct (tối đa 8 bước), dùng MockLLM với các tool đọc an toàn.
- **Shape C (Orchestrator):** Mô hình Trưởng nhóm điều phối Investigator chuyên trách qua cơ chế Handoff đã lọc sạch (allowlist).
- **Shape D (Swarm):** Multi-agent phân tán, tương tác ngang hàng (mốc đối chứng).

Từ đó, nhóm đánh giá định lượng về tỷ lệ thành công (pass rate), độ ổn định qua các seed (variance), và chi phí tính toán (cost per success) để chọn ra kiến trúc tối ưu theo nguyên tắc: **"Ít tự chủ nhất mà vẫn làm được việc"**.

### 2.2. Phân công vai trò trong nhóm 4 người
- **Bùi Quang Hiếu (Workflow + Integration):** Phụ trách Shape A, tích hợp luồng host và hỗ trợ merge.
- **Trần Tuấn Đăng (Single ReAct):** Phụ trách Shape B, vòng lặp ReAct an toàn và bộ test single shape.
- **Nguyễn Đức Trọng (Orchestrator + Handoff):** Phụ trách Shape C, cơ chế handoff allowlist, điều phối investigator.
- **Phạm Quỳnh Anh (Ledger + Evaluation):** Phụ trách sổ cái, tính idempotent, crash recovery, benchmark suite, ADR và research card.

---

## 3. Giải pháp kỹ thuật

### 3.1. Ranh giới an toàn giữa Code và Model (Guardrails)
Hệ thống áp dụng nghiêm ngặt nguyên tắc ranh giới bảo mật:
1. **Tiền do Code quyết định:** Model chỉ có vai trò đọc/soạn thảo văn bản phản hồi. Số tiền hoàn được tính toán xác định qua `compute_compensation()`.
2. **Dữ liệu ticket là Untrusted:** Coi nội dung khiếu nại của khách hàng là dữ liệu người dùng thuần túy, tuyệt đối không tin số ngày trễ khai trong ticket hay các yêu cầu vi phạm chính sách ("ignore policy"). Toàn bộ căn cứ xử lý phải lấy từ bảng `orders` và `policies` hiện hành.
3. **Cổng kiểm soát Host Gatekeeper:** Mọi giao dịch bồi hoàn phải đi qua `durability.issue_refund()` với **quote đầy đủ**. Host chịu trách nhiệm kiểm tra quyền sở hữu (`principal_id`), tính hợp lệ của approval (`approval_id`), trần bồi hoàn cấp đơn hàng (trừ đi phần đã hoàn trước đó), và ghi nhận audit `policy_denied` khi có vi phạm.
4. **Handoff và Trace an toàn:** Dữ liệu chuyển giao giữa các agent được lọc nghiêm ngặt qua allowlist. Trace chỉ ghi nhận cấu trúc kết quả đã qua sanitize, tuyệt đối không để lọt `parent_secret`, số điện thoại, địa chỉ, hay ghi chú nội bộ.

---

## 4. Kết quả thực nghiệm

### 4.1. Bảng so sánh 4 kiến trúc (Trích xuất từ `src/reports/benchmark.json`)

Chạy trên 14 claims public × 5 seeds (11, 22, 33, 44, 55):

| Kiến trúc | Pass Rate | Pass^5 | Min Seed | Max Seed | Seed Variance | Cost / Success (VND) | Mean Steps | P95 Steps | Mean Model Calls |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **A. Workflow** | **1.000** | **1.000** | **1.000** | **1.000** | **0.000000** | **27.86** | **1.64** | **2** | **0.0** |
| **B. Single ReAct** | 1.000 | 1.000 | 1.000 | 1.000 | 0.000000 | 29.98 | 4.57 | 5 | 3.71 |
| **C. Orchestrator** | 1.000 | 1.000 | 1.000 | 1.000 | 0.000000 | 38.08 | 4.71 | 5 | 1.0 |
| **D. Swarm** | 0.986 | 0.929 | 0.929 | 1.000 | 0.000816 | 49.57 | 2.80 | 3 | - |

*Ghi chú: Chi phí VND tính theo mô phỏng tài nguyên token và tool call. Latency để `unavailable` do dùng MockLLM.*

### 4.2. Kết quả Crash Safety
- Chạy qua 20 kịch bản ngắt điện/sập nguồn tại các điểm nhạy cảm (`before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`):
- Kết quả trong `src/reports/crash.json`: **`crash_safety = 20/20`** (100% đạt chuẩn an toàn, không xảy ra chi trả đúp).

### 4.3. Quyết định kiến trúc (ADR)
Nhóm thống nhất chọn **Shape A — Workflow** làm giải pháp sản xuất:
- Đạt độ chính xác tuyệt đối 100% (pass rate = 1.0, pass^5 = 1.0).
- Phương sai bằng 0 (hoàn toàn ổn định, không bị ảnh hưởng bởi seed ngẫu nhiên).
- Chi phí rẻ nhất (27.86 VND/success), tiết kiệm 43.8% chi phí so với Swarm (49.57 VND).
- Không có rủi ro bị prompt injection hay ảo giác từ nội dung ticket.

---

## 5. Khó khăn và cách xử lý

1. **Khấu trừ trần hoàn tiền theo đơn hàng (Case T14):** Nếu shape tự trừ số tiền đã hoàn rồi mới gửi host, host sẽ trừ lần hai. Các shape truyền **quote đầy đủ** từ `compute_compensation()`; host tự tra cứu lịch sử và khấu trừ. Kịch bản T14 được kiểm qua ma trận student.
2. **Xử lý crash kiểm thử (Case T12):** Nếu nuốt `InjectedCrash`, bài test không kiểm được điểm crash. Shape flush trace rồi raise lại; T12 xác nhận resume vẫn chỉ một refund.
3. **Bảo mật dữ liệu nhạy cảm trong trace:** Ghi thẳng claim/order có nguy cơ lộ `parent_secret`, SĐT hoặc địa chỉ. Các shape chỉ ghi event đã chọn lọc, không serialize toàn bộ row; kiểm tra public quét trace và kết quả.

---

## 6. Tổng hợp đóng góp

- **Bùi Quang Hiếu:** `6b1faff` — Hoàn thiện `src/lab8/shapes/workflow.py`, test `src/tests/test_workflow_integration.py`.
- **Trần Tuấn Đăng:** `98fa598` — Hoàn thiện `src/lab8/shapes/single.py`, test `src/tests/test_single_shape.py`.
- **Nguyễn Đức Trọng:** `998a79b` — Hoàn thiện `src/lab8/shapes/orchestrator.py`, test handoff.
- **Phạm Quỳnh Anh:** `c68eab8` — Hoàn thiện test `src/tests/test_ledger_evaluation.py`, sinh `reports/crash.json`, `reports/benchmark.json`, hoàn thành ADR, Research Card và tổng hợp báo cáo.

---

## 7. Hạn chế và hướng cải thiện

- MockLLM hoạt động ngoại tuyến và dựa trên seed cố định, chưa phản ánh đầy đủ độ trễ mạng thực tế và tính bất định của các mô hình LLM thương mại lớn trong môi trường production.
- Hướng cải thiện: Khi mở rộng sang các trường hợp khiếu nại bằng hình ảnh (hàng vỡ móp thực tế) hoặc hội thoại tự nhiên với khách hàng, có thể tích hợp mô hình Multimodal kết hợp kiến trúc Router phân luồng linh hoạt giữa Workflow và Single Agent.

---

## 8. Tài liệu tham khảo

1. Google Research 2026, *Towards a Science of Scaling Agent Systems*.
2. MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?*.
3. Tài liệu đặc tả kỹ thuật và luật nghiệp vụ LogiX-VN (`docs/business_rules.md`, `docs/contracts.md`).
