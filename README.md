# Arabic Legal Reward Reliability

### Better evidence and better rewards for Arabic legal AI in MENA

**A model can produce a convincing answer and still be wrong. An evaluator can also penalize an answer that is right.** This project investigates both sides of that problem through Arabic legal drafting: how to build tasks, evidence, and scoring rules that deserve to be used for evaluation—and eventually for training.

**Mohamed Zaghoo** · Applied physics → AI post-training → Arabic legal evaluation

[Verifier experiment](#main-result-measured-and-reduced-trap-errors) · [Expert rubric review](#second-result-an-expert-review-makes-the-rubric-stricter-and-raises-the-bar-for-the-checks) · [Historical model scores](#benchmark-results) · [Why this matters](#why-high-quality-data-matters-in-mena) · [Run the demo](#run-the-public-demo) · [Technical methods](docs/METHODS.md)

| Original exploratory study | Public, runnable release | Current research stage |
|---|---|---|
| 6 Arabic legal tasks · 109 weighted criteria | 1 synthetic task · 67 frozen matched sentences | Trap checks validated on held-out sentences; expert rubric review of 2 tasks; no expert answer grading or RL training yet |

## Main result: measured and reduced trap errors

On **30 held-out Arabic sentences**, false penalties fell from **7/17 to 0/17**, while missed violations fell from **3/13 to 1/13** after adding clause-scoped negation handling. The experiment contains 67 sentences: 37 development items and 30 held-out items. The reported procedure froze the dataset before changes and scored the held-out split once after the fix was frozen.

![Trap checker errors before and after](experiments/verifier_validation/trap_errors.svg)

The labels were written by an AI assistant, not independent legal reviewers, and the sample is small: 0/17 has a 95% upper bound of about 16%. [Method, per-phenomenon errors and limitations](experiments/verifier_validation/README.md).

## Second result: an expert review makes the rubric stricter, and raises the bar for the checks

One external expert, a licensed lawyer with five years in commercial and corporate practice, advising startups, corporates and public-sector entities, reviewed the full rubrics for two private-corpus tasks, the mutual NDA (26 criteria) and the probation memo (15 criteria):
- They kept every criterion.
- They changed the weights on eight.
- They added eight whole-answer penalties for missing core content, such as compelled-disclosure mechanics, or no compensation during probation.

The same saved answers were then rescored. Nothing was regenerated.

| Answer | NDA before → expert | Probation before → expert |
|---|---:|---:|
| Reference | 0.87 → 0.87 | 0.91 → 0.92 |
| Claude Opus 5.5 | 0.76 → 0.62 | 0.38 → 0.30 |
| GPT-6 Astra | 0.66 → 0.42 | 1.00 → 1.00 |
| Copy of the prompt | 0.16 → **0.07** | 0.36 → **0.14** |

What this shows:

- **The answer key got stricter in the right direction.** Copying the probation prompt used to earn nearly as much as a real memo. Under the expert rubric it earns 0.14, while the references are unchanged. The model order on both tasks is the same.
- **The new penalties make every pattern miss expensive.** Two failures traced to the pattern, not the answer:
  - Astra left the NDA durations open as «[مدة الاتفاقية]», but the check only recognises a time unit inside brackets, so it gave half credit and cut the whole answer by 20%.
  - Claude cited Art. 88 only as «(88)» in a table, which the expert's method would credit but the pattern doesn't.

  Under the old weights a miss cost 1–2 points. Now it costs 10–30% of the reward.
- **The hidden NDA requirement got more expensive too.** Neither model named Saudi forums, because the prompt never says the parties are Saudi. The expert's new forum penalty cuts both for it.

So expert input and verifier validation are complementary. The expert decides *what* matters and how much. Matched-sentence tests then have to show the code *detects* it reliably, because the expert's design makes detection errors cost more. The expert has not yet scored the answers themselves, so the agreement between the reward and an expert is not measured. [Changes, per-answer analysis and limitations](experiments/expert_rubric_review/README.md).

## Why high-quality data matters in MENA

An Arabic contract, legal memo, or demand letter carries more than language. Its meaning depends on the jurisdiction, relevant date, source documents, parties, and the action being requested. MENA is not a single legal system, and this Saudi-focused study does not establish coverage across the region or its languages.

The research thesis is straightforward: **useful domain data must tell a model what to do, give it the evidence needed to do it, and let an evaluator distinguish success from a plausible mistake.** More examples alone cannot resolve an unstated requirement or an unreliable answer key.

| What quality requires | In legal tasks |
|---|---|
| Explicit context | Jurisdiction, applicable date, contract terms, and the client's instructions |
| Traceable evidence | Source document, relevant provision, and citation verification status |
| Precise success conditions | Preserve rights, identify material issues, and obey drafting constraints |
| Meaning-sensitive grading | Distinguish a prohibition from its negation and contractual terms from statutory rules |
| Independent validation | Qualified review of legal reasoning and criterion-level labels |

The present pilot focuses on Arabic legal drafting and review. Its immediate priority is to validate whether the evaluator distinguishes substantive legal errors from correct variations in wording. A training reward is only as useful as the evidence and verifier behind it; whether better legal task data improves trained-model behavior remains a question for a future controlled experiment.

Arabic evaluation already has substantive foundations: [ArabicMMLU](https://aclanthology.org/2024.findings-acl.334/) draws on regional educational material, while [ArabLegalEval](https://aclanthology.org/2024.arabicnlp-1.20/) evaluates Arabic legal knowledge. This smaller project examines a specific issue within that broader effort: **the reliability of rewards for open-ended professional drafting.**

## Benchmark results

**Historical raw reward × 100—not accuracy, pass rate, or a model ranking.** These scores include known evaluator defects. There is one saved answer per model per task; the generation harnesses differ, and rubrics were revised after inspecting outputs.

| Task | GPT-6 Astra | Claude Opus 5.5 | Interpretation |
|---|---:|---:|---|
| Consulting contract review | 72.50 | 75.00 | Development-set rubric |
| Maintenance contract review | 77.38 | 85.71 | Development-set rubric |
| Partner exit preparation | 73.00 | 83.33 | Semantic structure checks miss paraphrases |
| Payment demand letter | 45.96 | 84.85 | Astra negation falsely triggers a penalty |
| Probation termination memo | 43.94 | 37.88 | Astra contractual example falsely triggers a penalty |
| Mutual NDA drafting | 29.47 | 76.36 | False IP-transfer penalty; jurisdiction missing from prompt |

Astra used Codex at high reasoning; Claude used the project's provider adapter. Full scores combine deterministic checks with a Claude Opus 5.5 judge. Names denote requested experiment settings, not independent confirmation of backend model identity. Judge dependence, incomplete references, and unverified source claims further limit comparisons.

The numbers are retained as observations about an experimental evaluator. No aggregate winner is reported. [Machine-readable summary](docs/benchmark_summary.json) · [Research note](docs/RESEARCH_NOTE.md)

### Supporting observations: private-corpus rerun

The reported rerun of the private six-task corpus changed three gate decisions previously identified as false positives. No other recorded decisions changed. These results are supporting observations: the source corpus and full model outputs are withheld, so this rerun cannot be independently reproduced from the public release.

| GPT-6 Astra answer | Raw reward, historical checker | After the fix |
|---|---:|---:|
| Payment demand letter | 0.46 | 0.72 |
| Probation termination memo | 0.44 | 1.00 |
| Mutual NDA drafting | 0.29 | 0.66 |

The probation rubric scores reverse after the fix: Astra receives 1.00 while Claude remains at 0.38. **A score of 1.00 means full credit under this rubric, not that the memo is legally correct in every respect.** This illustrates the sensitivity of model scores to grader errors; it does not establish overall model superiority or independent legal validation.

## Why it mattered: scoring errors can look like model failures

Three cases illustrate why data and verifier quality belong in the same research program:

1. **Negation changes the meaning, but not always the score.** A statement rejecting IP transfer triggered the NDA rule intended to prohibit IP transfer. The failed gate multiplied the entire answer score by 0.5.
2. **Context changes the rule being discussed.** The probation checker treated discussion of a contractual 90-day period as a claim about the statutory maximum, despite the response also stating a 180-day ceiling.
3. **The rubric can ask for information the prompt never supplied.** The NDA grader expected Saudi-specific grounding, but jurisdiction metadata was not included in the generation prompt. That calls for a new prompt version and new model runs.

The public demo reproduces the first mechanism with newly written text: a reminder saying it **does not** demand compensation receives the same score as one saying it does. It is deliberately small enough to inspect end to end. The demo rubric keeps the naive trap so the defect stays visible; a test shows that the same trap with `clause_negation` passes the negated reminder and still penalises the asserted one. Mechanisms 1 and 2 are addressed by the validation above. Mechanism 3 needs a new prompt version and is still open.

## How the evaluator works

```text
Task + candidate answer
        ↓
Arabic normalization → deterministic checks
        + optional rubric-based LLM judge
        ↓
Per-criterion scores and evidence
        ↓
Weighted score × failed-gate multipliers
```

`reward = (sum(weight × item_score) / sum(scored_weights)) × product(failed_gate_multipliers)`

Rules cover patterns, structure, citation ranges and known error signatures. The optional judge handles qualitative rubric items. A gate failure both loses criterion credit and reduces the whole answer's reward, amplifying false positives. Rules-only mode excludes judge criteria from the denominator.

This is a **hybrid evaluation and reward prototype**. An LLM judgment is not an objectively verifiable reward; a regex match is not a proof of legal correctness. The code supplies a single-step reward interface, but no model weights have been trained or updated.

## Run the public demo

Python 3.10+; no API key or external dependency needed:

```bash
python3 -m unittest discover tests
python3 -m experiments.verifier_validation.measure
python3 -m legal_rlvr.evaluate
python3 tools/check_release.py
```

The synthetic example earns **0.375**: a 0.75 base score multiplied by a 0.5 gate. Eight tests cover the demo (including reproduction of the known defect and its fix) and reproduce the verifier-validation numbers. Passing them does not establish semantic correctness. Evaluation writes and replaces `results/report.md` and `results/scores.json`.

```python
from legal_rlvr import LegalDraftingEnv

env = LegalDraftingEnv("synthetic_notice", use_judge=False)
prompt = env.reset()
reward, evidence = env.step("Your candidate answer")
```

The original six task packages, source-derived references, raw model outputs, and session logs are withheld. This checkout reproduces the synthetic failure demonstration, **not the historical six-task scores**. See [release scope](docs/RELEASE_SCOPE.md).

## Next experiments

- **Measure agreement with the expert:** have the expert score the saved answers blind to their source, then report how often each rubric version (historical, corrected, expert) ranks them the same way. Add a second expert to estimate how far experts agree with each other.
- **Validate the newly penalised criteria:** the eight criteria the expert turned into whole-answer penalties are pattern-matched and untested. Give each one a frozen matched-sentence test, like the traps, before using the expert rubric as a reward.
- **Finish validating the traps:** have a qualified Saudi lawyer review the 67 sentence labels and write a new held-out set from a different author.
- **Fix the hidden NDA requirement:** version the prompt with explicit jurisdiction and regenerate, rather than re-scoring old answers against a requirement they were never given.
- **Separate development from evaluation:** freeze prompts and rubrics before testing held-out task variants; record changes and keep historical results intact.
- **Improve model comparisons:** match prompts and tool access, collect multiple samples, and report uncertainty alongside independent human assessment.
- **Publish a complete legal case:** independently author a realistic Saudi legal task with dated official sources, an explicit rubric, and positive and negative controls that can be reviewed and reproduced publicly.

The aim is to develop **domain evaluation data with explicit assumptions, auditable evidence, and measured verifier reliability**. Those are prerequisites for investigating whether the same tasks can provide useful training signals.

[Technical methods and limitations](docs/METHODS.md) · [Historical research note](docs/RESEARCH_NOTE.md) · [Release scope](docs/RELEASE_SCOPE.md)

Development used AI assistance for code, examples, documentation and analysis. No employer or model-provider endorsement is implied.

No open-source license has been selected; public visibility alone does not grant reuse rights.
