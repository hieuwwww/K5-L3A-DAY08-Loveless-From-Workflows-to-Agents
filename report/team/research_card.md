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
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Kim và cộng sự, *Towards a Science of Scaling Agent Systems*, bản arXiv v3 (2026), [arXiv:2512.08296v3](https://arxiv.org/abs/2512.08296v3), nghiên cứu benchmark có kiểm soát.[1] |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Trên các benchmark/model của bài báo, lợi ích phối hợp phụ thuộc cấu trúc tác vụ: so với single-agent, tác vụ tài chính có thể tăng +80,8% còn sequential planning giảm đến −70,0%; không suy ra mọi tác vụ tuần tự đều giảm.[1] |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Bài báo đo các kiến trúc agent trên benchmark và họ LLM của nghiên cứu, không đo LogiX-VN; thử nghiệm lab riêng dùng MockLLM, 14 claim public và 5 seed, không suy rộng sang production.[1] |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Trên LogiX-VN public, chuyển A → C/D không tăng pass rate; cost/success C cao hơn A khoảng 36%, D cao hơn khoảng 78% (dự đoán cho lab, không phải con số của bài báo). |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | So sánh A/B/C/D bằng cùng MockLLM, corpus 14 claim, seed 11,22,33,44,55, bảng giá và evaluator; biến so sánh là shape. Lưu ý shape cũng khác số lần gọi model/tool và cách dùng tool, nên chưa thể quy mọi chênh lệch *chỉ* cho topology. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Đo bằng pass rate, pass^5, seed variance, và cost/success (VND). Dự kiến Swarm fail ở seed 55 do tin lời ticket làm variance tăng. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
- Run id / file report: `R-workflow-C-001-s11` (trial trong `src/reports/benchmark.json`; chạy riêng bằng `python -m lab8.run --shape workflow --claim-id C-001 --seed 11 --db data/research_card_C001.sqlite` từ `src/`, kết quả `resolved`, refund 35.000 VND, một audit). Run id này minh họa một trial; số liệu so sánh lấy từ toàn bộ report 280 trial. Dùng DB mới khi chạy lại lệnh riêng.
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Phù hợp với dự đoán trên public: Workflow pass rate 1.000, pass^5 1.000, variance 0.000000 và 27.86 VND/success; Orchestrator 38.08 (+36.7%); Swarm 49.57 (+77.9%), pass rate 0.986, pass^5 0.929, variance 0.000816 (sai C-002 ở seed 55). Không thể tách riêng ảnh hưởng topology khỏi cách dùng tool/model bằng phép so sánh này.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).

## Sources

[1] https://arxiv.org/abs/2512.08296v3 — Towards a Science of Scaling Agent Systems (v3)
