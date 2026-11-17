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
| CÂU | R1 / R2 / R3 / R4 | R1 · Scaling |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Google Research, 2026, `2512.08296v3.pdf`, benchmark 260 cấu hình trên 6 benchmark |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Coordination có diminishing returns khi baseline single-agent đã mạnh; task tuần tự có thể giảm khi thêm multi-agent. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Kết quả của paper dùng các benchmark/LLM/kiến trúc được nghiên cứu; không tự động suy ra cho LogiX, MockLLM hoặc mọi task. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Bật investigator song song giữ nguyên pass rate/ledger nhưng giảm critical path khoảng 40–50%; cost/success có thể tăng do thêm tool calls. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ MockLLM, tools, seeds 11/22/33/44/55, 14 public claims, pricing và evaluator; chỉ đổi lịch chạy investigator tuần tự → song song. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến là thread-order làm khác quote/ledger hoặc chi phí tăng; đo pass rate, pass^5, variance, cost/success, critical-path mean/p95 và safety counts. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes orchestrator,orchestrator_parallel --extra-shape orchestrator_parallel=bonus.orchestrator_parallel:run_claim --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json` và `python -m bonus.latency_report`
- Run id / file report: `cp5-b1-20261117` / `src/reports/bonus_benchmark.json`, `src/reports/bonus_latency.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Đúng một phần. Cả hai đạt pass rate/pass^5 1.000, variance 0, mean/p95 steps 4.36/5. Parallel có critical path 14/14 ms so với tuần tự 26/26 ms, nhưng cost/success tăng 55.71 → 74.29 VND vì tool calls trung bình 5.57 → 7.43. Chưa kết luận được latency production vì latency mô phỏng.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
