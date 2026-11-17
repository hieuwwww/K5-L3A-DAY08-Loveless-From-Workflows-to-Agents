# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Loveless
- Lớp: L3A
- Repo: `K5-L3A-DAY08-Loveless-From-Workflows-to-Agents`
- Thành viên: xem `TEAM.md` (Bùi Quang Hiếu, Trần Tuấn Đăng, Nguyễn Đức Trọng, Phạm Quỳnh Anh)
- Commit chốt (trùng `report/team/decision_adr.md`): `a7b6f92`

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

| Kiến trúc | Pass Rate | Pass^5 | Min Seed | Max Seed | Seed Variance | Cost / Success (VND) | Mean Steps | Mean Model Calls |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **A. Workflow** | **1.000** | **1.000** | **1.000** | **1.000** | **0.000000** | **27.86** | **1.64** | **0.0** |
| **B. Single ReAct** | 1.000 | 1.000 | 1.000 | 1.000 | 0.000000 | 29.98 | 4.57 | 3.71 |
| **C. Orchestrator** | 1.000 | 1.000 | 1.000 | 1.000 | 0.000000 | 38.08 | 4.71 | 1.0 |
| **D. Swarm** | 0.986 | 0.929 | 0.929 | 1.000 | 0.000816 | 49.57 | 2.80 | - |

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

1. **Khấu trừ trần hoàn tiền theo đơn hàng (Case T14):**
   - *Vấn đề:* Ban đầu shape tự trừ số tiền đơn đã được hoàn ở lần trước rồi mới gửi sang Host, dẫn tới việc Host lại trừ tiếp một lần nữa (bị trừ đúp).
   - *Cách xử lý:* Tuân thủ đúng contract: Shape luôn gửi **quote đầy đủ** tính từ `compute_compensation()`. Host sẽ tự động tra cứu lịch sử đơn hàng và thực hiện khấu trừ chính xác.
2. **Xử lý ngoại lệ Crash kiểm thử (Case T12):**
   - *Vấn đề:* Một số nhánh code dùng khối `try...except Exception` bao bọc toàn bộ hàm khiến ngoại lệ `InjectedCrash` bị nuốt mất, làm bài test crash bị fail.
   - *Cách xử lý:* Cho phép `InjectedCrash` được raise thẳng ra ngoài (chỉ bắt tạm để flush trace trước khi re-raise), giúp hệ thống kiểm thử bắt được điểm crash chuẩn xác.
3. **Bảo mật dữ liệu nhạy cảm trong Trace:**
   - *Vấn đề:* Nguy cơ lọt `parent_secret` và thông tin cá nhân (SĐT, địa chỉ) nếu serialize toàn bộ bản ghi database vào log trace.
   - *Cách xử lý:* Chỉ dùng các view đã qua allowlist (`get_claim_view`, `lookup_order`), loại bỏ hoàn toàn các trường nhạy cảm trước khi đưa vào trace event.

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
