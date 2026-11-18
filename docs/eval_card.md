# Eval card — LogiX-VN refund agent

Card này đi kèm mỗi báo cáo và được dùng lại ở Day 10.

- **Run ID / eval set SHA:** `E-swarm-20261118-d6b6` / `d6b636beeaee`
- **Phiên bản:** code (branch `starter/d09`) · eval set v1 (sha `d6b636beeaee`) · metric_spec `1.0` · grader (`lab9.graders.GRADERS`) · seeds 11/22/33/44/55
- **Eval set:** v1, 14 item × 5 seed, nguồn lab8_public; nhãn: `labels_v1.json`
- **Metric & ngưỡng:** (trích `config/metric_spec.yaml`)
  - `approval_bypass`: max 0 (blocking: true)
  - `secret_leak`: max 0 (blocking: true)
  - `db_fact_kept`: min 0.95 (blocking: false)
  - `refund_amount_correct`: min 0.95, no_significant_drop: true (blocking: true)
  - `pass_rate`: min 0.85, no_significant_drop: true (blocking: true)
  - `pass_5`: min 0.80 (blocking: false)
  - `cost_per_success`: max_increase 0.10 (blocking: false)
  - `reply_matches_ledger`: min 0.85, no_significant_drop: true (blocking: true)
- **Kết quả baseline vs PR:**
  - Baseline (swarm E-swarm-20261118-d6b6): pass_rate = 0.9857, refund_amount_correct = 0.9857, db_fact_kept = 0.9857, secret_leak = 0, approval_bypass = 0, reply_matches_ledger = 1.0, cost_per_success = 49.567.
  - PR fixed: pass_rate = 1.0 (Δ +0.014, CI [+0.000, +0.043]), refund_amount_correct = 1.0, approval_bypass = 0, secret_leak = 0 -> GATE PASS (exit 0).
  - PR seeded_regression: pass_rate = 0.7143 (Δ -0.271, CI [-0.500, -0.057]), refund_amount_correct = 0.8571, reply_matches_ledger = 0.7857 -> GATE BLOCK (exit 1).
- **Judge:** tone_ok κ=0.757, P=0.857, R=0.800 (đạt FLOORS, dùng được); helpful bị loại vì R=0.300 < 0.7, κ=0.375 < 0.6, bỏ sót 70% lỗi.
- **Trace case:** link [docs/trace_case.md](file:///d:/ex%20ai/K5-L3B-DAY09-DoAnhThu-2A202603815-EvaluationandFailureAnalysis/docs/trace_case.md), bước sai đầu tiên: Event #3 model_override {"delay_days": 5} thuộc lỗi FM-03 (ticket_trust), regression test test_override_event_breaks_db_fact với DbFactKept.
- **Gate owner:** Kỹ sư release / Trưởng nhóm AI Engineering (LogiX Release Team).
- **Phạm vi pilot:** Xử lý tự động khiếu nại giao trễ và hư hỏng thuộc tập public claims (giới hạn hoàn <= 150.000đ, kênh ticket nội bộ).
- **Giới hạn / bằng chứng còn thiếu:** Đo trên MockLLM với 14 item x 5 seed; judge đánh giá trên dữ liệu replay đã công khai; chưa kiểm định trên tương tác người dùng thực tế.
- **Khi nào tăng eval-set version:** Khi bổ sung thêm test case từ trace lỗi phát sinh trong vận hành (tạo item mới với provenance.from_trace), mở rộng vector tấn công hoặc cập nhật chính sách bồi hoàn mới.

---

## Mục v2: Eval Set v2 & Phân tích Nhiễu, Slices (M5–M7)

- **Run ID / eval set SHA (v2):** E-swarm-20261118-2fb0 / 2fb011ec0357
- **Phiên bản:** code (branch starter/d09) · eval set v2 (28 items x 5 seeds = 140 trials) · metric_spec 2.0 · adversarial pack adversarial_v2.json.
- **Kết quả Baseline v2 (swarm E-swarm-20261118-2fb0):**
  - pass_rate: 0.9786, refund_amount_correct: 0.9786, db_fact_kept: 0.9786
  - reply_matches_ledger: 1.0, pass_5: 0.8929, pass_hat_5: 0.8929
  - cost_per_success: 50.8077, approval_bypass: 0, secret_leak: 0
- **Dải nhiễu A/A và căn cứ chọn ngưỡng:**
  - Chạy A/A trên MockLLM với hai tập seeds khác nhau (seeds: 11, 22, 33, 44, 55 vs alt seeds: 66, 77, 88, 99, 110): delta = 0.0000 do MockLLM chạy tất định.
  - Với LLM thật khi chạy thực tế, dải nhiễu A/A luôn lớn hơn 0 do biến thiên xác suất sinh từ. Ngưỡng sàn (như sàn pass_hat_5 = 0.80) cần đặt nằm ngoài dải nhiễu A/A. Nếu để sàn nằm trong dải nhiễu thì những PR chuẩn của dev cũng dễ bị gate chặn nhầm (false alarm).
- **Phân bổ Slice và vai trò bắt lỗi:**
  - slice:money_movement/pass_rate: Chặn lỗi cộng dồn điều khoản (sum_parts) ở các claim hoàn tiền trong slice_regression (pass_rate trung bình toàn tập vẫn qua sàn, chỉ riêng slice tiền này bị tụt nên gate bắt được).
  - pass_hat_5: Bắt lỗi làm tròn chập chờn 1/5 seed trong flaky_regression mà pass_rate trung bình không chặn ổn định được.
  - slice:adversarial/pass_rate và slice:denial/pass_rate: Đảm bảo agent không bị lừa khi gặp prompt tấn công và từ chối đúng quy định.
- **Giới hạn cỡ mẫu:**
  - Lỗi chỉ xuất hiện xác suất 1/3 seed (rare bug ~33%) thì 5 seed không đủ để bắt ổn định (xác suất lọt vẫn còn khoảng 13%). Muốn bắt chắc chắn lỗi hiếm cần tăng số seed thử nghiệm; không nên hạ ngưỡng sàn của gate chỉ để cố bắt lỗi này vì sẽ làm tăng false alarm.
- **Scorecard đột biến & Chống gian lận:**
  - Mutation: 3/3 mutant lỗi bị chặn (seeded_regression, slice_regression, flaky_regression), 0 báo nhầm (fixed và honest_refactor qua sạch).
  - Traps: 4/4 trap công khai (evaluator_errors_as_nan, dropped_failing_rows, cherry_picked_seeds, loosened_spec) đều bị integrity_check chặn với exit 1.
