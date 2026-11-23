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

A successful model fit does not establish any of these automatically.

---

# 7. Research project standard: from broad idea to defensible design

AI can help move a project from a broad idea to a concrete research design, but the project must not become a sequence of AI-selected defaults. The researcher should be able to reconstruct the reasoning that connects the question, estimand, design, data and model.

The central rule is:

> **Question first → estimand second → design third → data feasibility fourth → model fifth → implementation sixth.**

In practice these stages interact and may require iteration, but a convenient model or available dataset should not silently redefine the scientific question.

## 7.1 Start by converting the broad idea into a research target

A broad idea such as “What is the effect of A on B?” is not yet a research design.

Before selecting data or a model, specify as far as possible:

- **Phenomenon:** what relationship or mechanism is of interest?
- **Target population:** to whom should the result apply?
- **Exposure/treatment/predictor:** what exactly is A?
- **Outcome:** what exactly is B?
- **Time scale:** immediate, cumulative, annual, lifetime, event-time, etc.?
- **Geographical scale:** individual, household, neighbourhood, region, country, etc.?
- **Research purpose:** descriptive, associative, predictive, causal, attributional, forecasting or policy evaluation?
- **Estimand:** what quantity would answer the question?
- **Unit of analysis:** what does one analytical observation represent?
- **Counterfactual, if relevant:** compared with what alternative state?
- **Decision relevance:** what would be learned if the answer were known?

If several versions are plausible, keep them as alternatives rather than forcing an early choice.

## 7.2 Build a research decision map before committing to one approach

For any consequential research choice, identify the plausible alternatives.

Examples:

- individual-level versus area-level analysis;
- observational versus quasi-experimental design;
- station observations versus gridded exposure data;
- short high-quality period versus long lower-quality period;
- simple linear specification versus flexible non-linear specification;
- complete-case analysis versus a method that retains partial information;
- one outcome definition versus another defensible definition.

The purpose is not to compare every imaginable method. It is to compare the **serious alternatives that could materially change the answer**.

For each alternative, ask:

1. What question does this option actually answer?
2. What assumptions does it require?
3. What data does it require?
4. What major bias or failure mode does it reduce?
5. What new weakness does it introduce?
6. What information is lost or gained?
7. How interpretable is the resulting estimand?
8. Can the method be implemented and validated with the available resources?
9. Would a reasonable researcher accept the assumptions even if the result were inconvenient?

AI should help construct this comparison, not simply recommend one option.

## 7.3 Use a research-design trade-off matrix

When the choice is non-trivial, compare candidate approaches across the following dimensions.

| Dimension | Question |
| --- | --- |
| Question alignment | Does the design estimate the quantity the research question actually asks for? |
| Identification strength | What variation identifies the effect, and how credible is it? |
| Counterfactual quality | Is the comparison group/state meaningful and defensible? |
| Assumption burden | Which assumptions are required, and which are most fragile? |
| Confounding/control | Which important sources of confounding are handled, and which remain? |
| Measurement quality | How accurately are exposure, treatment, outcome and covariates measured? |
| Population coverage | Who is included or excluded, and does that change the target population? |
| Time coverage | Is the available period long enough and comparable across time? |
| Spatial/temporal resolution | Does the data resolution match the mechanism and estimand? |
| Missingness/selection | Could missing data or linkage/selection processes distort the sample? |
| Statistical information | Is there enough variation, sample size and effective information for the design? |
| Model dependence | How much does the answer depend on functional-form or tuning choices? |
| Interpretability | Can the estimand and model output be explained clearly? |
| Robustness potential | Can the main threats be tested or bounded empirically? |
| External validity | To which settings or populations can the result plausibly generalise? |
| Comparability | Can the result be compared with relevant literature or previous work? |
| Novelty/value added | Does this option answer something meaningfully new rather than merely more complicated? |
| Feasibility | Can the data, software, time and governance requirements actually be met? |
| Reproducibility/auditability | Can the analytical chain be reconstructed and checked? |

Do not reduce this table to a mechanical score unless scoring is genuinely useful. Some weaknesses are **fatal**, while others are acceptable trade-offs.

A design with slightly less statistical efficiency may be preferable if it has substantially stronger identification. A larger dataset may be inferior if the key variable is badly measured. A more sophisticated model may be inferior if the estimand becomes opaque and the added flexibility does not address an identified threat.

