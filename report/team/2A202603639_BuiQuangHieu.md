# Báo cáo cá nhân — Bùi Quang Hiếu

## 1. Thông tin thành viên

- Họ tên: Bùi Quang Hiếu
- MSSV: 2A202603639
- Email: 26ai.hieubq@vinuni.edu.vn
- GitHub: https://github.com/hieuwwww

## 2. Phần việc phụ trách

Workflow + integration: triển khai shape A và kiểm tra tích hợp với host và bộ test public.

## 3. Đóng góp cụ thể

- Triển khai `src/lab8/shapes/workflow.py`: đọc claim/order có scope, quote từ dữ liệu order, truyền quote đầy đủ qua cổng host, trả `RunResult` và trace trên mọi nhánh.
- Thêm `src/tests/test_workflow_integration.py` để kiểm tra claim không tồn tại, trần hoàn tiền theo đơn và trace không lộ dữ liệu nhạy cảm.
- Commit kỹ thuật: `6b1faff` (`feat: implement workflow claim handling and integration tests`).

## 4. Bằng chứng

- Test: `python3 -m pytest -q tests/test_workflow_integration.py` và `python3 -m pytest -q -m student -o addopts= -k workflow` (chạy trong `src/`).
- Run id / báo cáo benchmark: cập nhật sau khi nhóm chạy benchmark chung.

## 5. Công cụ AI đã dùng

- OpenCode: hỗ trợ đọc tài liệu, triển khai workflow và viết test. Kiểm chứng bằng test integration, ma trận student của workflow và cổng host; số liệu thực tế ghi sau khi chạy.

## 6. Tự đánh giá

Đã hoàn thiện và kiểm thử phần workflow; tích hợp tổng thể bốn shape cần chạy lại sau khi các thành viên hoàn thành phần của họ.
