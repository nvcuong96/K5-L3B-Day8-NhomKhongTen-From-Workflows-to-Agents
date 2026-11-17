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
|   3 | Võ Hồng Lương | 2A202603839 | 26ai.luongvh@vinuni.edu.vn | luongvh2510 | Orchestrator + handoff |
|   4 | Nguyễn Đức Mạnh | 2A202603731 | 26ai.manhnd8@vinuni.edu.vn | nguyenducmanh-itus | Workflow + intergration |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Văn Hiếu | Single ReAct agent | 451c154 | tests/test_single_agent.py |
| Võ Hồng Lương | Orchestrator + handoff | c394196, 30007b2 | tests/test_orchestrator_handoff.py |
| Nguyễn Đức Mạnh | Workflow | 17315ef | tests/test_workflow_cp4.py |

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

## 2A202603731

### Phần việc phụ trách

- Shape A : Workflow (`src/lab8/shapes/workflow.py`)

### Đóng góp cụ thể

- Hoàn thiện luồng workflow dùng `get_claim_view`, `lookup_order`, `compute_compensation` và `durability.issue_refund`; giữ nguyên quote đầy đủ khi gọi host.
- Bổ sung event quyết định/kết quả host vào trace; khi crash được tiêm, trace được ghi và `InjectedCrash` tiếp tục được ném để runner có thể resume.
- Thêm test CP4 xác nhận approval thuộc claim khác bị từ chối, không tạo refund, có audit `policy_denied` với lý do `approval_claim_mismatch`, và secret không xuất hiện trong trace/stdout.

### Bằng chứng

- Shape workflow : `src/lab8/shapes/workflow.py`.
- Test mới: `src/tests/test_workflow_cp4.py` — `1 passed`.
- Kiểm tra workflow trên T01–T10, T13–T14 với 5 seed: `60/60`; T11 retry và T12 crash/resume cũng đạt trong lượt tự kiểm.
- Crash suite: `src/reports/crash.json` báo `crash_safety: 20/20`.