## 7.4 Data choice: more data is not automatically better data

When choosing between datasets or exposure sources, assess at least:

- target-population coverage;
- inclusion/exclusion rules;
- linkage quality;
- missingness;
- measurement error;
- coding changes over time;
- temporal completeness;
- spatial completeness;
- temporal and spatial alignment with the outcome;
- granularity;
- consistency across years;
- availability of important confounders or stratifiers;
- confidentiality/governance constraints;
- computational feasibility;
- whether the data-generating process changes during the study period.

Important trade-offs should be explicit.

Examples of legitimate trade-offs include:

- **longer time coverage vs more consistent measurement**;
- **greater spatial resolution vs noisier measurement**;
- **individual-level detail vs incomplete linkage**;
- **direct observation vs modelled/predicted exposure with better coverage**;
- **larger sample vs a more narrowly defined but cleaner population**.

The decision should follow the research question. Do not prefer a dataset merely because it is larger, newer, more granular or easier to use.

For each key data source, distinguish:

1. **What is directly observed?**
2. **What is linked or inferred?**
3. **What is modelled or imputed?**
4. **What is unavailable?**
5. **What errors could this introduce into the estimand?**

## 7.5 Research design: choose the source of identification before the regression command

A statistical model does not create identification by itself.

Before choosing the estimator, state:

- what variation identifies the parameter;
- which observations are being compared with which;
- what makes those observations comparable;
- which confounders are controlled by design;
- which confounders rely on modelling;
- what temporal ordering is assumed;
- whether selection into the sample can depend on exposure or outcome;
- whether interference, spillovers or repeated observations matter;
- whether standard errors must account for clustering, serial correlation or repeated measures.

Candidate designs should be judged by their **identifying logic**, not by which one is more fashionable or produces a cleaner result.

For causal work, explicitly identify the counterfactual logic. For descriptive or associative work, avoid importing causal language that the design cannot support.

## 7.6 Model choice: the model serves the design, not the reverse

Once the estimand and design are clear, select a model that implements them.

Default principles:

- start with the simplest defensible specification that answers the question;
- add complexity only when it addresses a substantive feature or identified statistical problem;
- distinguish necessary flexibility from decorative complexity;
- prefer specifications whose assumptions can be inspected and explained;
- do not select the final model because it yields the most significant, novel or convenient result.

When comparing models, examine:

- estimand produced by each model;
- functional-form assumptions;
- treatment/exposure-response shape;
- time trend and seasonality handling;
- lag structure;
- interaction structure;
- fixed/random effects or stratification;
- weighting;
- variance estimator;
- clustering;
- regularisation or smoothing;
- extrapolation;
- effective degrees of freedom;
- convergence and numerical stability;
- sensitivity to influential observations;
- interpretability.

Where appropriate, maintain:

- a **primary model** chosen for substantive reasons;
- a small set of **alternative defensible specifications** linked to specific threats or uncertainties.

The alternatives should not become an unlimited specification search.

## 7.7 Separate four questions that are often incorrectly collapsed

For every major model, separately ask:

### A. Is the scientific question coherent?
This is about the concept and estimand.

### B. Can the available data answer it?
This is about measurement, coverage, timing and selection.

### C. Does the research design identify or estimate the target quantity?
This is about assumptions and counterfactual logic.

### D. Does the code correctly implement the chosen design/model?
This is about computation.

A correct answer to D cannot rescue a failure in A–C.

## 7.8 Use a layered validity framework

A research result should be examined through distinct layers:

1. **Conceptual validity** — does the estimand represent the scientific idea?
2. **Measurement validity** — do the observed variables adequately represent the intended concepts?
3. **Design/identification validity** — does the comparison recover the intended association/effect under defensible assumptions?
4. **Data-construction correctness** — were cohorts, exposures, outcomes, dates and covariates constructed as intended?
5. **Implementation correctness** — does the code implement the algorithm/model correctly?
6. **Statistical adequacy** — are model assumptions, uncertainty estimates and diagnostics acceptable?
7. **Robustness** — do conclusions survive defensible alternatives targeted at real threats?
8. **Substantive plausibility** — are results coherent with the data-generating context and relevant evidence?
9. **Reporting validity** — does the manuscript claim no more than the analysis establishes?

Each layer requires different evidence. No single p-value, robustness check, simulation or code review can validate all nine.

