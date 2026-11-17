# Báo cáo cá nhân — 2A202603839_VoHongLuong

## 1. Thông tin thành viên

- Họ tên: Võ Hồng Lương
- MSSV: 2A202603839
- Email: 26ai.luongvh@vinuni.edu.vn
- GitHub: luongvh2510

## 2. Phần việc phụ trách

Thành viên 3 — **Shape C: Orchestrator và handoff** (theo [docs/team_work_plan.md](../../docs/team_work_plan.md)).

- Triển khai `src/lab8/shapes/orchestrator.py`: orchestrator lập kế hoạch, giao việc cho investigator chỉ có tool đọc qua handoff allowlist, rồi tự tính quote và là bên duy nhất gọi `durability.issue_refund`.
- Bảo đảm handoff và trace là ranh giới bảo mật: không mang `parent_secret`, header, claim row thô hay PII của order qua ranh giới.
- Viết bộ test `src/tests/test_orchestrator_handoff.py` và dùng nó để rà trace của cả ba shape nhóm viết (`workflow`, `single`, `orchestrator`) trước CP4.

## 3. Đóng góp cụ thể

**Commit chính**

| Commit | Nội dung |
| --- | --- |
| `c394196` | `done orchestrator shape` — cài đặt shape C (`src/lab8/shapes/orchestrator.py`) |
| `30007b2` | `test(orchestrator): handoff allowlist, trace leak scan for 3 shapes, full quote, crash resume` — thêm `src/tests/test_orchestrator_handoff.py` |

Cả hai commit vào `main` qua PR #1 (merge commit `d94ae1b`).

**Luồng xử lý của orchestrator** (mọi nhánh return đi qua `common.finish()` nên luôn có trace đúng contract v1):

1. Đọc claim bằng `get_claim_view` (không `SELECT *`). Không tồn tại → `failed / unknown_claim`; thiếu `order_id` → `needs_info`, không đoán mã đơn từ ticket.
2. Dựng handoff bằng `make_handoff` với `facts={}` và `allowed_tools = [lookup_order, lookup_policy, lookup_refund_history, compute_compensation]` — toàn bộ thuộc `READONLY_TOOLS`, `lookup_order` đứng trước `compute_compensation` vì investigator chạy tuần tự.
3. `run_readonly_investigators` chạy các tool đọc; `principal_id` truyền qua tham số để lọc scope, không nằm trong handoff hay trace.
4. Đơn không thấy / không thuộc principal → `escalated`. Quote do code tính (`compute_compensation` từ order row + policy hiện hành); quote = 0 → `denied`, không gọi host.
5. Gọi `issue_refund(conn, claim_id, <quote đầy đủ>, crash_at=crash_at)`. Bắt `InjectedCrash` chỉ để ghi trace rồi `raise` lại.
6. Host từ chối → `denied` (hoặc `failed` khi `conflict`), `proposal_amount` vẫn là quote đầy đủ; host chấp nhận → `resolved`.

**Quyết định kỹ thuật**

- **`facts={}` trong handoff:** lúc dựng handoff orchestrator chưa có dữ liệu order; investigator tự lấy bằng tool đọc. Để rỗng loại bỏ hẳn rủi ro đưa claim row/ticket vào handoff.
- **Truyền quote đầy đủ cho host:** host tự trừ phần các claim trước đã nhận trên cùng đơn. `lookup_refund_history` chỉ dùng để quan sát và ghi trace, không dùng để tự trừ (tự trừ thì T14 bị trừ hai lần).
- **Không tin số ngày trễ trong ticket:** không sao chép khối `REFERENCE_WEAKNESS` của swarm, nên T02 đúng ở cả 5 seed.
- **Không gọi MockLLM:** đề không yêu cầu, grader không đọc câu chữ, và MockLLM không được quyết tiền hay trạng thái. Bỏ lời gọi model giữ `cost/success` thấp; hệ quả là về hành vi shape C gần với workflow, chỉ khác ở cấu trúc handoff + investigator (đã báo cho người viết ADR).
- **Trace cố ý không ghi:** `ticket`, `principal_id`, nguyên claim row, nguyên order row (SĐT, địa chỉ, tên hàng, internal note), `parent_secret`, `raw_headers`, và `effect` của host. Trace chỉ có: cờ `found`/`has_order_id`, handoff đã sanitize, order đã qua allowlist `ORDER_FIELDS`, quote (`amount_vnd`, `reason`, `clause_ids`), cờ kết quả host (`ok`, `reason`, `conflict`), và điểm crash nếu có.

**Test đã thêm — `src/tests/test_orchestrator_handoff.py` (110 test, chạy trong cổng host mặc định)**

