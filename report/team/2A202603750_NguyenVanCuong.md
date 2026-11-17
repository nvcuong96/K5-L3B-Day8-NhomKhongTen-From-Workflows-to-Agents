# Báo cáo cá nhân — 2A202603750_NguyenVanCuong

## 1. Thông tin thành viên

- Họ tên: Nguyễn Văn Cường
- MSSV: 2A202603750
- Email: 26ai.cuongnv2@vinuni.edu.vn
- GitHub: nvcuong96

## 2. Phần việc phụ trách

- Ledger và evaluation.
- Hoàn thiện CP5 benchmark, báo cáo nhóm và bonus B1 về investigator song song
  và critical path.

## 3. Đóng góp cụ thể

- Chạy benchmark 4 shape trên cùng 14 public claims và 5 seed, tạo
  `src/reports/benchmark.json`.
- Chọn workflow làm shape chính dựa trên pass rate 1.000, variance 0 và
  cost/success thấp nhất 27.86 VND.
- Viết `src/bonus/orchestrator_parallel.py`: các investigator chỉ đọc chạy
  bằng `ThreadPoolExecutor`, mỗi thread dùng SQLite connection riêng, kết quả
  được join theo thứ tự cố định và chỉ writer chính gọi `issue_refund`.
- Viết mô hình latency deterministic trong `src/bonus/latency.yaml` và báo cáo
  critical path; kết quả mô phỏng giảm từ 26 ms xuống 14 ms.
- Thêm test B1 kiểm tra critical path và tính tương đương status, quote, ledger
  giữa orchestrator tuần tự và song song.
- Hoàn thiện `decision_adr.md`, `research_card.md`, `group_report.md` và
  `bonus_report.md`.

## 4. Bằng chứng

- `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
  tạo benchmark 4 shape x 14 claim x 5 seed.
- `python -m lab8.benchmark --shapes orchestrator,orchestrator_parallel --extra-shape orchestrator_parallel=bonus.orchestrator_parallel:run_claim --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json`
  cho cả hai biến thể pass rate/pass^5 = 1.000 và variance = 0.
- `python -m bonus.latency_report` tạo `reports/bonus_latency.json`, ghi
  sequential 26/26 ms và parallel 14/14 ms (mean/p95).
- `python -m pytest -q tests/test_bonus_orchestrator_parallel.py`: **2 passed**.
- `python -m pytest -q`: **173 passed, 248 deselected**.
- `55e92d0`: commit CP5 chứa code bonus, benchmark, báo cáo và test B1.

## 5. Hạn chế và tự đánh giá

Latency chỉ là mô hình deterministic, không phải wall-clock latency của
production. Parallel variant giảm critical path nhưng tăng cost/success từ
55.71 lên 74.29 VND vì số read-tool calls tăng. Kết luận vì vậy chỉ ủng hộ
song song trong workload có latency đủ lớn hoặc nhiều investigator độc lập,
không khẳng định multi-agent luôn tốt hơn.