## 7.9 Before seeing the main result, state expectations and failure signals

Where practical, document what would be expected if the implementation is correct.

This is not a requirement to predict the scientific answer. It is a requirement to distinguish **pre-result reasoning** from **post-result rationalisation**.

Examples:

- expected row-count changes;
- known relationships in toy data;
- sign or scale implied by a simulated data-generating process;
- values that must be impossible;
- coefficients that should be close to zero in a placebo case;
- known benchmark totals;
- qualitative patterns expected from the design rather than from the hypothesis.

For exploratory research, hypotheses may remain open. Label exploration as exploration rather than inventing a prior prediction afterwards.

## 7.10 Robustness should target threats, not generate a wall of regressions

Every important robustness or sensitivity analysis should answer:

> **Which possible failure of the main analysis does this test address?**

Use a threat-based table:

| Threat | Why it matters | Diagnostic / alternative | What result would reduce concern? | What result would change interpretation? |
| --- | --- | --- | --- | --- |
| Exposure misclassification | estimated effect may be attenuated or spatially distorted | alternative exposure source / restriction | similar substantive pattern | large change tied to exposure definition |
| Temporal specification | residual time structure may confound estimates | alternative trend/seasonality control | stable main inference | materially different effect |
| Lag choice | cumulative effect depends on temporal window | alternative defensible lag windows | stable shape/summary | reversal or strong instability |
| Sample definition | result may depend on selected population | alternative eligibility rule | similar inference | subgroup/sample-driven result |
| Variance estimation | inference may be overstated | alternative valid SE/cluster method | similar uncertainty conclusion | significance/inference changes materially |

Robustness does not mean “the coefficient stays identical”. The question is whether the substantive conclusion remains defensible and, if it changes, whether the reason is understood.

## 7.11 When two defensible options disagree, investigate rather than average them away

If two credible designs, data sources or models produce different answers:

1. confirm that both implementations are correct;
2. identify which observations or assumptions differ;
3. determine whether the estimands are actually the same;
4. locate the source of disagreement;
5. assess which assumptions are more credible for the stated question;
6. report consequential sensitivity rather than hiding it.

Disagreement is information. It may reveal measurement problems, effect heterogeneity, model dependence or a poorly defined estimand.

## 7.12 Rules for AI-proposed methods

When AI suggests an unfamiliar design, estimator, package or interpretation, treat it as a research lead.

Before adoption:

1. ask AI for at least one serious alternative;
2. ask what assumptions distinguish the options;
3. identify failure modes and data requirements;
4. verify the method in original methodological literature and/or authoritative documentation;
5. confirm that the method has actually been used for comparable data structures where relevant;
6. reproduce a simple example or simulation;
7. explain the method in plain language;
8. decide whether the added complexity solves a real research problem.

A method is not preferable merely because it is newer, more technical or more publishable-sounding.

## 7.13 Research stage gates

A paper can move through the following gates. Returning to an earlier gate is normal when new information appears.

### Gate 1 — Question and estimand
Before heavy data work:
- research question is explicit;
- target population/outcome/exposure are defined;
- research purpose is classified;
- estimand or target quantity is stated.

### Gate 2 — Data feasibility
Before committing to the design:
- required variables exist;
- study period and coverage are known;
- missingness/linkage/measurement issues are mapped;
- major data limitations are identified;
- confidentiality/governance constraints are feasible.

### Gate 3 — Design choice
Before large-scale modelling:
- identifying comparison is explicit;
- serious alternatives have been considered;
- main assumptions are documented;
- reasons for the chosen design are stronger than convenience alone.

### Gate 4 — Analytical construction
Before interpreting results:
- critical transformations have validation contracts;
- toy/known cases pass;
- real-data diagnostics pass;
- model implementation has been checked.

### Gate 5 — Scientific challenge
Before treating the main result as stable:
- key threats have targeted sensitivity analyses;
- surprising results have been traced through the pipeline;
- disagreement across defensible specifications is understood.

### Gate 6 — Manuscript claim
Before submission:
- every important number traces to final output;
- causal/associational language matches the design;
- literature claims trace to verified sources;
- limitations correspond to real unresolved threats;
- the researcher can explain why the selected design/data/model were chosen over the main alternatives.

## 7.14 General research choice template

For a consequential decision, use a table like this:

