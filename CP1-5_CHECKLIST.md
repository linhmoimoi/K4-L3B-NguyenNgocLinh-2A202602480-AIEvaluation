# Checklist rà soát CP1–CP5

Checklist này được đối chiếu với file và artifact trong repository. Required tests và validator đã được chạy lại trong lượt CP5 ngày 2026-10-01; kết quả được ghi dưới đây.

## CP1 — Data Models

- [x] `QAPair`, `EvalResult` và `overall_score()` đã được triển khai trong `template.py`.
- [x] `solution/solution.py` đồng bộ với `template.py` (SHA256 trùng nhau).

## CP2 — Metrics & LLM Judge

- [x] Các evaluator cho answer metrics và retrieval metrics cùng `run_full_eval()` đã có trong code.
- [x] `LLMJudge.score_response()` và `detect_bias()` đã có trong code.
- [x] Không thấy TODO bắt buộc còn lại trong phần implementation; TODO reranker là bonus CP4/Exercise 3.5.

## CP3 — Runner & Failure Analyzer

- [x] `BenchmarkRunner` có các hàm chạy benchmark, tạo report, regression và lọc failures.
- [x] `FailureAnalyzer` có phân loại, phân tích nguyên nhân, gợi ý và improvement log.
- [x] Worksheet có phần giải thích regression và quality gate.
- [x] Chạy `pytest tests/ -v`: 41 passed, 1 skipped (reranking bonus), exit code 0.

## CP4 — Dataset & Benchmark

- [x] `golden_dataset.json` có 20 QA: 5 Easy, 7 Medium, 5 Hard, 3 Adversarial.
- [x] `exercises.md` đã có Exercise 3.1, bảng đủ 5 metrics của Exercise 3.2, aggregate report và 3 case điểm thấp nhất.
- [x] `artifacts/actual_answers.json` và `artifacts/benchmark_results.json` hiện diện; benchmark artifact có 20 kết quả.
- [x] Chạy `python validate_golden_dataset.py`: PASS; 20 QA, counts 5/7/5/3 và document coverage 10/10.
- [ ] Exercise 3.4 (so sánh framework) và 3.5 (reranking) là bonus, chỉ cần nếu chọn làm.

## CP5 — Reflection & Finalize

- [x] `solution/solution.py` đã đồng bộ với `template.py`.
- [x] Hoàn thành 5 Whys cho ba cases A01, A03, H03; tách trace facts, inference và hypothesis.
- [x] Điền ba failure clusters, QA IDs, mức ưu tiên và lý do chọn cluster ưu tiên.
- [x] Chọn ba improvement actions, target metrics và cách đo lại; giữ Analyzer hints là gợi ý chưa được xác minh.
- [x] Hoàn thành regression policy về thời điểm chạy, baseline, ngưỡng 0.05, gate riêng và xử lý trước deploy.
- [x] Điền pre-deploy flow, continuous-improvement priorities và ba thiết kế case cho vòng benchmark sau; giữ nguyên 20 QA hiện tại.
- [x] Viết reflection cá nhân về kết quả và giới hạn của word-overlap metrics.
- [x] Rà trạng thái Git và ignore rule cho `.env`; lượt này không stage/commit. Không đọc giá trị trong `.env`.
- [ ] Nộp link repository lên Codelab và xác nhận quyền truy cập theo yêu cầu lớp học (bước ngoài repository, chưa thể xác nhận từ workspace).

## Tóm tắt

**Việc còn ngoài workspace:** nộp link repository lên Codelab và xác nhận quyền truy cập theo yêu cầu lớp học. CP3.4/3.5 là bonus; không làm trong lượt này.
