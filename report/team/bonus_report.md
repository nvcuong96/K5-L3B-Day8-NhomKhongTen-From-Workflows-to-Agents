# Bonus report: B1

- Topic + nguồn (tên, năm, slide): B1 · Investigator song song và critical path; Google Research (2026), *Towards a Science of Scaling Agent Systems*, slides 15, 24, 33, 41. File thực tế trong repo là `2512.08296v3.pdf`; không có file `2512.0829v3.pdf`.
- CLAIM của nguồn (1 câu): Multi-agent có lợi khi task chia song song được; task tuần tự có thể chịu overhead và giảm hiệu năng.
- BOUNDARY (model / dataset / setup): Nguồn dùng 260 cấu hình trên 6 benchmark, nhiều LLM và kiến trúc; thí nghiệm này chỉ dùng MockLLM, 14 public claims và SQLite offline.
- HYPOTHESIS trên LogiX: Investigator đọc order, policy và refund history có thể chạy song song; quote vẫn phụ thuộc order. Critical path giảm nhưng cost/success có thể tăng.
- Biến cố định / biến thay đổi: Giữ claim, seed, MockLLM, tools, pricing, evaluator và host; chỉ thay lịch investigator tuần tự thành song song.
- Lệnh tái lập (copy-paste được):
  `python -m lab8.benchmark --shapes orchestrator,orchestrator_parallel --extra-shape orchestrator_parallel=bonus.orchestrator_parallel:run_claim --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json`
  `python -m bonus.latency_report`
  `python -m pytest -q tests/test_bonus_orchestrator_parallel.py`
- File report + run id: `src/reports/bonus_benchmark.json`, `src/reports/bonus_latency.json`; `cp5-b1-20261117`.

## Kết quả

| Variant | Pass rate | Pass^5 | Variance | Cost/success (VND) | Mean/P95 steps |
| --- | ---: | ---: | ---: | ---: | ---: |
| orchestrator tuần tự | 1.000 | 1.000 | 0.000000 | 55.71 | 4.36 / 5 |
| orchestrator_parallel | 1.000 | 1.000 | 0.000000 | 74.29 | 4.36 / 5 |

| Critical path model | Mean | P95 |
| --- | ---: | ---: |
| Tuần tự | 26 ms | 26 ms |
| Song song | 14 ms | 14 ms |

Ledger, quote và status giống nhau bất kể thread nào hoàn thành trước; test
đảm bảo mỗi thread có SQLite connection riêng và chỉ writer chính gọi
`issue_refund`.

## Kết luận

Kết quả **ủng hộ một phần** hypothesis: ba investigator đọc độc lập giúp
critical path mô phỏng giảm 46.2%, trong khi pass rate và ledger không đổi.
Tuy nhiên cost/success tăng 33.3%, nên trên corpus nhỏ này không đủ lý do đổi
workflow/orchestrator tuần tự sang parallel. Phần thực sự song song là các
read-only lookup; bước `compute_compensation` phụ thuộc order, còn
`issue_refund` phải tuần tự sau quote và là writer duy nhất. Kết luận sẽ đảo
nếu latency thật lớn hơn chi phí tool bổ sung hoặc workload có nhiều lookup
độc lập hơn.

## Giới hạn

MockLLM không đo latency; latency ở đây là mô hình deterministic trong
`bonus/latency.yaml`, không phải thời gian wall-clock. Chỉ có 14 claims và 5
seed; chưa có hidden result. Chi phí là VND mô phỏng, không phải hóa đơn API.