| Decision | Option | What it answers | Main assumptions | Data requirements | Main advantage | Main weakness | Validation needed | Reason to choose/reject |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Design | A | ... | ... | ... | ... | ... | ... | ... |
| Design | B | ... | ... | ... | ... | ... | ... | ... |
| Data source | A | ... | ... | ... | ... | ... | ... | ... |
| Data source | B | ... | ... | ... | ... | ... | ... | ... |
| Model | A | ... | ... | ... | ... | ... | ... | ... |
| Model | B | ... | ... | ... | ... | ... | ... | ... |

The table exists to expose reasoning, not to create paperwork. Use it when the choice could materially change the scientific conclusion.

## 7.15 Practical trade-off rules

When choices remain difficult, use the following priorities:

- **For causal questions, identification usually outranks sample size.**
- **For measurement-sensitive questions, data quality can outrank granularity.**
- **For long-period studies, comparability over time can outrank nominal coverage length.**
- **For complex models, additional flexibility must justify its assumptions and loss of interpretability.**
- **For competing datasets, the source that better measures the target construct may be preferable even if it is less convenient.**
- **For competing models with similar validity, prefer the more transparent and auditable one.**
- **If a weakness could reverse the conclusion, treat it as a design issue, not a footnote.**
- **If two options estimate different quantities, do not compare them as though they were competing estimates of the same estimand.**
- **Do not optimise simultaneously for significance, novelty and simplicity. Optimise for a defensible answer to the stated question.**

## 7.16 Research decision standard

The final question is not:

> “Is this the best possible design in the abstract?”

It is:

> **“Given the scientific question, available data, feasible alternatives and known assumptions, is this a defensible design, and can I explain why its trade-offs are preferable to the main alternatives?”**

That is the level of reasoning required before AI-assisted research becomes researcher-owned research.

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

At the beginning of every substantive new project, use this standard as the default operating framework rather than creating a separate AI rulebook for each project.

## 16.1 For a research paper

Before heavy implementation, create a compact project map covering:

1. **Broad idea** — what phenomenon is being investigated?
2. **Research question** — what exact question could become the paper's central question?
3. **Target estimand** — what quantity would answer it?
4. **Candidate designs** — what are the main serious alternatives?
5. **Data map** — which sources are required, what do they measure, and where are the gaps?
6. **Trade-offs** — what is gained and lost under each plausible design/data choice?
7. **Identification assumptions** — what must be true for the preferred interpretation?
8. **Model strategy** — what is the simplest defensible primary model, and which alternatives correspond to real threats?
9. **Validation plan** — which transformations require toy/known-case tests, diagnostics or simulation?
10. **Evidence plan** — which methodological or factual claims require primary literature or official documentation?
11. **Decision points** — which choices require explicit researcher sign-off?
12. **Stopping rule** — what evidence would make the current approach unsuitable and force redesign?

The project map can live in the paper's normal research notes or README. A separate AI-validation document is not required unless there is a specific reason to create one.

## 16.2 For data/analysis projects

For each critical node, maintain a validation table such as:

| Step | Prediction | Toy/known-case test | Real-data check | Pass condition |
| --- | --- | --- | --- | --- |
| Example: join A to B | one B record per A record | 4-row synthetic join | key uniqueness + pre/post counts | no unintended duplication |
| Example: construct lag 1 | exposure shifts exactly one day | 5-day sequence | inspect boundary dates | hand result matches |
| Example: aggregate person-days | totals preserved within group | 2-person toy dataset | reconcile counts | group sums equal source totals |

## 16.3 For writing, job applications and workplace deliverables

Define:

- intended audience;
- intended claim/action;
- source material;
- factual constraints;
- what AI may draft or transform;
- what requires human verification;
- final acceptance criteria.

The level of documentation should be proportional to risk. The purpose is to preserve reasoning and verification, not to create unnecessary process.

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
- [ ] For consequential research choices, I can explain why the selected design, data source and model are preferable to the main alternatives.
- [ ] I am willing to take responsibility for the final deliverable.

---

## Operating philosophy

The objective is **not to minimise AI use**.

The objective is to increase output capability **without surrendering epistemic responsibility**.

A strong AI-assisted researcher or analyst is not the person who manually performs every operation. It is the person who can organise questions, data, methods, software, evidence and AI into a system that produces results that are inspectable, challengeable and defensible.

**AI can generate. The owner must verify, understand and sign off.**
