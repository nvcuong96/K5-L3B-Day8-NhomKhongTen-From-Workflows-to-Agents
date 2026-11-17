# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 55e92d0
- Run id dẫn chứng: cp5-public-20261117
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000-1.000 / 0.000000
- Cost per success (mô phỏng, VND): 27.86
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Workflow được chọn vì đạt pass rate và pass^5 = 1.000 trên 14 claim x 5 seed,
seed variance = 0 và có cost/success thấp nhất (27.86 VND) trong
`src/reports/benchmark.json`. Single cũng đạt 1.000 nhưng đắt hơn 31.63 VND;
orchestrator đạt 1.000 nhưng đắt hơn 55.71 VND; swarm có một lỗi seed 55.

## Có đáng đổi lấy multi-agent không

Nhóm giữ workflow vì không có lợi ích pass để bù cho chi phí cao hơn của
multi-agent. Quyết định sẽ đảo nếu hidden corpus cho thấy workflow dưới ngưỡng
0.80, hoặc có nhiều claim cần phân tích độc lập khiến lợi ích latency của B1
vượt chi phí và vẫn giữ safety counts bằng 0.

## Trace và bằng chứng

Trace ghi claim flags, quote, host result và crash marker khi có crash. Nhóm
không ghi ticket, parent secret, raw headers, PII hay toàn bộ claim/order row;
handoff dùng allowlist. Bằng chứng là `python -m pytest -q` và crash suite
20/20, cùng các test trace/handoff/secret trong `src/tests/`.

## Giới hạn

MockLLM không phải model sản xuất; benchmark public không thay thế hidden
40 x 5. Latency chính thức là unavailable, nên latency B1 chỉ là mô hình
offline riêng và không được dùng để sửa pass rate.
