# AI Working Standard

> **Default rule:** AI is an accelerator, critic and implementation partner. It is not the final authority.  
> The human owner retains responsibility for the question, logic, evidence, validation, interpretation and final sign-off.

This standard applies to research, SQL/Python/R workflows, academic writing, job applications, workplace analysis, reports and professional communication.

---

## 1. Core principle: keep the responsibility chain intact

A valid AI-assisted workflow must preserve the full chain:

**Question → Evidence → Logic → Implementation → Validation → Interpretation → Decision**

AI may assist at every stage, but no stage is accepted merely because an AI produced it.

The test is not “Did I write every line myself?” The test is:

- Can I explain why this step exists?
- Can I state what I expected before running it?
- Can I detect at least the main ways it could be wrong?
- Can I show evidence that the implementation behaves as intended?
- Can I distinguish what the data show from what I infer?
- Am I willing to take responsibility for the final claim or deliverable?

If the answer to one of these is “no”, that point becomes the next learning or validation task.

---

# 2. The default workflow

Every substantive task follows:

## **Define → Predict → Implement → Inspect → Challenge → Reproduce → Explain**

### 2.1 Define

Before implementation, state:

- What is the actual question?
- What is the target output or estimand?
- What are the inputs and their unit/grain?
- Which assumptions are being made?
- What constraints matter?
- What would count as success or failure?
- Which decisions are substantive human decisions rather than coding details?

For research, define the scientific question and estimand before choosing the model.

For data work, define what one row represents before joining, aggregating, reshaping or modelling.

For writing, define the intended claim before polishing the language.

### 2.2 Predict

Before seeing the result, write down what should happen if the logic is correct.

Predictions may include:

- expected row counts or cardinality;
- expected uniqueness of keys;
- expected direction of a transformation;
- expected handling of missing values;
- expected date boundaries;
- expected output for a hand-calculated example;
- expected coefficient or pattern in a simple simulated case;
- expected facts that should or should not appear in a document.

**Prediction must precede inspection whenever practical.** Otherwise it is too easy to rationalise an incorrect output after seeing it.

### 2.3 Implement

Use AI, documentation, packages and existing code to implement the smallest auditable steps possible.

Implementation rules:

- break complex transformations into inspectable stages;
- avoid unnecessary cleverness;
- preserve stable row identifiers when useful;
- keep assumptions visible in code or project documentation;
- prefer deterministic and reproducible procedures;
- do not treat “the code ran” as evidence of correctness.

### 2.4 Inspect

Immediately inspect the consequences of each critical step.

Typical checks include:

- row counts before and after;
- uniqueness of keys;
- duplicate creation;
- unmatched records;
- missingness before and after;
- value ranges and impossible values;
- distributions and extreme observations;
- group counts;
- date coverage;
- random or deliberately selected individual-level traces;
- comparison with known totals or hand calculations.

Inspection asks: **Did the data actually change in the way we predicted?**

### 2.5 Challenge

Actively try to make the result fail.

Examples:

- edge cases;
- alternative specifications;
- different date boundaries;
- duplicate keys;
- missing values;
- unusual groups;
- alternative definitions;
- sensitivity analyses;
- placebo or negative-control logic where appropriate;
- independent implementation of a key calculation;
- asking AI to critique the analysis after the logic has been specified.

The objective is not to defend the preferred result. It is to discover where it breaks.

### 2.6 Reproduce

A trusted result must be reproducible from a defined starting point.

Where applicable:

- rerun from raw or approved analytical inputs;
- avoid undocumented manual edits;
- keep code, parameters and versions under version control;
- fix random seeds when randomness matters;
- regenerate tables and figures from source data;
- ensure the same workflow produces the same result.

### 2.7 Explain

Before a result is considered complete, the owner must be able to explain:

- what was done;
- why it was done;
- what assumptions it relies on;
- how it was validated;
- what could still be wrong;
- what the result does and does not imply;
- what would change the conclusion.

