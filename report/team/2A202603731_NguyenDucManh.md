# Báo cáo cá nhân — MSSV_NguyenDucManh

> Bản nháp cá nhân. Thay `MSSV` bằng mã sinh viên chính xác trước khi nộp.

## 1. Thông tin thành viên

- Họ tên: Nguyễn Đức Mạnh
- MSSV: 2A202603731
- Email: manhgu3110@gmail.com
- GitHub: nguyenducmanh-itus

## 2. Phần việc phụ trách

- Shape `workflow`: triển khai luồng claim theo business rules, gọi host để kiểm approval và ghi refund, đồng thời bảo đảm trace không chứa dữ liệu nhạy cảm.
- CP3–CP4: kiểm tra crash/resume, idempotency, nhánh approval bị từ chối và secret trong trace/stdout.

## 3. Đóng góp cụ thể

- Hoàn thiện luồng workflow dùng `get_claim_view`, `lookup_order`, `compute_compensation` và `durability.issue_refund`; giữ nguyên quote đầy đủ khi gọi host.
- Bổ sung event quyết định/kết quả host vào trace; khi crash được tiêm, trace được ghi và `InjectedCrash` tiếp tục được ném để runner có thể resume.
- Thêm test CP4 xác nhận approval thuộc claim khác bị từ chối, không tạo refund, có audit `policy_denied` với lý do `approval_claim_mismatch`, và secret không xuất hiện trong trace/stdout.
- Commit chính: "Update shape workflow".

## 4. Bằng chứng
- Shape workflow : `src/lab8/shapes/workflow.py`.
- Test mới: `src/tests/test_workflow_cp4.py` — `1 passed`.
- Kiểm tra workflow trên T01–T10, T13–T14 với 5 seed: `60/60`; T11 retry và T12 crash/resume cũng đạt trong lượt tự kiểm.
- Crash suite: `src/reports/crash.json` báo `crash_safety: 20/20`.
- Run id tham khảo cho resume T12: `R-workflow-C-012-s11`.
- Lệnh CP4: `python -m lab8.crash_test --count 20 --out reports/crash.json`.

## 5. Công cụ AI đã dùng

- Dùng Codex để đọc business rules/contracts, rà luồng workflow, đề xuất và viết test CP4.
- Tự kiểm tra bằng test mới, ma trận public workflow, kiểm tra T11/T12 và crash suite. Test mới chạy thành công; crash suite đạt 20/20.

## 6. Tự đánh giá

- Đã kiểm chứng các nhánh approval và crash/resume bằng harness của repo. Cần chạy lại `pytest` trong môi trường `.venv` chuẩn của nhóm trước khi chốt.
- Cần bổ sung MSSV, email, GitHub và commit hash; đối chiếu lại tên hiển thị trước khi nộp.
- Nắm được vì sao workflow phải gửi quote đầy đủ cho host: host mới áp dụng approval, idempotency và mức cap theo order một cách nhất quán.
