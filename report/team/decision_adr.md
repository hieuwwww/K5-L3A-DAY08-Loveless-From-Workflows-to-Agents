# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: a7b6f92
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000-1.000 / 0.000000
- Cost per success (mô phỏng, VND): 27.86
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

## Vì sao shape này

Workflow đạt hiệu suất tuyệt đối trên toàn bộ 14 public claims qua cả 5 seed (pass rate = 1.000, pass^5 = 1.000, variance = 0.000000) với chi phí thấp nhất trong cả 4 kiến trúc (27.86 VND/success, trung bình chỉ 1.64 bước). Trong khi đó, Swarm có chi phí cao nhất (49.57 VND/success, gấp 1.78 lần) và bị trượt ở seed 55 do mô hình bị ảnh hưởng bởi nội dung chém gió trong ticket (variance = 0.000816, pass^5 = 0.929). Đối với bài toán xử lý claim logistics vốn có dữ liệu có cấu trúc và luật nghiệp vụ rõ ràng, Workflow thể hiện rõ nguyên tắc "ít tự chủ nhất mà vẫn làm được việc": tính xác định cao (deterministic), loại trừ hoàn toàn nguy cơ ảo giác và prompt injection, đồng thời tiết kiệm tài nguyên tính toán tối đa.

## Có đáng đổi lấy multi-agent không

Không đáng đổi. Khi so sánh giữa Workflow và các kiến trúc agent phức tạp hơn (Single ReAct: 29.98 VND; Orchestrator: 38.08 VND; Swarm: 49.57 VND), các kiến trúc agent không mang lại bất kỳ sự gia tăng nào về tỷ lệ thành công (pass rate không cao hơn Workflow) nhưng lại làm tăng đáng kể chi phí token/tool và độ trễ, thậm chí Swarm còn làm tăng rủi ro phương sai (variance > 0 do seed 55). Nhóm chỉ cân nhắc chuyển sang kiến trúc multi-agent nếu bài toán phát sinh các yếu tố phi cấu trúc phức tạp mà code tĩnh không thể xử lý, ví dụ: chính sách bồi thường thay đổi động bằng văn bản ngôn ngữ tự nhiên không có API tra cứu chuẩn hóa, cần đàm phán thương lượng linh hoạt với khách hàng, hoặc cần tổng hợp dữ liệu từ nhiều nguồn phi đồng nhất chưa có schema xác định.

## Trace và bằng chứng

Trace của Workflow ghi nhận các sự kiện chính theo Trace Contract v1: `claim_view` (thông tin claim đã qua lọc, chỉ ghi nhận mã claim và cờ có order_id hay không), `decision` (các nhánh kết thúc sớm như `needs_info`, `escalated`, `denied`), `quote` (kết quả tính toán bồi hoàn từ `compute_compensation`), `host_outcome` (kết quả từ `issue_refund` của host), và `crash` (khi kiểm thử điểm sự cố). Trace cố ý **không ghi** các trường nhạy cảm: `parent_secret`, `ticket` thô, thông tin định danh cá nhân (PII như họ tên, số điện thoại, địa chỉ nhận hàng, ghi chú nội bộ của đơn hàng). Nếu trace nằm ngoài vùng scanner quét trực tiếp, đội ngũ an ninh thông tin có thể chứng minh tính an toàn bằng cách đối chiếu log với mã nguồn để xác minh tất cả các điểm ghi trace đều đi qua các hàm allowlist (`get_claim_view`, `lookup_order`, `safe_dump`), chứng minh không có bất kỳ lệnh ghi chuỗi thô hay câu lệnh SQL đọc trực tiếp bảng chứa secret nào.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75. Kết luận được rút ra trên tập dữ liệu và bộ quy tắc giả lập của LogiX-VN với các giả định về môi trường kiểm thử cô lập.