A useful final test is:

> **If a supervisor, reviewer, manager or interviewer points to any important step and asks “why?”, can I answer without saying “because AI told me to”?**

---

# 3. Validation contracts for data work

For every **critical data transformation**, create a small validation contract.

A critical transformation is any step that can change the meaning, membership, timing, linkage or numerical value of the analytical data. Examples include joins, filters, deduplication, aggregation, longitudinal episode construction, date/lag logic, exposure assignment, matching, weighting and outcome construction.

The validation contract should specify:

| Item | Question |
| --- | --- |
| Input grain | What does one input row represent? |
| Key | Which variables identify a row/entity? Are they unique? |
| Intended transformation | What exactly should change? |
| Prediction | What should the output look like before we run it? |
| Toy/known case | What minimal example can be checked by hand? |
| Invariants | What must remain true before and after? |
| Real-data diagnostics | Which counts/distributions/records will be checked? |
| Failure signal | What result would make us stop and investigate? |

---

# 4. Toy-data rule

## 4.1 When toy data are required

Use toy data, known cases or an equivalent hand-verifiable test at every critical logical node where a mistake could silently propagate.

This is especially important for:

- joins;
- one-to-many / many-to-one relationships;
- deduplication;
- episode construction;
- overlapping intervals;
- date shifting;
- lag/lead creation;
- rolling windows;
- aggregation;
- weighting;
- matching;
- exposure assignment;
- classification rules;
- complex conditional logic;
- custom functions;
- model-preparation datasets.

A toy dataset should be **small enough to solve by hand** but include the edge cases that matter.

## 4.2 A good toy dataset is adversarial

Do not build only the easy case.

Where relevant include:

- one perfect match;
- one unmatched row;
- duplicate keys;
- missing values;
- boundary dates;
- overlapping intervals;
- zero counts;
- ties;
- an entity appearing in multiple periods/groups;
- values exactly at classification cut-offs.

Before running the code, write the expected output.

Then compare code output with the manually predicted output.

## 4.3 What toy data prove — and what they do not

Toy data can provide strong evidence that **the implemented logic behaves correctly for explicitly tested cases**.

They do **not**, by themselves, establish that:

- the scientific question is well posed;
- the identification strategy is valid;
- the statistical model is appropriate;
- the assumptions hold in the real population;
- the real data are free from measurement error or selection problems.

Implementation validity and scientific validity are separate layers.

---

# 5. SQL standard

For SQL work on large administrative data, every important transformation must be validated rather than inferred from a successful query.

## Joins

Before joining:

1. identify the intended relationship: 1:1, 1:m, m:1 or m:m;
2. test uniqueness of each join key;
3. predict the expected effect on row counts;
4. construct a toy join including duplicates and unmatched rows.

After joining:

- compare row counts;
- count unmatched records on each side;
- check whether duplicates were introduced;
- inspect several known IDs/row IDs end-to-end;
- verify that a many-to-many join is intentional rather than accidental.

## Filters

Check:

- counts before and after;
- excluded categories;
- missing-value behaviour;
- boundary conditions;
- a small set of records expected to remain and to disappear.

## Aggregations

Verify:

- grouping grain;
- number of groups;
- whether sums/counts reconcile with the pre-aggregation data;
- a hand-calculated toy group.

## Dates, intervals and lags

Explicitly test:

- first and last dates;
- leap years where relevant;
- inclusive vs exclusive interval boundaries;
- same-day vs prior-day definitions;
- lag 0 vs lag 1;
- gaps and overlapping spells.

For large confidential datasets, synthetic toy records are preferred to copying real confidential records outside the approved environment.

---

# 6. Python and R standard

For important functions or transformations:

