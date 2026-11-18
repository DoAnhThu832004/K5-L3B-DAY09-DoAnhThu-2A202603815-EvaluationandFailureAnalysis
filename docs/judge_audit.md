# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (κ=0.757≥0.6, P=0.857≥0.8, R=0.800≥0.7) | Dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (R=0.300 < 0.7, κ=0.375 < 0.6) | Loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | `H-008`: "Em đã hoàn tiền cho anh rồi nha, anh check giúp em." | PASS | FAIL | Người đúng | Cách xưng hô "Em - anh" với từ "nha", "check" nói chuyện như bạn bè, không đúng chuẩn CSKH. Judge tưởng vậy là thân thiện nên chấm pass. Sửa rubric: cấm dùng đại từ xưng hô thân mật tự do, phải dùng ngôi xưng chuẩn (LogiX - Quý khách/Anh/Chị). |
| 2 | `H-017`: "Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha." | PASS | FAIL | Người đúng | Xưng hô "bạn ơi", "nha" nghe suồng sã, thiếu chuyên nghiệp. Sửa rubric: cấm các từ đệm suồng sã kiểu này trong câu trả lời chính thức. |
| 3 | `H-029`: "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm." | PASS | FAIL | Người đúng | Câu "không cần nhắn thêm" mang tính đuổi khéo và ra lệnh cho khách, làm khách khó chịu. Judge thấy có "Dạ, vui lòng" ở đầu là cho pass. Sửa rubric: phạt nặng câu có ý ra lệnh/đuổi khách, chấm theo ngữ cảnh cả câu chứ không chỉ bắt từ khóa lịch sự ở đầu. |
| 4 | `H-042`: "Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ." | FAIL | PASS | Người đúng | Trả lời ngắn gọn, đủ thông tin số tiền và thời gian nhận tiền. Judge bắt bẻ vì không thấy có câu xin lỗi. Sửa rubric: khi đã hoàn tiền thành công thì không bắt buộc phải xin lỗi, trả lời rõ ràng đúng trọng tâm là được. |
| 5 | `H-051`: "Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ." | FAIL | PASS | Người đúng | Dùng từ "refund" rất quen thuộc trong mua sắm online. Judge máy móc thấy có tiếng Anh là đánh trượt. Sửa rubric: cho phép dùng một số từ chuyên ngành thông dụng như refund, voucher, tracking. |

## Kết luận
- Tiêu chí bị loại: helpful.
- Lý do:
  - Ma trận nhầm lẫn của helpful: tp=9, fp=0, fn=21, tn=70.
  - Recall quá thấp, chỉ được 0.300 (sàn yêu cầu là 0.7), tức là bỏ lọt tới 21/30 trường hợp không hữu ích (lọt 70% lỗi).
  - Điểm Cohen's Kappa chỉ đạt 0.375 (sàn là 0.6).
  - Judge bị bệnh "dễ dãi": cứ thấy reply dài dòng hoặc có xin lỗi là cho PASS, dù câu trả lời không giải quyết được gì cho khách. Vì vậy không thể dùng judge này để làm cổng release gate.
- Rủi ro còn lại: Cả 2 judge mới chỉ thử trên tập replay public, cần thêm tập nhãn người mới (held-out data) để test chuẩn hơn khi chạy production thật.
