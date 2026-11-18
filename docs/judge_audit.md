# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (κ=0.757≥0.6, P=0.857≥0.8, R=0.800≥0.7) | Dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (R=0.300 < 0.7, κ=0.375 < 0.6) | Loại |

## ≥5 case audit thủ công (trích từ `judge_tone_public.csv`)
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | `H-008`: "Em đã hoàn tiền cho anh rồi nha, anh check giúp em." | PASS | FAIL | Người đúng | Ngôn từ xưng hô "Em - anh", dùng từ "nha", "check" quá suồng sã, thiếu chuẩn mực CSKH. Judge nhầm suồng sã là thân thiện. **Sửa rubric:** cấm đại từ xưng hô thân mật không chính thức, bắt buộc dùng ngôi chuẩn "LogiX - Quý khách/Anh/Chị". |
| 2 | `H-017`: "Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha." | PASS | FAIL | Người đúng | Xưng hô "bạn ơi", "nha" suồng sã, thiếu tôn trọng khách hàng. **Sửa rubric:** cấm dùng các trợ từ cảm thán xuề xòa ("bạn ơi", "nha") trong phản hồi chính thức. |
| 3 | `H-029`: "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm." | PASS | FAIL | Người đúng | Giọng điệu mệnh lệnh ("không cần nhắn thêm") gây ức chế và xua đuổi khách. Judge bị đánh lừa bởi từ "Dạ, vui lòng". **Sửa rubric:** phạt nặng câu mang tính áp đặt/ra lệnh, thẩm định toàn văn thay vì chỉ bắt từ khóa lịch sự mở đầu. |
| 4 | `H-042`: "Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ." | FAIL | PASS | Người đúng | Phản hồi ngắn gọn, lịch sự, đủ thông tin giải quyết và mốc thời gian. Judge phạt vì "thiếu xin lỗi". **Sửa rubric:** quy định rõ với case hoàn tiền thành công thì lời xin lỗi là tùy chọn, không phạt câu trả lời ngắn gọn và chuẩn xác. |
| 5 | `H-051`: "Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ." | FAIL | PASS | Người đúng | Dùng thuật ngữ "refund" là thuật ngữ TMĐT thông dụng, được quy định cho phép trong CS LogiX. Judge phạt máy móc vì có từ tiếng Anh. **Sửa rubric:** bổ sung danh mục thuật ngữ nghiệp vụ được chấp nhận (whitelist: refund, voucher, tracking...). |

## Kết luận
- **Tiêu chí bị loại:** `helpful`.
- **Lý do chi tiết (ma trận nhầm lẫn):**
  - Ma trận của `helpful`: `tp=9, fp=0, fn=21, tn=70`.
  - Recall cực thấp: chỉ đạt $0.300$ (ngưỡng tối thiểu là $0.7$), nghĩa là bỏ sót tới $21/30$ trường hợp không hữu ích ($70\%$ lỗi bị lọt).
  - Hệ số tương đồng Cohen's Kappa chỉ đạt $0.375$ (ngưỡng sàn là $0.6$).
  - Judge bị thiên lệch dễ dãi (lenient bias): hễ thấy câu trả lời xin lỗi lịch sự/dài dòng là chấm PASS dù thực chất câu trả lời không giải quyết được vấn đề hay cung cấp thông tin hữu ích cho khách hàng. Do đó, tuyệt đối **không được dùng `judge_helpful` làm release gate**.
- **Rủi ro còn lại:** Cả hai judge hiện chỉ được kiểm tra trên dữ liệu replay đã công khai (public replay benchmark); cần thu thập thêm tập nhãn người mới (held-out data) để đo lường độ tin cậy trong môi trường sản xuất thật.