- define the expected input and output shape;
- test a minimal hand-verifiable case;
- test missing values and edge cases;
- use assertions where a condition must always hold;
- compare important intermediate objects rather than only final output;
- validate joins, groups, date logic and reshaping exactly as in SQL;
- when rewriting code in another language, compare outputs on the same small dataset.

For statistical code, separate:

1. **data-construction correctness**;
2. **model implementation correctness**;
3. **statistical/design validity**.

A successful model fit only addresses none of these automatically.

---

# 7. Statistical and research-method standard

AI may propose models, estimators or research designs, but an unfamiliar method remains a **candidate**, not an accepted choice.

Before a method becomes part of a formal analysis:

1. define the estimand;
2. explain why the data structure can identify or estimate it;
3. identify the key assumptions;
4. consult primary methodological literature, authoritative documentation or both;
5. verify that the implementation matches the intended method;
6. test the implementation on toy or simulated data where useful;
7. examine diagnostics and sensitivity to defensible alternatives;
8. distinguish robustness of implementation from robustness of scientific conclusion.

For causal or attribution work, explicitly separate:

- observed association;
- identification assumptions;
- counterfactual construction;
- statistical estimation;
- uncertainty;
- causal interpretation.

Do not let AI collapse these into one step.

---

# 8. Results standard

Every reported result should pass four questions:

1. **Source:** Which code/output/data produced this number?
2. **Scale:** What exactly is the unit, reference group, denominator or effect scale?
3. **Uncertainty:** What uncertainty accompanies it?
4. **Interpretation:** Does the written claim stay within what the result supports?

When results are surprising:

- do not immediately “explain” them;
- first check data construction, sample composition, coding, model specification and scale;
- then consider substantive explanations.

---

# 9. Academic writing standard

AI can be used extensively for English expression, structure, editing and translation. The boundary is not who typed the sentence; the boundary is whether the underlying claim is owned and verified.

Every substantive sentence should be understood as one of:

- **Data fact** — directly supported by the dataset/output;
- **Literature fact** — supported by a verified source;
- **Analytical result** — produced by the analysis;
- **Interpretation** — a reasoned explanation of results;
- **Hypothesis/speculation** — a plausible but unconfirmed explanation;
- **Recommendation** — a normative or policy judgment.

AI must not silently move a statement from one category to another.

Rules:

- never accept a citation solely because AI supplied it;
- verify citations against the original source;
- check every reported number against the final output;
- do not let language polishing add mechanisms, certainty or causality that the analysis did not establish;
- when AI proposes an unfamiliar explanation, treat it as a lead to investigate, not as manuscript-ready knowledge;
- final text should be something the author can defend sentence by sentence.

AI may perform **linguistic encoding** of the author's ideas. New scientific content requires independent verification before inclusion.

---

# 10. Literature and factual claims

Use an evidence hierarchy.

For important claims, prefer:

1. primary data / original output;
2. original methodological or substantive research;
3. official documentation or authoritative institutional sources;
4. high-quality secondary synthesis;
5. AI-generated explanation only as a starting point.

AI is useful for finding search terms, explaining concepts and comparing sources. It is not itself the evidentiary source.

When an important claim matters to a decision, seek independent confirmation rather than asking the same AI the same question repeatedly.

---

# 11. Job applications and professional profiles

AI can help tailor CVs, cover letters and application answers, but every claim must be traceable to real experience.

Rules:

- do not invent responsibilities, software use, achievements or metrics;
- distinguish “used with AI assistance” from “cannot operate without AI”;
- describe proficiency based on the ability to frame tasks, implement, debug, validate and adapt workflows — not on memorising syntax;
- only claim a method/tool at a level that can be demonstrated and discussed in an interview;
- retain the applicant's own motivations and judgments; AI may improve expression.

A defensible description of modern technical work is:

> I use AI as a coding accelerator for syntax, implementation and unfamiliar APIs, while retaining responsibility for analytical logic, debugging, validation and interpretation.

