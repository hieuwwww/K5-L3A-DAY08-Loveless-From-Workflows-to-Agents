# Báo cáo cá nhân — 2A202603671_PhamQuynhAnh

## 1. Thông tin thành viên

- Họ tên: Phạm Quỳnh Anh
- MSSV: 2A202603671
- Email: 26ai.anhpq3@vinuni.edu.vn
- GitHub: https://github.com/phamquynhanh-17

## 2. Phần việc phụ trách

- Phụ trách vai trò **Ledger + Evaluation** trong nhóm 4 người.
- Xác thực cơ chế bền vững sổ cái (Ledger Durability, Idempotency và Crash Recovery).
- Thiết kế bộ kiểm thử an toàn sổ cái và chống thanh toán đúp (`src/tests/test_ledger_evaluation.py`).
- Thực thi kiểm thử Crash Suite 20 kịch bản (`reports/crash.json`) và Benchmark 4 kiến trúc x 5 seeds (`reports/benchmark.json`).
- Tổng hợp phân tích số liệu, hoàn thiện hồ sơ quyết định kiến trúc (`decision_adr.md`) và phiếu nghiên cứu (`research_card.md`).

## 3. Đóng góp cụ thể

- **Thiết kế kiểm thử an toàn sổ cái:** Thêm file `src/tests/test_ledger_evaluation.py` với 5 bài test kiểm tra các invariant sống còn:
  1. `test_ledger_idempotency_key_format`: Định dạng khóa idempotency `refund:{claim_id}` chuẩn.
  2. `test_ledger_exact_once_guarantee`: Khi gọi lại `issue_refund` nhiều lần với cùng số tiền, hệ thống chỉ ghi nhận 1 dòng `refund_ledger` và 1 audit event `refund_committed`.
  3. `test_ledger_conflict_on_amount_mismatch`: Khi gọi lại với số tiền bị can thiệp khác số tiền đã commit, hệ thống kích hoạt xung đột (`conflict=True`) và giữ nguyên số tiền ban đầu.
  4. `test_order_refund_cap_tracking`: Xác thực trần hoàn tiền cấp đơn hàng không bị vượt quá khi có nhiều claim.
  5. `test_crash_resilience_preserves_single_effect`: Mô phỏng sự cố sập nguồn sau khi ghi effect nhưng trước khi lưu checkpoint; kiểm tra quá trình khôi phục diễn ra an toàn theo cơ chế `idempotent_replay`.
- **Thực thi Crash Suite & Benchmark:**
  - Chạy `python -m lab8.crash_test --count 20 --out reports/crash.json` đạt tuyệt đối `20/20` điểm an toàn sập nguồn.
  - Chạy `python -m lab8.benchmark` trên 14 claims x 5 seeds (11, 22, 33, 44, 55) thu thập 280 trials đo lường pass rate, pass^5, phương sai và chi phí.
- **Biên soạn tài liệu và phân tích ADR/Research Card:**
  - Viết `report/team/decision_adr.md`: Đối chiếu số liệu Workflow (chi phí 27.86 VND, pass rate 1.0, variance 0.0) so với Swarm (chi phí 49.57 VND, fail ở seed 55) để chứng minh nguyên tắc "ít tự chủ nhất mà vẫn làm được việc".
  - Viết `report/team/research_card.md`: Phân tích câu hỏi R1 (Scaling) về tính tuần tự của bài toán logistics và kết quả thực nghiệm ablation.
- **Commit chính:** Commit `c68eab8` (`feat(eval): add ledger evaluation tests, benchmark and crash reports`).

## 4. Bằng chứng

- **Test passed:**
  - `python -m pytest -q tests/test_ledger_evaluation.py` (5 passed).
  - `python -m pytest -q` (76 passed).
  - `python -m pytest -q -m student -o addopts=` (248 passed).
- **Crash test report:** `src/reports/crash.json` ghi nhận `crash_safety: "20/20"`.
- **Benchmark report:** `src/reports/benchmark.json` ghi nhận Workflow đạt pass rate 1.000, pass^5 1.000, variance 0.000000, chi phí 27.86 VND/success.
- **Kiểm tra hợp lệ submission:** `python -m lab8.check_submission` đạt `0 FAIL`.

## 5. Công cụ AI đã dùng

- **Công cụ:** Antigravity AI Assistant.
- **Mục đích:**
  - Hỗ trợ xây dựng các kịch bản kiểm thử biên cho tính bất biến của sổ cái (idempotency invariants) và mô phỏng crash recovery.
  - Hỗ trợ phân tích thống kê từ file `benchmark.json` và định dạng văn bản báo cáo theo đúng schema kiểm tra tự động của ban giám khảo.
- **Tự kiểm tra:**
  - Toàn bộ test case đều được thực thi độc lập qua pytest trong môi trường Python local.
  - Số liệu trong ADR và Research Card được trích xuất trực tiếp từ các file kết quả do công cụ sinh ra, đối chiếu chéo với code kiểm tra `check_submission.py`.

## 6. Tự đánh giá

- Hoàn thành đầy đủ 100% trách nhiệm của vai trò Ledger + Evaluation.
- Tất cả các điều kiện must-pass (không bypass approval, không lộ secret, crash safety 20/20) được kiểm tra kỹ lưỡng và đạt chuẩn tuyệt đối.
