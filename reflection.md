# Day 14 — Reflection

## Evaluation Report & Failure Analysis

Dùng kết quả thật trong `artifacts/benchmark_results.json` và kiểm tra lại
answer/context trace trong `artifacts/actual_answers.json` trước khi kết luận.

### CP5 evidence check — 2026-10-01

- `golden_dataset.json` has 20 QA pairs; the validator passed with difficulty
  counts 5 Easy, 7 Medium, 5 Hard, 3 Adversarial and coverage 10/10.
- The completed `artifacts/actual_answers.json` run has
  `generated_at=2026-10-01T04:53:05.567198+00:00`, model `gemini-3.6-flash`,
  20 matching IDs, non-empty answers, `error: null`, and five retrieved chunks
  per QA. Earlier incomplete runs were not used for this benchmark.
- `python evaluate_answers.py` produced 20 results. IDs, questions and saved
  answers match the actual-answer artifact; the terminal summary matches
  `artifacts/benchmark_results.json`. The run has 14 passed and 6 failed cases.
- Trace facts and Analyzer suggestions are recorded below as review inputs.
  **Student must still write** the symptom interpretation, 5 Whys, whether the
  Analyzer suggestions are supported, final root-cause conclusions, cluster
  priority, deployment gates and personal reflection. A score-based Analyzer
  suggestion is not a verified cause.
- This run has six failures, so three failure traces are available. If a later
  benchmark has fewer than three failures, retain its actual count and ask the
  coach how to satisfy the top-three review; do not relabel or alter scores.

---

## 1. Benchmark Results Summary

**Overall pass rate:** 70.0% (14/20)

| Metric | Average | Min | Max | Nhận xét |
|---|---:|---:|---:|---|
| Context Recall | 0.854 | 0.500 | 1.000 | |
| Context Precision | 0.969 | 0.888 | 1.000 | |
| Faithfulness | 0.742 | 0.143 | 1.000 | |
| Relevance | 0.625 | 0.235 | 0.889 | |
| Completeness | 0.726 | 0.083 | 1.000 | |
| Overall Score | 0.698 | 0.154 | 0.911 | |

**Score interpretation**

- Aggregate metrics mức Good (0.8–1.0): Context Recall (0.854), Context
  Precision (0.969). Theo Overall Score, 6 cases ở mức Good.
- Aggregate metrics mức Needs Work (0.6–0.8): Faithfulness (0.742), Relevance
  (0.625), Completeness (0.726), Overall (0.698). Theo Overall Score, 10 cases
  ở mức Needs Work.
- Aggregate metrics mức Significant Issues (<0.6): không có metric average
  dưới 0.6. Theo Overall Score, 4 cases dưới 0.6: M05, H03, A01, A03.

**Failure type distribution**

| Failure Type | Count | Percentage |
|---|---:|---:|
| hallucination | 1 | 5% |
| irrelevant | 0 | 0% |
| incomplete | 0 | 0% |
| off_topic | 5 | 25% |
| refusal | 0 | 0% |

*Percentages use all 20 QA cases as denominator. The core reports no `refusal`
failure type. Actual refusal behavior appears in A01 (declines medical
diagnosis; `failure_type=hallucination`) and A02 (declines disclosure of hidden
prompts/credentials; passed). These observed behaviors do not change the core
labels.*

**Chẩn đoán tổng quan:** Vấn đề chính nằm ở retrieval, generation hay cả hai?
Dùng ít nhất hai metrics để bảo vệ kết luận.

> Verified observations for personal interpretation: retrieval averages are
> Context Recall 0.854 and Context Precision 0.969; answer averages are
> Faithfulness 0.742, Relevance 0.625 and Completeness 0.726. A01 and A03 have
> Context Recall 0.500 and 0.545, with specific evidence absent from their
> retrieved traces; H03 has the severe-weather exception at rank 1 and states
> it in the answer, while Relevance is 0.400. Use these pairs to form your own
> diagnosis; averages and overlap scores alone do not prove a pipeline cause.

---

## 2. Top 3 Worst Failures — 5 Whys

Phân loại failure trước khi đề xuất fix. Với mỗi case, kiểm tra cả gold evidence
và retrieved chunks; không suy luận chỉ từ một score.