Using AI does not invalidate proficiency in Python, R or SQL when the user can independently define the computational task, understand and modify the workflow, diagnose problems and validate the result.

---

# 12. Workplace analysis and deliverables

For professional work, add explicit acceptance criteria before analysis begins.

A deliverable should document, at the appropriate level:

- business/policy question;
- data source;
- transformation logic;
- assumptions;
- validation checks;
- known limitations;
- version or date;
- reviewer/sign-off where risk warrants it.

Higher-stakes outputs require stronger independent checks.

The objective is not “zero possibility of error”. The objective is a system in which important errors have multiple opportunities to be detected before delivery.

---

# 13. Professional communication

AI can draft emails, messages and reports.

The human owner remains responsible for:

- factual accuracy;
- commitments and deadlines;
- recipients;
- confidentiality;
- tone;
- statements that could affect professional relationships.

Before sending, check: **Is every commitment in this message one I actually intend to make?**

---

# 14. Risk-based validation

Not every task needs the same burden of proof.

| Risk | Examples | Minimum validation |
| --- | --- | --- |
| Low | wording, formatting, brainstorming | meaning check |
| Medium | data cleaning, descriptive analysis, job materials | source check + toy/known cases + diagnostics |
| High | model design, causal claims, published results, policy/production decisions | authoritative sources + implementation tests + diagnostics + sensitivity/independent review where feasible |

Do not waste effort proving trivial things repeatedly. Concentrate validation where an unnoticed error could materially change the conclusion or deliverable.

---

# 15. The AI must help design the verification, not only the solution

When AI proposes code or a workflow, it should also propose how to verify it.

For each critical step, AI should provide or help construct:

1. the prediction;
2. a toy or known-case test;
3. real-data diagnostics;
4. explicit pass/fail conditions;
5. likely failure modes.

“Run this code and inspect whether it looks reasonable” is not an adequate validation plan for an important transformation.

---

# 16. New-project protocol

At the beginning of every substantive new project:

1. state the project question and intended deliverable;
2. identify high-risk decisions;
3. create a validation plan for critical data/method steps;
4. decide which claims require external authoritative sources;
5. define what AI may assist with;
6. identify what must remain a human sign-off decision;
7. record important methodological decisions and why they were made.

For data projects, begin a validation table such as:

| Step | Prediction | Toy/known-case test | Real-data check | Pass condition |
| --- | --- | --- | --- | --- |
| Example: join A to B | one B record per A record | 4-row synthetic join | key uniqueness + pre/post counts | no unintended duplication |
| Example: construct lag 1 | exposure shifts exactly one day | 5-day sequence | inspect boundary dates | hand result matches |
| Example: aggregate person-days | totals preserved within group | 2-person toy dataset | reconcile counts | group sums equal source totals |

The table should evolve with the project.

---

# 17. Final sign-off checklist

Before treating an important output as finished:

- [ ] I can state the question and estimand/output clearly.
- [ ] I wrote down the expected behaviour of critical steps.
- [ ] Critical transformations have toy/known-case or equivalent tests.
- [ ] Real-data row counts, keys, missingness and distributions were inspected where relevant.
- [ ] Important edge cases were challenged.
- [ ] The workflow can be reproduced from a defined starting point.
- [ ] Important methods/facts are grounded in authoritative sources rather than AI alone.
- [ ] Reported numbers match final outputs.
- [ ] Interpretation does not exceed the evidence.
- [ ] I can explain the main logic, assumptions and limitations.
- [ ] I am willing to take responsibility for the final deliverable.

---

## Operating philosophy

The objective is **not to minimise AI use**.

The objective is to increase output capability **without surrendering epistemic responsibility**.

A strong AI-assisted researcher or analyst is not the person who manually performs every operation. It is the person who can organise questions, data, methods, software, evidence and AI into a system that produces results that are inspectable, challengeable and defensible.

**AI can generate. The owner must verify, understand and sign off.**
