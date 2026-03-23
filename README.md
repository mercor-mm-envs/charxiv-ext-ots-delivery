# CharXiv Extension — OTS Delivery

Evaluation data and results for the CharXiv Extension benchmark — 100 validated chart-understanding figures from arXiv papers (post-2024), evaluated on Claude Opus 4.6, GPT-5.4, and Gemini 3.1 Pro Preview.

## Structure

```
data/
  descriptive_val.json           — 92 figures × descriptive Q&A (368 question-answer pairs, Q1–Q19)
  reasoning_val.json             — 129 reasoning Q&A pairs across 100 figures

images/                          — 100 chart images (PNG), figure IDs 3000–5005

qc/
  qc_results.json                — per-figure image QC + QA QC scores (89 figures scored)

results/                         — Eval results, output from charxiv-ext-eval-harness (Inspect AI)
  claude-opus-4-6/
    accuracy_summary.json        — overall + per-mode + per-qid + per-category + per-figure accuracy
    trajectories/                — per-sample JSON artifacts (497 files)
  gpt-5.4/
    accuracy_summary.json
    trajectories/                — 497 files
  gemini-3.1-pro-preview/
    accuracy_summary.json
    trajectories/                — 497 files
```

The `results/` folder is produced by running the [charxiv-ext-eval-harness](https://github.com/mercor-mm-envs/charxiv-ext-eval-harness) — an Inspect AI harness using GPT-4o as grader (same binary grading pipeline as the original CharXiv benchmark). Each `accuracy_summary.json` contains overall, per-mode, per-question-type, per-reasoning-category, and per-figure accuracy. Each `trajectories/` folder contains one JSON file per question per figure with the full model response, predicted answer, ground truth, and correctness flag.

## Eval Results Summary (100 figures)

| Mode | Claude Opus 4.6 | GPT-5.4 | Gemini 3.1 Pro Preview |
|------|----------------|---------|------------------------|
| Descriptive | 70.7% | 69.3% | 69.8% |
| Reasoning | 58.1% | 43.4% | 42.6% |
| **Overall** | **67.4%** | **62.6%** | **62.8%** |

## Benchmark Difficulty: Extension vs Original CharXiv

The original CharXiv benchmark (Princeton NLP, NeurIPS 2024) tests chart understanding across 2,323 natural arXiv figures with 19 descriptive templates and expert-crafted reasoning questions. Frontier models have begun to saturate its descriptive tasks — Claude Opus 4.6 achieves **90.5% on descriptive** with four of five categories exceeding 88%.

Our extension uses 100 independently sourced post-2024 arXiv figures with the same Q&A format and grading pipeline, rebalancing toward high-signal axes with the largest descriptive-to-reasoning gaps.

| Mode | Claude Opus 4.6 (Original CharXiv) | Claude Opus 4.6 | GPT-5.4 | Gemini 3.1 Pro |
|------|-------------------------------------|-----------------|---------|----------------|
| Descriptive | 90.5% | **70.7%** | **69.3%** | **69.8%** |
| Reasoning | 64.3% | **58.1%** | **43.4%** | **42.6%** |
| **Overall** | — | **67.4%** | **62.6%** | **62.8%** |


### Descriptive: Saturated Questions on Original CharXiv

Five question types exceed 95% for Claude Opus 4.6 on the original benchmark — these are downweighted in our extension:

| QID | Question | Claude (Original) |
|-----|----------|------------------|
| Q18 | What is the layout of the subplots? | 97.98% |
| Q19 | What is the number of subplots? | 96.92% |
| Q8  | Diff between consecutive x-axis ticks? | 96.88% |
| Q14 | Diff between max/min colorbar values? | 96.81% |
| Q15 | Maximum colorbar tick label? | 96.81% |

High-value questions retained and expanded (< 83% on original):

| QID | Question | Claude (Original) |
|-----|----------|------------------|
| Q13 | Names of labels in the legend? | 79.91% |
| Q16 | General trend of data left to right? | 80.56% |
| Q3  | Label of the y-axis? | 81.97% |
| Q17 | Total labeled ticks across all axes? | 82.14% |

## Question Types

### Descriptive (Q1–Q19)

Each figure has a subset of the 19 question types applied based on chart structure, weighted toward high-value questions (Q13, Q16, Q3, Q17).

| QID | Question |
|-----|----------|
| Q1  | What is the title of the plot? |
| Q2  | What is the label of the x-axis? |
| Q3  | What is the label of the y-axis? |
| Q4  | What is the leftmost labeled tick on the x-axis? |
| Q5  | What is the rightmost labeled tick on the x-axis? |
| Q6  | What is the spatially lowest labeled tick on the y-axis? |
| Q7  | What is the spatially highest labeled tick on the y-axis? |
| Q8  | What is the difference between consecutive numerical tick values on the x-axis? |
| Q9  | What is the difference between consecutive numerical tick values on the y-axis? |
| Q10 | How many lines are there? |
| Q11 | Do any lines intersect? |
| Q12 | How many discrete labels are there in the legend? |
| Q13 | What are the names of the labels in the legend? (from top to bottom, then left to right) |
| Q14 | What is the difference between the maximum and minimum values of the tick labels on the continuous legend (i.e., colorbar)? |
| Q15 | What is the maximum value of the tick labels on the continuous legend (i.e., colorbar)? |
| Q16 | What is the general trend of data from left to right? |
| Q17 | What is the total number of explicitly labeled ticks across all axes? |
| Q18 | What is the layout of the subplots? |
| Q19 | What is the number of subplots? |

### Reasoning (4 Instruction Categories)

Reasoning questions require synthesizing information across complex visual elements. Number-in-General is deliberately oversampled (highest headroom at 52.4% on original CharXiv). Each figure has 1–2 reasoning questions.

| Category | Code | Description |
|----------|------|-------------|
| Text-in-General | TiG (cat 1) | Answer grounded to text explicitly present in the chart |
| Text-in-Chart | TiC (cat 2) | Answer from predefined chart options or semantic meaning |
| Number-in-Chart | NiC (cat 3) | Number explicitly written in the chart |
| Number-in-General | NG (cat 4) | General numeric reasoning requiring multi-step computation |

## Trajectory Format

Each file in `trajectories/` follows this schema:

```json
{
  "figure_id": 3000,
  "mode": "reasoning",
  "qid": null,
  "inst_category": 4,
  "inst_category_name": "NG",
  "question": "In the 'Response of Y to Y Innovation' panel, what is the approximate difference between the peak y-value and the value at period 25?",
  "ground_truth": "0.05",
  "model_answer": "0.05",
  "is_correct": true,
  "score": 1,
  "valid": true,
  "response": "Based on the graph...\n\n1. Identify the peak y-value: ...\n2. Identify the value at period 25: ...\n\n0.05",
  "grader_explanation": "ground_truth='0.05' | extracted='0.05' | score=1",
  "run": "claude-opus-4-6"
}
```

Fields:
- `score`: `1` = correct, `0` = incorrect, `-1` = grader parse failure (excluded from accuracy)
- `valid`: `false` if grader failed to parse a score (excluded from accuracy denominator)
- `response`: full untruncated model output
- `grader_explanation`: GPT-4o grader's extracted answer and scoring rationale

## Eval Harness

Results produced by [charxiv-ext-eval-harness](https://github.com/mercor-mm-envs/charxiv-ext-eval-harness) — an Inspect AI harness supporting both descriptive and reasoning evaluation modes across Claude, GPT, and Gemini. Uses the same GPT-4o grading pipeline as the original CharXiv benchmark.
