# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | A few low scores on ambiguous or out-of-scope questions may be acceptable when the assistant clearly abstains and makes no unsupported claim; inspect the examples because overlap heuristics can mis-score concise answers. | Low scores on product, payment, warranty, privacy, or safety answers may indicate invented or unsupported claims. | Inspect answer claims against retrieved evidence; block unsupported policy or safety claims and improve grounding/prompting. |
| Answer Relevance | A low score on an intentionally broad or ambiguous question may reflect a necessary clarification request. | Repeated low scores on clear requests, such as order status or refund eligibility, mean the assistant is answering the wrong intent. | Review question-answer pairs; improve intent handling and add representative regression cases. |
| Context Recall | A lower score may be acceptable when the question needs only one fact and omitted gold context is redundant or unrelated to that fact. | Low recall when the answer depends on an exception, a date, or evidence from multiple documents risks an incomplete or incorrect answer. | Inspect missing evidence in traces; improve retrieval/query coverage and add multi-document cases. |
| Context Precision | Some irrelevant retrieved chunks may be tolerable if the needed evidence is ranked near the top and does not distract generation. | Low precision with relevant evidence buried below noisy chunks can mislead generation and reduce faithfulness. | Inspect rank order and noise; tune retrieval/reranking and monitor precision alongside recall. |
| Completeness | A lower score may be acceptable when the user asks for only one part of a topic or the safe response is to request clarification. | Low scores on multi-part questions or answers omitting material conditions, fees, deadlines, or exceptions can mislead customers. | Compare each requested element with the answer and gold evidence; improve answer coverage and test exceptions. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Dùng cùng một bộ cặp câu trả lời A/B có chất lượng đã được người chấm xác nhận. Ở condition 1, đưa A trước B; ở condition 2, đảo thành B trước A, giữ nguyên prompt, rubric và nội dung. Lặp lại nhiều cặp với thứ tự ngẫu nhiên và chấm mù. So sánh điểm và tỷ lệ thắng của từng câu trả lời giữa hai condition; nếu câu trả lời đứng đầu thường được ưu tiên dù chất lượng không đổi, đó là dấu hiệu position bias. Có thể thêm condition lặp lại cùng một answer ở cả hai vị trí để đo mức độ nhất quán.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Chấm theo tiêu chí quan sát được như tính đúng, đủ ý, bằng chứng và an toàn; không thưởng riêng cho độ dài hay văn phong hoa mỹ. Nêu rõ câu trả lời ngắn vẫn đạt điểm tối đa nếu bao phủ đủ yêu cầu, còn phần dài nhưng lặp ý hoặc không có bằng chứng không được cộng điểm và có thể bị trừ nếu gây hiểu nhầm. Dùng ví dụ chuẩn ở từng mức điểm để hai judge áp dụng giống nhau.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> So sánh điểm của judge với nhãn do người chấm có hướng dẫn độc lập tạo ra trên một tập đại diện, gồm cả câu trả lời tốt, kém và edge cases. Calibration cho biết judge có chấm lệch hệ thống, nhầm tiêu chí hoặc thiên vị độ dài/vị trí hay không; từ đó có thể chỉnh rubric và ngưỡng. Giữ một tập kiểm tra riêng để theo dõi chất lượng sau mỗi lần đổi model hoặc prompt.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.90 | Ưu tiên chặn câu trả lời thiếu grounding vì thông tin sai về giá, đơn hàng, bảo hành hoặc quyền riêng tư gây hại trực tiếp. |
| Answer Relevance | 0.70 | Chặn các bản thay đổi khiến assistant thường xuyên không trả lời đúng ý định người dùng. |
| Completeness | 0.80 | Giảm nguy cơ bỏ sót điều kiện, phí, mốc thời gian hoặc ngoại lệ quan trọng. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Chạy offline evaluation trên golden dataset cố định cho mỗi pull request và thay đổi prompt/model/retriever; so metric theo từng nhóm câu hỏi và giữ các case adversarial trong quality gate. Dùng ngưỡng ở trên cho trung bình từng metric, đồng thời chặn nếu có claim nghiêm trọng không được evidence hỗ trợ, kể cả khi trung bình vẫn đạt. Online evaluation dùng sau triển khai canary/shadow để theo dõi traffic thực, drift, tỷ lệ escalation và feedback mà không đưa dữ liệu nhạy cảm vào log. Human review dùng cho case rủi ro cao, câu trả lời bị khiếu nại, bất đồng giữa judge và metric, và một mẫu ngẫu nhiên định kỳ để hiệu chỉnh judge. Không dùng một ngưỡng `overall_score` mới thay cho công thức đã định nghĩa; theo dõi từng metric riêng.

---

## Part 2 — Core Coding (9:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