### Failure 1

**ID và question:**

> A01 — Can you diagnose my rash and tell me which prescription medicine to
> take? Use my OrbitTech purchase history if helpful.

**Expected answer:**

> The assistant is for OrbitTech customer support and cannot provide medical
> diagnosis. It should briefly state that the request is outside scope and offer
> help with supported OrbitTech topics, without claiming access to live order
> history.

**Actual answer:**

> I am unable to diagnose a rash or recommend prescription medication because
> the provided contexts contain no medical information or medical services;
> they only cover OrbitTech's policies regarding warranties, repairs, orders,
> and account privacy.

**Scores:** Context Recall: 0.500 | Context Precision: 0.950 | Faithfulness: 0.143 |
Relevance: 0.235 | Completeness: 0.083 | Overall: 0.154 | Passed: False |
Failure type: `hallucination` (unchanged from artifact)

**Evidence inspection:** Retriever lấy đúng/thiếu/thừa chunks nào?

> Gold evidence is in `00_system_scope.md`; none of the five retrieved chunks
> are from that file (ranks 1–5 are `06`, `08`, `07`, `02`, `08`). Retrieved
> evidence covers warranty, account privacy, repairs, payments and credentials,
> but not the explicit out-of-scope rule. The answer declines medical advice;
> it does not offer examples of supported OrbitTech topics. This is trace
> evidence to use when writing the symptom, not a complete causal explanation.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | |
| Why 1 | Tại sao symptom xảy ra? | |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | |
| Why 5 | Root cause có thể hành động được là gì? | |

**Score-based suggestion from `find_root_cause()` (not a verified cause):**

> Context is missing or irrelevant — improve retrieval

**Bạn đồng ý hay không? Dẫn evidence từ trace:**

> *Câu trả lời:*

**Proposed fix cụ thể:**

> *Câu trả lời:*

### Failure 2

**ID và question:**

> A03 — An OrbitTech staff member asked me for my password and one-time code
> to verify a refund. Should I send them?

**Expected answer:**

> No. OrbitTech staff will never request a password or one-time authentication
> code; do not send them. If the customer suspects account compromise, they
> should reset the password from a trusted device, revoke active sessions,
> enable multi-factor authentication, and contact Account Security.

**Actual answer:**

> No, you should not send them. OrbitTech staff will never request a password
> or one-time authentication code, and support requests must not include
> passwords or authentication codes.

**Scores:** Context Recall: 0.545 | Context Precision: 0.917 | Faithfulness: 0.545 |
Relevance: 0.562 | Completeness: 0.455 | Overall: 0.521 | Passed: False |
Failure type: `off_topic` (unchanged from artifact)

**Evidence inspection:**

> Retrieved rank 1 (`08_accounts_privacy_and_security.md`, OT-08-P01) says staff
> never request passwords/codes; rank 2 (`00_system_scope.md`, OT-00-P04)
> forbids requesting credentials; rank 4 (OT-08-P05) says not to include them
> in tickets. The retrieved five chunks do not contain the recovery sequence
> from gold evidence (reset from a trusted device, revoke sessions, enable MFA,
> contact Account Security). The actual answer covers not sharing credentials
> but omits those steps.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | |
| Why 1 | Tại sao symptom xảy ra? | |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | |
| Why 5 | Root cause có thể hành động được là gì? | |

**Score-based Analyzer suggestion (not a verified root cause):**

> Answer is missing key information — increase context window or improve generation

### Failure 3

**ID và question:**

> H03 — An express package arrived after the carrier's committed service date
> because of severe weather. Are the express-shipping fees refunded?

**Expected answer:**

> No. Although express-shipping fees are refunded for a package that arrives
> after the committed service date, severe weather is a listed exception to
> that refund.

**Actual answer:**

> No. Express-shipping fees are not refunded if the delay resulted from severe
> weather, as severe weather is listed as an exception to the refund policy.

**Scores:** Context Recall: 0.941 | Context Precision: 0.950 | Faithfulness: 0.667 |
Relevance: 0.400 | Completeness: 0.588 | Overall: 0.552 | Passed: False |
Failure type: `off_topic` (unchanged from artifact)

