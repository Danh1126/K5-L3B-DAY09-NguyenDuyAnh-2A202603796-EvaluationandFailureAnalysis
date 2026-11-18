# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (`PASS floors`) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (`FAIL floors`: κ < 0.6, R < 0.7) | loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | `H-008` (`tone`) | PASS | FAIL | Người đúng | Reply dùng văn phong quá suồng sã ("Em đã hoàn tiền cho anh rồi nha, anh check giúp em"), thiếu chuẩn mực CSKH. Cần bổ sung rule cấm từ lóng/suồng sã ("nha", "check") vào rubric `tone_ok`. |
| 2 | `H-017` (`tone`) | PASS | FAIL | Người đúng | Dùng "bạn ơi" và "rồi nha", judge bị đánh lừa vì có chữ "Dạ," ở đầu câu. Cần cập nhật rubric: có "Dạ" nhưng thân câu cợt nhả vẫn tính là `FAIL`. |
| 3 | `H-029` (`tone`) | PASS | FAIL | Người đúng | Câu "không cần nhắn thêm" mang sắc thái đuổi khách/cộc lốc dù mở đầu bằng "Dạ, vui lòng chờ". Rubric cần cấm các cụm từ chặn giao tiếp khách hàng. |
| 4 | `H-042` (`tone`) | FAIL | PASS | Người đúng | Reply lịch sự, đầy đủ thông tin ("Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."), judge bắt nhầm (FP). Cần thêm ví dụ mẫu câu ngắn gọn nhưng lịch sự chuẩn vào rubric. |
| 5 | `HL-016` (`helpful`) | PASS | FAIL | Người đúng | Reply chỉ xin lỗi sáo rỗng dài dòng mà không nêu trạng thái xử lý hay bước tiếp theo cho khách, nhưng judge vẫn chấm PASS vì thấy từ khóa "xin lỗi". Rubric `helpful` cần yêu cầu có thông tin/hành động giải quyết cụ thể. |
| 6 | `HL-015` (`helpful`) | PASS | FAIL | Người đúng | Reply báo hoàn 55.000đ nhưng không kèm mốc thời gian nhận tiền hoặc hướng dẫn kiểm tra lại; judge quá dễ dãi khi chỉ kiểm tra có đề cập số tiền. |

## Kết luận
- Tiêu chí bị loại và lý do (dẫn ma trận nhầm lẫn): Loại bỏ `helpful` (`κ = 0.375 < 0.6`, `Recall = 0.300 < 0.7`). Từ ma trận nhầm lẫn (`tp=9, fp=0, fn=21, tn=70`), judge bỏ sót tới 21/30 ca lỗi thật (`FN = 21`, tức 70% ca không hữu ích bị lọt qua như PASS), khiến tỉ lệ lỗi quan sát được chỉ là `0.09` trong khi tỉ lệ lỗi thực tế sau hiệu chỉnh Rogan-Gladen lên tới `0.30`. Nếu dùng làm CI gate sẽ để lọt các bản agent trả lời sáo rỗng, vô dụng.
- Rủi ro còn lại: Judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới để có held-out thật.
