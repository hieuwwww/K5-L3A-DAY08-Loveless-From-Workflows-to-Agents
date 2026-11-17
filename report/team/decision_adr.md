# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 3fa141f (mốc tích hợp code và report benchmark trên main; các sửa báo cáo sau mốc này không đổi code hoặc JSON benchmark)
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000-1.000 / 0.000000
- Cost per success (mô phỏng, VND): 27.86
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

## Vì sao shape này

Workflow đạt 70/70 trial trên 14 claim public × 5 seed (pass rate = 1.000, pass^5 = 1.000, variance = 0.000000) với chi phí mô phỏng thấp nhất trong bốn shape (27.86 VND/success; mean steps = 1.64). Swarm đạt 69/70 (49.57 VND/success, khoảng 1.78 lần Workflow), sai tại C-002 seed 55 do tin số ngày trễ ghi trong ticket thay vì order (variance = 0.000816, pass^5 = 0.929). Với bộ dữ liệu có cấu trúc và luật nghiệp vụ cố định của lab, Workflow là kiến trúc ít tự chủ nhất vẫn đạt kết quả đúng trên tập public này; kết quả không chứng minh hiệu năng hoặc tính an toàn tuyệt đối trên dữ liệu ngoài lab.

## Có đáng đổi lấy multi-agent không

Trên cùng corpus và seed, Single ReAct (29.98 VND/success) và Orchestrator (38.08 VND/success) cùng đạt pass rate 1.000 như Workflow nhưng có chi phí mô phỏng cao hơn; Swarm (49.57 VND/success) còn có một trial sai và variance > 0. Do đó tập public hiện tại chưa cho thấy lợi ích về độ chính xác xứng với chi phí của multi-agent. Benchmark ghi latency là `unavailable`, nên không kết luận kiến trúc nào nhanh hơn. Nhóm sẽ cân nhắc lại nếu dữ liệu phi cấu trúc hoặc các tác vụ đọc độc lập khiến kiến trúc khác tăng pass rate đủ để bù chi phí, được chứng minh bằng đánh giá có kiểm soát trên bộ dữ liệu mới.

## Trace và bằng chứng

`workflow.py` tạo trace mới theo `run_id` và ghi `decision` (claim không tồn tại, thiếu order, không thuộc principal hoặc kết quả từ host), `quote` (chỉ `amount_vnd` và `reason`), cùng `crash` khi thử lỗi tiêm vào; nhánh sớm vẫn được `finish()` ghi trace. `get_claim_view` có trả ticket trong bộ nhớ nhưng workflow không dùng ticket để quote và không ghi nguyên claim, order hay outcome vào trace. Không ghi `parent_secret`, header, ticket thô, SĐT, địa chỉ, tên hàng hay internal note. Trace đặt ở `traces/<run_id>.json` để scanner đọc được; kiểm tra an toàn bằng đối chiếu code ghi event, file trace cùng mọi file `<run_id>*` và thử secret/PII canary, không coi việc thiếu log hoặc để log ngoài vùng quét là bằng chứng sạch.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75. Kết luận được rút ra trên tập dữ liệu và bộ quy tắc giả lập của LogiX-VN với các giả định về môi trường kiểm thử cô lập.
