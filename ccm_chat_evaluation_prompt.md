# Question-Answering Evaluation Prompt

This document contains the evaluation prompt used for assessing question-answering quality in technical code migration scenarios.

## Overview

The evaluation uses an LLM-as-Judge approach to measure semantic alignment between generated answers and ground truth answers. The judge assigns a discrete coverage score (0, 0.5, or 1) based on how comprehensively the generated answer captures the ground truth content.

---

## Evaluation Prompt Template
```
You are a code migration expert with deep knowledge of ABAP legacy system modernization, API deprecations, and enterprise software architecture. Your task is to evaluate the quality of model generated's answer by comparing it against a ground truth answer.

Given a QUESTION, a GROUND TRUTH ANSWER, and a GENERATED ANSWER, assess how well the GENERATED ANSWER covers the content in the GROUND TRUTH ANSWER.

Scoring Guidelines:
- SCORE 1: The GENERATED ANSWER fully covers the GROUND TRUTH ANSWER (all key information present)
- SCORE 0.5: The GENERATED ANSWER partially covers the GROUND TRUTH ANSWER (some key information present, but important details missing)
- SCORE 0: The GENERATED ANSWER does not cover the GROUND TRUTH ANSWER (key information absent or incorrect)

--
QUESTION:
{question}

--
GROUND TRUTH ANSWER:
{truth_ans}

--
GENERATED ANSWER:
{gen_ans}

--

Output your evaluation in JSON format with the keys "REASONING" and "SCORE":
{"REASONING": "<your reasoning in one sentence>", "SCORE": <0, 0.5, or 1>}
```

---

## Scoring Criteria

| Score | Coverage Level | Description |
|-------|---------------|-------------|
| **1.0** | **Full Coverage** | Generated answer contains all essential information from ground truth. Response completely addresses the question with all critical technical details, though exact wording may differ. |
| **0.5** | **Partial Coverage** | Generated answer captures some essential information but omits important details from ground truth. Response is partially correct but incomplete for fully answering the question. |
| **0.0** | **No Coverage** | Generated answer lacks essential information from ground truth, provides incorrect information, or is irrelevant to the question. |

### Evaluation Notes:
- Focus on **semantic content**, not exact wording or phrasing
- A response with additional correct information beyond ground truth should still receive full score if it covers all ground truth content
- Partial credit (0.5) should be reserved for cases where meaningful information is present but key details are missing
- Incorrect information that contradicts ground truth should receive no credit (0)

---

## Example Evaluations

### Example 1: Full Coverage (Score = 1.0)

**QUESTION:**
```
How do I handle custom code that references VBBS after the system conversion?
```

**GROUND TRUTH ANSWER:**
```
If the VBBS table is used in customer code, you should create a view on VBBE table instead, as VBBS is deprecated in the new system.
```

**GENERATED ANSWER:**
```
After system conversion, VBBS table is no longer available. You need to modify your custom code to use VBBE table. The recommended approach is to create a CDS view or database view on VBBE that mimics the VBBS structure, then update your code references to use this new view.
```

**EVALUATION OUTPUT:**
```json
{
  "REASONING": "Generated answer fully addresses the deprecation of VBBS and correctly recommends creating a view on VBBE, with additional implementation guidance that enhances completeness.",
  "SCORE": 1
}
```

**Analysis:** The generated answer covers all essential points (VBBS deprecated, use VBBE, create view) and provides additional helpful detail without contradicting the ground truth.

---

### Example 2: Partial Coverage (Score = 0.5)

**QUESTION:**
```
What changes are needed for BDC programs when migrating to the new system?
```

**GROUND TRUTH ANSWER:**
```
BDC programs require transaction code updates (e.g., MSC3 to MSC3N) and screen structure changes (e.g., SAPMM03S to SAPLMGMM with corresponding field updates from RM03S to RMMG1).
```

**GENERATED ANSWER:**
```
When migrating BDC programs, you need to update transaction codes. For example, MSC3 should be changed to MSC3N to ensure compatibility with the new system.
```

**EVALUATION OUTPUT:**
```json
{
  "REASONING": "Generated answer correctly addresses transaction code updates but completely omits the equally critical screen structure and field mapping changes required for compatibility.",
  "SCORE": 0.5
}
```

**Analysis:** The response captures one of two essential components (transaction codes) but misses the screen structure migration, which is equally important for functional correctness.

---

### Example 3: No Coverage (Score = 0.0)

**QUESTION:**
```
How should I handle the deprecated BSEG table in the new system?
```

**GROUND TRUTH ANSWER:**
```
BSEG is replaced by ACDOCA in the new system. You should migrate custom code to read from ACDOCA instead, adjusting for the new table structure and field mappings.
```

**GENERATED ANSWER:**
```
You can continue using BSEG table with appropriate authorization settings. Make sure to configure the system parameters correctly.
```

**EVALUATION OUTPUT:**
```json
{
  "REASONING": "Generated answer provides fundamentally incorrect information by suggesting BSEG can continue to be used, failing to mention it is replaced by ACDOCA.",
  "SCORE": 0
}
```

**Analysis:** The response contradicts the ground truth by suggesting continued BSEG usage when it should recommend ACDOCA migration, demonstrating no understanding of the deprecation.

---


### Key Findings:

The improvement stems from GraphRAG's ability to retrieve entity-grounded context through 1-hop graph traversal, enabling more complete responses to technical queries that require understanding of component relationships and dependencies.
