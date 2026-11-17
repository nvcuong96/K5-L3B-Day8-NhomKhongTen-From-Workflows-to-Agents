# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-TenNhom-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 |        |      |       |        |         |
|   2 | Nguyễn Văn Hiếu | 2A202603746 | 26ai.hieunv3@vinuni.edu.vn | lunaHieu | Single ReAct agent |
|   3 | Nguyễn Đức Mạnh | 2A202603731 | 26ai.

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Văn Hiếu | Single ReAct agent | 451c154 | tests/test_single_agent.py |
| Nguyễn Đức Mạnh | Workflow | 

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603746_NguyenVanHieu

### Phần việc phụ trách

- Shape B: Single ReAct agent (`src/lab8/shapes/single.py`).

### Đóng góp cụ thể

- Triển khai vòng lặp ReAct tối đa 8 bước với `run_react`, model chỉ gọi read tools (`get_claim_view`, `lookup_order`, `compute_compensation`).
- Code Python gọi `durability.issue_refund` sau vòng ReAct, truyền full quote để host tự xử lý trần theo đơn (T13, T14).
- Truyền `crash_at` vào `issue_refund`, không bắt `InjectedCrash` để đảm bảo durability và idempotency.
- Lấy `delay_days` từ database thay vì tin vào câu chữ trong ticket, vô hiệu hóa prompt injection (vượt qua T02 ở seed 55).
- Thêm bộ test `src/tests/test_single_agent.py` kiểm chứng toàn diện 6 kịch bản kỹ thuật của Single Agent.

### Bằng chứng

- `pytest -q tests/test_single_agent.py`: 6 passed.
- `pytest -q -m student -k "single and not workflow and not orchestrator and not swarm"`: 62 passed.
- Cổng host `pytest -q`: 60 passed.
- `python -m lab8.crash_test --count 20 --out reports/crash.json`: crash_safety 20/20.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-