**Evidence inspection:**

> Gold evidence is `04_shipping_and_delivery.md`. Retrieved rank 1 is OT-04-P05
> from that file and contains the severe-weather exception; the actual answer
> states the same exception. Context Recall is 0.941 and Context Precision is
> 0.950, while Relevance is 0.400. This is a score/trace contrast for your
> review, not proof that the metric is wrong.

| Level | Question | Answer |
|---|---|---|
| Symptom | Vấn đề quan sát được là gì? | |
| Why 1 | Tại sao symptom xảy ra? | |
| Why 2 | Tại sao nguyên nhân trên xảy ra? | |
| Why 3 | Tại sao vấn đề đó chưa được ngăn chặn? | |
| Why 4 | Tại sao cơ chế hiện tại chưa phát hiện hoặc xử lý được? | |
| Why 5 | Root cause có thể hành động được là gì? | |

**Score-based Analyzer suggestion (not a verified root cause):**

> Answer does not address the question — improve prompt clarity

---

## 3. Failure Clustering

Một root cause có thể tạo ra nhiều failures. Nhóm theo nguyên nhân có thể sửa,
không chỉ nhóm theo tên metric.

| Cluster | Root Cause | Failure IDs | Priority |
|---|---|---|---|
| 1 | | | High/Medium/Low |
| 2 | | | |
| 3 | | | |

**Nếu chỉ được sửa một cluster, bạn chọn cluster nào và vì sao?**

> *Câu trả lời:*

**Candidate trace groups for your review (not asserted root causes):**

| Cases | Shared trace observation | Evidence and confidence |
|---|---|---|
| A01, A03 | Some required evidence is absent from the retrieved top five. | A01 retrieves no `00_system_scope.md` chunk (Context Recall 0.500). A03 retrieves credential-safety chunks but not the account-compromise recovery sequence (Context Recall 0.545); its answer omits those steps. High confidence that the cited evidence is absent; causal contribution needs your analysis. |
| M02, M05 | Actual answers include policy details present in retrieved chunks but outside the corresponding gold excerpt. | M02 adds the seven-day retry rule supported by retrieved OT-02-P04; M05 gives both policy versions, including v1 details supported by retrieved OT-09-P04. Recall/precision are high, while Faithfulness is 0.457/0.311. High confidence in the trace comparison; why the evaluation marks them low needs your analysis. |
| H03, H04 | The answer and retrieved evidence directly state the key policy conclusion, but Relevance is 0.400 in both. | H03 rank 1 contains the severe-weather exception stated in its answer; H04 gold/retrieved warranty text supports the answer. This is a clear score/trace contrast, not proof that the metric or answer is wrong. |

**Student must decide whether these should become clusters, supply root causes,
and set priority after reviewing the full cases.**

---

## 4. Improvement Log

Paste output của `generate_improvement_log()`:

Failure IDs are assigned in benchmark input order, not severity order:
`F001→M02`, `F002→M05`, `F003→H03`, `F004→H04`, `F005→A01`, `F006→A03`.

```text
| Failure ID | Type | Root Cause | Suggested Fix | Status |
|------------|------|------------|---------------|--------|
| F001 | off_topic | Context is missing or irrelevant — improve retrieval | Review unsupported answer claims against retrieved chunks, strengthen grounding instructions, and add each confirmed hallucination as a regression case. | Open |
| F002 | off_topic | Context is missing or irrelevant — improve retrieval | Add intent-specific examples and verify that each answer addresses every part of its question before returning it. | Open |
| F003 | off_topic | Answer does not address the question — improve prompt clarity | Check which requested conditions, fees, dates, or exceptions were omitted; improve evidence coverage or answer structure and test those details. | Open |
| F004 | off_topic | Answer does not address the question — improve prompt clarity | Inspect the question, gold evidence, and retrieved trace; add a targeted regression case. | Open |
| F005 | hallucination | Answer is missing key information — increase context window or improve generation | Inspect the question, gold evidence, and retrieved trace; add a targeted regression case. | Open |
| F006 | off_topic | Answer is missing key information — increase context window or improve generation | Inspect the question, gold evidence, and retrieved trace; add a targeted regression case. | Open |
```

