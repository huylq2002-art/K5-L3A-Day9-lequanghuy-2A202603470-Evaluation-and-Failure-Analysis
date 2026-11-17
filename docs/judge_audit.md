```markdown
# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có ($\kappa=0.757 \ge 0.6$, $P=0.857 \ge 0.8$, $R=0.800 \ge 0.7$) | **dùng** (non-blocking) |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không ($\kappa=0.375 < 0.6$, $R=0.300 < 0.7$) | **loại** |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | `HL-015` (helpful) | `PASS` | `FAIL` | **Người đúng** | Agent báo `"Dạ, LogiX đã hoàn 55.000đ cho đơn O-112 ạ."` nhưng thiếu thông tin thời gian nhận tiền/mã giao dịch và sai bối cảnh xử lý. **Sửa rubric:** Bắt buộc chấm `FAIL` nếu câu trả lời xác nhận hoàn tiền nhưng thiếu chi tiết đối chiếu giao dịch hoặc không giải quyết đúng yêu cầu gốc. |
| 2 | `HL-016` (helpful) | `PASS` | `FAIL` | **Người đúng** | Agent chỉ xin lỗi chung chung (`"Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này..."`) mà không đưa ra hướng xử lý hay bước tiếp theo. **Sửa rubric:** Quy định lời xin lỗi suông không kèm hành động giải quyết cụ thể (actionable next step) phải bị chấm `FAIL`. |
| 3 | `HL-020` (helpful) | `PASS` | `FAIL` | **Người đúng** | Agent báo hoàn `90.000đ` cho đơn `O-303` nhưng không giải thích căn cứ tính toán hoặc hướng dẫn bước kế tiếp cho khách. **Sửa rubric:** Yêu cầu câu trả lời hoàn tiền phải nêu rõ trạng thái xử lý và căn cứ áp dụng chính sách. |
| 4 | `HL-028` (helpful) | `PASS` | `FAIL` | **Người đúng** | Lặp lại mẫu xin lỗi lịch sự nhưng bỏ trống thông tin cần thiết để khách bổ sung hồ sơ/mã đơn. **Sửa rubric:** Phân tách rõ tiêu chí *lịch sự* (`tone_ok`) khỏi tiêu chí *hữu ích* (`helpful`); câu trả lời lịch sự nhưng vô ích vẫn phải chấm `FAIL`. |
| 5 | `HL-035` (helpful) | `PASS` | `FAIL` | **Người đúng** | Agent chỉ mong khách thông cảm mà không nêu thời hạn phản hồi hoặc kênh hỗ trợ tiếp theo. **Sửa rubric:** Bổ sung ví dụ few-shot âm tính cho các mẫu câu từ chối/xin lỗi thiếu thông tin hướng dẫn tiếp theo. |

## Kết luận
- **Tiêu chí bị loại và lý do:** Loại bỏ `helpful` khỏi các metric dùng để đánh giá/chặn gate. Trên ma trận nhầm lẫn (`TP=9, FP=0, FN=21, TN=70`), judge chỉ phát hiện được 9/30 ca lỗi thật (`Recall = 0.300`, bỏ sót 21 ca `FN`) và đạt $\kappa = 0.375$, làm sai lệch tỷ lệ lỗi quan sát ($0.09$) so với tỷ lệ lỗi thực tế sau hiệu chỉnh Rogan–Gladen ($0.30$).
- **Rủi ro còn lại:** Judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới để có held-out thật.