# Expert rubric review: what changes when a domain expert reweights the rubric?

The rubrics in this project were written by the author with AI assistance and revised after reading model answers. As a first step toward independent validation, one external expert, a licensed lawyer with five years in commercial and corporate practice, advising startups, corporates and public-sector entities, reviewed the full rubrics for two private-corpus tasks: the **mutual NDA** (26 criteria) and the **probation termination memo** (15 criteria). The expert assessed each criterion, its weight, its scoring method and whether failing it should reduce the whole answer.

The expert did **not** score any candidate answers. This is a review of the answer key, not yet a measurement of agreement between the reward and an expert.

## What the expert changed

- **Kept all 41 criteria.** No criterion was removed as legally wrong, and none was added.
- **Changed weights on seven criteria** and **added eight whole-answer penalties**:
  - NDA: five penalties (×0.8–0.9) on the definition, representatives, compelled disclosure, bracketed terms and Saudi forum options.
  - Probation: three penalties (×0.7–0.8) on mutual termination, no compensation or end-of-service award, and the seven-day settlement of dues.
- **Rebalanced emphasis on probation:** compensation (2 → 4), settlement of dues (1 → 3) and repeat probation (1 → 2).

The full list of changes is in [`results.json`](results.json).

In this project's reward, a penalty applies whenever a criterion scores below full credit, including partial credit, and penalties multiply together.

## Result

The same saved answers were rescored with the corrected trap checker. Judge scores came from cache; nothing was regenerated.

| Answer | NDA before | NDA expert | Probation before | Probation expert |
|---|---:|---:|---:|---:|
| Reference | 0.87 | 0.87 | 0.91 | 0.92 |
| Claude Opus 5.5 | 0.76 | 0.62 | 0.38 | 0.30 |
| GPT-6 Astra | 0.66 | 0.42 | 1.00 | 1.00 |
| Copy of the prompt | 0.16 | **0.07** | 0.36 | **0.14** |
| Deliberately flawed answer | 0.01 | 0.005 | 0.12 | 0.05 |

**1. Rewards fell for prompt-copy and deliberately flawed controls.** The probation prompt-copy reward fell from 0.36 to 0.14, and the deliberately flawed control fell from 0.12 to 0.05. NDA control rewards also fell, while reference scores remained similar. These observations do not by themselves establish better legal grading.

**2. The model order doesn't change.** Astra is still at 1.00 on probation, while Claude is still sunk by the outdated-law trap. On the NDA, both models fall, Astra more.

**3. The new penalties turn pattern misses into whole-answer cuts.** Most of each model's drop comes from the penalties, not the weights. Checking the failures by hand found two cases where the expert's intent and the code's pattern disagree:

| Case | What the answer wrote | What the check expects | Effect |
|---|---|---|---|
| NDA, Astra, bracketed terms | Durations left open as «[مدة الاتفاقية]» and «[مدة استمرار الالتزامات]» | A time unit inside the bracket, e.g. «[سنة/سنتين]» | Half credit and a ×0.8 cut. Recognising «[مدة …]» would give 0.54 instead of 0.42 |
| Probation, Claude, settlement of dues | Lists «(88)» in a table of articles; never states the seven-day deadline | «المادة 88» or "seven days" near "dues" | Zero and a ×0.8 cut. The expert's method credits citing Art. 88 alone; substantively the deadline is missing |

Neither pattern was changed after seeing these results; both are recorded here as open calibration questions for the expert.

**4. The forum penalty amplifies the hidden NDA requirement.** Neither model named the Saudi commercial courts or Saudi arbitration, because the NDA prompt never says the parties are Saudi. Both left jurisdiction as a placeholder. The new ×0.8 penalty makes that existing prompt defect cost more, which strengthens the case for fixing the prompt rather than the rubric.

## Interpretation

The expert rubric revisions **lowered rewards for prompt-copy and deliberately flawed controls**, while reference scores remained similar. Lower scores alone do not establish better legal grading; agreement with independent expert answer judgments has not yet been measured. The revisions also **raised the reliability bar for the checks.** Every newly penalised criterion is a pattern-matched positive check, and none has had the matched-sentence test the traps received. Before these rubrics are used as a training reward, each of the eight penalised criteria needs that test, because a single regex miss now costs 10–30% of the answer's reward instead of 1–2 points.

## Limitations

- **One expert, two tasks.** The reviewer's qualifications are as reported by the author and are not independently verified here. Inter-expert agreement is unknown.
- **No expert scores of answers.** The review can't yet say whether the expert rubric agrees with the expert's own judgement of these answers.
- **Small and private.** The task packages and answers are withheld, so these numbers can't be reproduced from this repository. They summarise local reruns.
- **One saved answer per model per task**, from different generation harnesses. This is not a model ranking.

## Next step

Ask the expert to score the five answers per task, blinded to their source and before seeing any rubric scores. Then report how often each rubric version (historical, corrected and expert) ranks the answers the way the expert does.