| Nhóm | Kiểm tra |
| --- | --- |
| Secret/PII cả 3 shape | 3 shape × 14 claim × seed 11, 55: quét trace + mọi file `<run_id>*` + `RunResult` tìm `forbid_substrings` của claim, `CANARY_SECRET_`, `Bearer`, `parent_secret`, `raw_headers`, và SĐT/địa chỉ/ghi chú nội bộ/tên hàng của mọi order |
| Handoff | Investigator chỉ có tool đọc và đúng thứ tự; `make_handoff` từ chối `issue_refund`/`append_audit`; event `handoff` chỉ có field `ALLOWED_FIELDS`; trace không có ticket/`principal_id` |
| Trace contract v1 | 6 nhánh `failed`, `needs_info`, `escalated`, `denied` (quote 0), `denied` (host từ chối), `resolved`: file ghi trong lần chạy, `run_id` khớp |
| Tiền | Spy `issue_refund`: T13 gửi 35.000, T14 gửi 150.000 (quote đầy đủ); T02 `denied` ở 5 seed; quote 0 không gọi host; C-007, C-013 có audit `policy_denied` |
| Crash | 3 điểm crash: `InjectedCrash` được ném, resume ra đúng 1 refund + 1 audit `refund_committed`, chạy lại vẫn idempotent |

## 4. Bằng chứng

Mọi lệnh chạy trong `src/`, trên `main` tại commit `afab602` (đã có đủ 3 shape của nhóm).

| Lệnh | Kết quả |
| --- | --- |
| `python -m pytest -q -m student -o addopts= -k orchestrator` | 62 passed (T01–T14 × 5 seed, T11, T12) |
| `python -m pytest -q tests/test_orchestrator_handoff.py` | 110 passed |
| `python -m pytest -q` (cổng host) | 171 passed |

- Rà trace trước CP4: nhóm test secret/PII ở trên chạy trên code thật của `workflow` và `single` sau khi merge → 0 secret leak, 0 PII leak ở cả 3 shape.
- Kiểm chứng test bắt được lỗi thật: tạm sửa orchestrator để (a) tự trừ `refunded_vnd` trước khi gọi host và (b) ghi ticket vào trace → 6 test đỏ (full quote T13/T14, không ghi ticket, 3 test crash/resume). Đã khôi phục file, `git diff` sạch, chạy lại 110/110 xanh.
- Run id mẫu (seed 11): `R-orchestrator-C-001-s11` → `resolved`, proposal 35.000, ledger 35.000; `R-orchestrator-C-014-s11` → `resolved`, proposal 150.000, ledger 120.000 (host chỉ ghi phần chênh). Mỗi lần chạy: 0 model call, 6 tool call, cost mô phỏng 60 VND.
- Số liệu benchmark 4 shape chính thức nằm trong `src/reports/benchmark.json` và [decision_adr.md](decision_adr.md) do thành viên 4 tổng hợp.

## 5. Công cụ AI đã dùng

- **Công cụ:** Claude Code (Anthropic).
- **Dùng vào việc:**
  - Đọc và tóm tắt đề, `docs/business_rules.md`, `docs/contracts.md`, các module host (`handoff.py`, `runtime.py`, `tools.py`, `durability.py`) để xác định file được/không được sửa.
  - Tách `orchestrator.py` thành template có TODO đánh số; viết bộ test `test_orchestrator_handoff.py`.
- **Cách tự kiểm tra:**
  - Tự chạy cổng host, ma trận student của orchestrator và bộ test riêng trong `src/.venv` sau mỗi bước điền TODO.
  - Đọc trực tiếp file trace JSON của C-001 (claim có `CANARY_SECRET_PARENT_C-001`) để xác nhận handoff chỉ có field allowlist.
  - Cố ý làm hỏng code (tự trừ quote, ghi ticket vào trace) để xác nhận test thực sự đỏ, rồi khôi phục.
  - Đối chiếu lại với `RULES.md`: không sửa file thước đo, không sửa tay JSON trong `src/reports/`; so `git diff upstream/main` chỉ thấy 3 file shape thay đổi trong `src/lab8`.
  - Số liệu trong báo cáo này lấy từ lệnh đã chạy, không do AI tự đặt ra.

## 6. Tự đánh giá

- Hoàn thành phần việc thành viên 3: shape C qua 62/62 test student, handoff allowlist, investigator chỉ đọc, trace sạch, T11–T14 đúng; bộ test riêng 110 test giữ trong cổng host để chặn hồi quy.
- Hoàn thành việc rà trace cả 3 shape trước CP4 bằng test tự động thay vì đọc tay.
- Hạn chế:
  - Lệnh `lab8.crash_test` của repo chỉ chạy swarm, nên crash safety của orchestrator được chứng minh qua T12 và test crash 3 điểm của tôi, không qua `crash.json`.
