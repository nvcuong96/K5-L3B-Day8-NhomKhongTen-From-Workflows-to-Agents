# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Nhom Khong Ten
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): 55e92d0

## 2. Mục tiêu và cách tiếp cận

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

Nhóm triển khai workflow, single và orchestrator với ranh giới code/model rõ
ràng; host là nơi duy nhất kiểm approval, cap theo order và ghi ledger. B1
được triển khai riêng trong `src/bonus/` để so sánh investigator tuần tự và
song song mà không sửa host.

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

Workflow tính quote bằng dữ liệu có cấu trúc rồi gọi host. Single dùng ReAct
nhưng chỉ cho model gọi read tools. Orchestrator dựng handoff allowlist và
investigator chỉ đọc. B1 mở SQLite connection riêng cho mỗi investigator,
join theo thứ tự cố định và chỉ connection chính gọi `issue_refund`.

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

| Shape | Pass rate | Pass^5 | Seed min-max | Variance | Cost/success | Mean/P95 steps |
| --- | ---: | ---: | --- | ---: | ---: | ---: |
| workflow | 1.000 | 1.000 | 1.000-1.000 | 0.000000 | 27.86 | 3.79 / 4 |
| single | 1.000 | 1.000 | 1.000-1.000 | 0.000000 | 31.63 | 2.00 / 2 |
| orchestrator | 1.000 | 1.000 | 1.000-1.000 | 0.000000 | 55.71 | 4.36 / 5 |
| swarm | 0.986 | 0.929 | 0.929-1.000 | 0.000816 | 49.57 | 2.80 / 3 |

Workflow được chọn vì rẻ nhất mà vẫn đạt 1.000 trên public benchmark. Crash
safety là 20/20; approval bypass và secret leak đều 0.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

Khó khăn chính là parallel investigator làm tăng số tool calls dù critical
path giảm. Test B1 xác nhận thread completion order không làm thay đổi status,
quote hoặc ledger.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

Kết luận bị giới hạn bởi 14 public claims, MockLLM và latency mô phỏng. Nếu
hidden corpus có nhiều dữ kiện độc lập hoặc latency thật chi phối SLA, lựa
chọn có thể chuyển sang orchestrator song song; cần xác nhận thêm safety counts.

## 8. Tài liệu tham khảo

1. Google Research (2026), *Towards a Science of Scaling Agent Systems*,
   `2512.08296v3.pdf` (file trong repo; tên người dùng nêu `2512.0829v3.pdf`
   không tồn tại).
2. LogiX-VN lab contracts, business rules và benchmark reports trong repo.
