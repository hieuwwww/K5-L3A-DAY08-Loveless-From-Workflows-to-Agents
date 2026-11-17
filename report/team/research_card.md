# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R1 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Google Research 2026, Towards a Science of Scaling Agent Systems, benchmark & empirical report |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Gia tăng số lượng agent và độ phức tạp topology trên tác vụ tuần tự không làm tăng tỷ lệ thành công mà làm tăng tuyến tính chi phí và rủi ro phương sai. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Tác vụ xử lý bồi hoàn logistics LogiX-VN với dữ liệu có cấu trúc và luật nghiệp vụ xác định, 14 claims public trên 5 seeds. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu chuyển từ Workflow (A) sang Orchestrator (C) hoặc Swarm (D), pass rate không tăng mà cost/success tăng lần lượt khoảng 36% và 78%. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ cố định: MockLLM backend, bộ công cụ đọc, 5 seeds (11,22,33,44,55), 14 claims public, pricing manifest. Biến duy nhất thay đổi: kiến trúc topology (A/B/C/D). |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Đo bằng pass rate, pass^5, seed variance, và cost/success (VND). Dự kiến Swarm fail ở seed 55 do tin lời ticket làm variance tăng. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
- Run id / file report: src/reports/benchmark.json
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Đúng hoàn toàn so với giả thuyết. Workflow đạt pass rate 1.000, pass^5 1.000, variance 0.000000 với chi phí thấp nhất 27.86 VND/success. Orchestrator tăng chi phí lên 38.08 VND (+36.7%). Swarm tăng chi phí lên 49.57 VND (+77.9%) và bị sụt giảm pass rate xuống 0.986, pass^5 xuống 0.929 (variance = 0.000816 do seed 55 bị ảo giác).
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
