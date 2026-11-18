# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | seeded_regression C-002@11, run ID E-seeded_regression-20261118-7c10 |
| Triệu chứng | TrialPass (pass_rate) và RefundAmountCorrect fail: reply nói "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." và ledger ghi refund_vnd=20000, refund_count=1, trong khi nhãn chuẩn là status=denied, refund_vnd=0, refund_count=0. |
| Bước sai đầu tiên | Event #3 (model_override): {"delay_days": 5} |
| Vì sao là bước *đầu tiên* | Event #1 (handoff) chỉ cấp danh sách allowed_tools chuẩn (["lookup_order", "lookup_policy", "compute_compensation"]) cho claim_id: C-002 và chưa có bất kỳ tính toán hay ghi sổ nào; tại event #3, model ghi đè delay_days: 5 theo lời khai trên ticket thay vì dùng dữ liệu DB, kéo theo event #4 (quote) tính nhầm amount_vnd: 20000 theo SHIP-LATE-03. |
| Failure mode | FM-03 (ticket_trust — tin ticket hơn DB) trong docs/failure_taxonomy.md |
| Đối chứng | Trên shape fixed (C-002@11): không có event model_override; sau event #1 (handoff), hệ thống đối chiếu dữ liệu DB chuẩn nên event #3 (quote) trả về {"amount_vnd": 0, "reason": "no_entitlement", "clause_ids": [], "parts": {}} và event #4 (reply) từ chối đúng chính sách (status=denied, refund_vnd=0). |
| Regression test | test_override_event_breaks_db_fact (DbFactKept) và test_wrong_amount_in_ledger (RefundAmountCorrect) trong src/tests/test_lab9_graders.py — bắt được lỗi trên seeded_regression và pass 100% trên fixed. |
| Chưa chắc chắn | Trace trên MockLLM chỉ ghi nhận sự kiện model_override, chưa chứng minh được trên LLM thật liệu lỗi này xuất phát từ prompt hệ thống thiếu ràng buộc ưu tiên DB hay do dữ liệu trả về từ tool lookup_order bị thiếu ngữ cảnh. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