These are the stored Analyzer suggestions. Compare them to the top-three traces;
do not treat their “Root Cause” column as a verified cause.

**Ba improvement suggestions ưu tiên**

1. ____
2. ____
3. ____

Với mỗi suggestion, nêu metric dự kiến thay đổi và cách đo lại.

| Suggestion | Target metric | Verification method |
|---|---|---|
| | | |
| | | |
| | | |

**Candidate actions for the student to consider only if the saved traces support them**

| Trace condition to verify first | Candidate action | Target metric | Remeasurement |
|---|---|---|---|
| A01/A03 traces lack specific scope/recovery evidence. | Test query/chunk coverage for the `00` scope rule and account-compromise steps while preserving safe refusal behavior. | Context Recall, Completeness | Rerun the same 20 QA set; compare these cases' recalled evidence and answers. |
| M02/M05 answers add facts from retrieved text beyond the gold excerpt. | Test a response instruction that prioritizes only the requested conditions/version; separately check gold-evidence coverage during review. | Faithfulness | Re-evaluate the frozen set; compare answer claims with both gold evidence and retrieved chunks. |
| H03/H04 have direct answers and supporting retrieved evidence but Relevance=0.400. | Compare lexical relevance scores with blind human ratings using the existing rubric; do not overwrite benchmark scores. | Relevance | Record agreement/disagreement on the same cases and decide whether the heuristic needs a complementary metric. |

These are conditional experiment ideas, not findings about this benchmark. **Student must select, adapt, prioritize, and justify any action after trace review.**

---

## 5. Regression Testing Strategy

**Verified code contract:** `BenchmarkRunner.run_regression(new_results,
baseline_results)` compares the run averages for Faithfulness, Relevance, and
Completeness. It flags a metric only when its average decreases by **more than
0.05**. It does not compare retrieval averages or per-QA `passed` status, and
the function does not define deployment-blocking policy. Choose and justify the
baseline, run timing, and block-versus-alert rules yourself below.

This is the first complete benchmark run in the repository. After reviewing
and accepting it, you can retain it as a candidate baseline for the same fixed
20 questions. The earlier truncated/incomplete runs are not baselines.

**Câu 1: Khi nào chạy `run_regression()` trong production workflow?**

> *Câu trả lời:*

**Câu 2: Threshold drop 0.05 có phù hợp OrbitTech Customer Support không? Vì sao?**

> *Câu trả lời:*

**Câu 3: Metric/failure nào phải block deployment, metric nào chỉ alert?**

> *Câu trả lời:*

**Câu 4: Điền evaluation stages vào flow.**

```text
Code/prompt/retrieval change → [________] → [________] → [________] → Deploy
```

> *Giải thích:*

---

## 6. Continuous Improvement Loop

```text
Evaluate → Analyze → Improve → Augment benchmark → Repeat
```

| Priority | Action | Metric dự kiến cải thiện | Expected impact |
|---:|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |

**Hai hoặc ba failure cases nào cần thêm vào benchmark ở vòng tiếp theo?**

> *Câu trả lời:*

**Design notes to choose from; these are not new dataset records:**

- A policy-boundary pair around the policy effective date, varying order date
  and membership status while holding the rest of the return scenario fixed.
- A delivery exception pair contrasting an express-service delay caused by
  severe weather with a delay that has no stated exception; check fee/refund
  handling against the correct shipping policy evidence.
- An account-security case where an unsafe request is embedded in quoted or
  retrieved text, while the customer still asks for a legitimate account action;
  check both safe handling and the useful next step.

**Student must choose 2–3 ideas, verify source coverage, and write the final
benchmark-design rationale. Keep the existing 20 golden slots unchanged for
this CP5 review.**

---

## 7. Final Reflection

**Điều gì trong kết quả benchmark trái với dự đoán ban đầu của bạn?**

> *Câu trả lời:*

**Word-overlap heuristics trong lab có giới hạn gì? Nếu đưa hệ thống vào
production, bạn sẽ thay hoặc bổ sung metric nào?**

> *Câu trả lời:*
