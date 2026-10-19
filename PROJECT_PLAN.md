# AI Cart Recommendations A/B Test: End-to-End Project Plan

## Project brief

**Scenario:** An e-commerce platform tests an AI-driven product recommendation widget on the cart page against the current generic top-selling-items list.

**Business question:** Does showing personalized cart recommendations increase revenue per eligible cart, by improving cart conversion and/or the value of completed orders, without harming customer or business guardrails?

This project will demonstrate the full lifecycle of a controlled experiment: defining a decision, designing a statistically sound test, generating reproducible simulated data, validating it, analyzing outcomes, and communicating a launch recommendation.

> **Important:** This is a portfolio simulation, not evidence about a real product. Simulated results must never be presented as observed customer behavior.

| Item | Plan |
|---|---|
| Audience | Product manager, product analyst, and analytics/recruiting reviewers |
| Primary outcome | Revenue per randomized eligible cart (total attributed revenue / randomized eligible carts) |
| Key component metrics | Cart conversion rate and average order value (AOV) among completed orders |
| Experiment unit | User at their first eligible cart; keep assignment persistent for returning users |
| Variants | Control: generic top-selling-items list; Challenger: AI-driven personalized recommendations |
| Allocation | 50/50 randomized assignment |
| Analysis | Intention-to-treat (analyze users in their assigned groups) |
| Estimated effort | 18–22 hours, including dashboard build |
| Tools | Python, pandas, NumPy, SciPy, statsmodels, matplotlib/seaborn, and Power BI or Tableau |

## Decision and success criteria

The experiment should answer whether to launch, iterate, or stop. Before generating or inspecting outcomes, record the decision criteria in the experiment configuration and report:

- **Primary metric:** revenue per randomized eligible cart = total revenue attributed to index carts / number of randomized eligible users (one index cart per user). Include non-converting carts with zero revenue. Predefine a consistent revenue definition, including the treatment of discounts, shipping, taxes, and refunds. This is the most direct user-level business outcome and avoids interpreting AOV among purchasers as if treatment had not changed who purchases.
- **Key component metrics:** cart conversion rate = randomized eligible carts with an attributed purchase / randomized eligible carts; AOV = attributed revenue / completed orders. Report their denominators and confidence intervals. AOV is conditional on ordering, so changes in who converts can change AOV; interpret it alongside conversion and revenue per cart rather than as an isolated causal effect.
- **Statistical tests:** two-sided tests at `alpha = 0.05`: an appropriate user/cart-level mean comparison with a confidence interval for revenue per randomized cart (for example, bootstrap or randomization inference because revenue is often skewed), and a two-proportion test for cart conversion. Treat component metrics as secondary; adjust for multiple confirmatory secondary comparisons (for example, Holm) if making significance claims about both conversion and AOV.
- **Power target:** at least 80% for the pre-agreed minimum detectable effect (MDE).
- **Practical significance:** the estimated lift must meet the product team's minimum worthwhile improvement, not merely have a small p-value.
- **Guardrails:** monitor recommendation-widget errors/latency, checkout errors, refunds/cancellations, and customer experience. Define acceptable thresholds before the test; do not trade a severe guardrail regression for revenue growth.
- **Decision rule:** recommend launch only when the primary result meets the pre-specified statistical and practical criteria and no guardrail breaches its agreed limit. Otherwise recommend no launch, more evidence, or a follow-up test, as appropriate.

### Planning assumptions to confirm

There is no real historical dataset or product specification in this repository yet. For a reproducible demo, define illustrative baselines for cart conversion and order value, plus a minimum worthwhile lift in revenue per randomized cart; use 5% two-sided significance, 80% power, and equal allocation. Choose numeric values explicitly in the experiment configuration before simulating results. These are **illustrative assumptions**, not industry benchmarks. Replace them with validated historical estimates if real data becomes available.

Calculate the sample size for the primary revenue-per-cart outcome using a documented variance estimate and suitable power method; use historical cart-level data or clearly stated simulation assumptions. Separately assess power for cart conversion, and account for the chosen multiple-testing policy for confirmatory component outcomes. Add allowance for expected invalid/ineligible records only after documenting the reason and rate. Do not select sample size after looking at simulated outcomes.

## Phased delivery plan

Each phase has a concrete output and a gate. Complete the gate before moving on; this keeps the analysis auditable and prevents outcome-driven design changes.

### Phase 1 — Define the product question and estimand (1–2 hours)

**Work**

1. Write the decision the experiment will inform and identify the intended user population.
2. Define cart eligibility, the randomization unit, assignment moment, exposure, recommendation attribution, and purchase window. Randomize each user at their first eligible cart and keep assignment persistent; attribute the primary outcome to that index cart to avoid counting repeat carts as independent units.
3. Specify the primary metric, guardrails, and any diagnostic metrics.
4. Decide how repeated visits, cross-device identities, missing events, and users whose recommendation widget fails to render will be handled.
5. Document assumptions and exclusions before creating outcome data.

**Deliverables:** experiment brief; metric dictionary; eligibility and exclusion rules.

**Gate:** another analyst can determine exactly who belongs in the denominator and what counts as a conversion without making assumptions.

### Phase 2 — Pre-register the experiment design and power (2–3 hours)

**Work**

1. State hypotheses for revenue per randomized cart and cart conversion. Example primary hypothesis: `H0: mean revenue/cart is equal`; `H1: mean revenue/cart differs`. State secondary hypotheses for conversion and AOV and their multiplicity handling.
2. Record the two-sided alpha, target power, baseline conversion and revenue/cart, practical MDE, allocation ratio, and test horizon.
3. Calculate and save the required number of randomized users per arm and the assumptions used.
4. Set a fixed sample-size or calendar-time stopping rule. Do not stop early because a p-value becomes favorable.
5. Record the analysis plan, decision criteria, guardrail limits, and any planned subgroup analyses.

**Deliverables:** version-controlled experiment configuration; power calculation output; pre-analysis plan.

**Gate:** sample size and stopping rule are fixed before outcome analysis. If the calculated sample is impractical, revise the design with a documented product decision—not by silently lowering power.

### Phase 3 — Create a reproducible synthetic experiment (2–3 hours)

**Work**

1. Generate one row per eligible user at their first eligible cart, with a stable unique ID, index-cart ID, assigned variant, assignment timestamp, exposure status, and outcome fields.
2. Randomly assign users 50/50 using a recorded seed. Keep assignment independent of user characteristics and outcomes.
3. Simulate cart conversion and attributed order value using configured baselines and treatment effects; derive revenue per randomized cart with zero revenue for non-converting carts.
4. Simulate widget rendering/latency, checkout errors, refunds/cancellations, and other guardrails with explicit assumptions.
5. Include realistic data-quality cases only if they are labeled and intentionally used to test validation (for example, duplicate IDs or missing assignment).

**Deliverables:** generator script; synthetic data file; metadata describing the seed, assumptions, and schema.

**Gate:** rerunning with the same configuration and seed produces identical data; changing the seed produces a new valid sample. Synthetic records are clearly labeled and contain no real personal data.

### Phase 4 — Build data checks and experiment monitoring (2–3 hours)

**Work**

1. Validate schema, types, unique user IDs, allowed variant values, timestamp order, and nullability.
2. Check assignment counts and the sample ratio mismatch (SRM) against the planned allocation.
3. Verify eligibility, exposure, purchase attribution, and revenue definitions; identify duplicate carts/orders or impossible event sequences.
4. Report missingness, exclusions, and data-quality failures rather than silently dropping records.
5. Add unit tests for core metric calculations and validation failures.

**Deliverables:** validation module; data quality summary; automated tests.

**Gate:** invalid input fails clearly, and the report records any exclusions. An SRM or instrumentation failure blocks interpretation until investigated.

### Phase 5 — Run the pre-specified primary and guardrail analysis (3–4 hours)

**Work**

1. Produce a sample flow and summarize users assigned, exposed, eligible for analysis, and excluded by reason.
2. Calculate revenue per randomized cart, cart conversion rate, and AOV per arm, with absolute/relative effects and confidence intervals.
3. Analyze the primary revenue-per-cart outcome with a method robust to skew and zero inflation (for example, bootstrap or randomization inference); analyze conversion with a two-proportion test. Report estimates, uncertainty, and p-values.
4. Analyze AOV among completed orders with its conditional denominator and interpret alongside conversion and revenue per cart. Apply the pre-specified multiple-testing policy to secondary confirmatory metrics.
5. Analyze guardrails using methods appropriate to their data types (for example, proportion methods for error/refund rates and a suitable latency comparison).
6. Check the design and data-quality assumptions before interpreting results.
7. Use cart/user-level binary conversion analysis. **Do not use a t-test for a binary conversion outcome**; revenue/order-value comparisons need uncertainty methods suitable for their skew and randomization unit.
8. Keep planned confirmatory analyses separate from exploratory segments. Label multiple comparisons and avoid claims based on underpowered subgroups.

**Deliverables:** executable analysis script or notebook; tables of effect estimates and uncertainty; saved analysis output.

**Gate:** another analyst can rerun the analysis from the saved data and configuration and reproduce the reported values.

### Phase 6 — Explain results and make the product recommendation (2–3 hours)

**Work**

1. Interpret the revenue-per-cart estimate and confidence interval in business terms, not just the p-value.
2. Compare the primary effect with both zero and the pre-agreed practical MDE; explain whether conversion and AOV component metrics support the business story.
3. Explain uncertainty, power, data limitations, and whether the design could detect a worthwhile effect.
4. Apply the decision rule, including guardrails, and recommend launch, no launch, or further testing.
5. Distinguish “no statistically significant difference” from proof that the variants are equivalent.

**Deliverables:** concise decision memo for a product manager; limitations and follow-up recommendations.

**Gate:** the recommendation follows the pre-specified criteria and does not overstate what simulated data can establish.

### Phase 7 — Build the native Power BI or Tableau results dashboard (2–4 hours)

**Work**

1. Recreate the supplied reference as a polished, one-page **A/B Test Results Summary**. Use a clear title and subtitle identifying the AI recommendations (Challenger) and generic best-sellers list (Control).
2. Put a **Test Overview** panel beside a **Key Results** table. Include goal, primary metric, test period, traffic split, randomized carts, statistical methods, and alpha in the overview. The results table compares randomized carts, converting carts, cart conversion rate, revenue per cart, AOV, and the relevant 95% confidence intervals.
3. Add four prominent result cards: **revenue per cart lift** (absolute and relative), **cart conversion lift** in percentage points, **AOV** with its order-based denominator, and the **primary p-value / 95% confidence interval**.
4. Finish with a prominent **Decision** banner: winner/no winner/inconclusive, the evidence supporting it, and a recommendation such as rollout, no rollout, or more testing. A rollout recommendation is conditional on guardrails passing.
5. Use a clean, accessible visual hierarchy inspired by the image: navy headings, blue comparison table, green positive-result treatment, and distinct but restrained colors for lift, p-value, and interval. Preserve readable labels, denominators, contrast, and uncertainty.
6. Show the actual experiment horizon, sample counts, allocation, and alpha. Mark the report prominently as a **simulation** when using generated data.
7. Treat the screenshot as a layout reference, not a numeric source of truth. Recalculate every displayed number from the reproducible analysis output; do not copy example values or hard-code a winner. Check that displayed counts, rates, lift, p-value, and confidence interval are mutually consistent.

**Deliverables:** native Power BI `.pbix` or Tableau `.twb`/`.twbx` dashboard matching the reference layout; export to a shareable image/PDF; documented Python-generated input data and supporting analysis output. A Python-generated image alone does not satisfy this deliverable.

**Gate:** the native dashboard opens in the selected BI tool, its numbers match the analysis outputs, a reviewer can understand the result without hidden filters, and the decision is generated from the configured criteria and guardrails.

### Phase 8 — Package, document, and review (2–3 hours)

**Work**

1. Organize code, data, configuration, tests, and outputs so a new user can reproduce the work.
2. Add setup and run instructions, dependency versions, data provenance, and a project limitations section.
3. Run the full validation and analysis from a clean environment.
4. Review the work for leakage (outcomes influencing design), incorrect denominators, inconsistent metrics, and unsupported claims.
5. Ensure generated files and any local environment/secret files are appropriately excluded from version control.

**Deliverables:** finished repository; reproducibility checklist; final review.

**Gate:** a reviewer can follow the documented commands from setup through final report without relying on undocumented manual steps.

## Proposed repository layout

Create this structure incrementally as each phase is implemented; it is a target layout, not a requirement to create empty files up front.

```text
.
├── README.md
├── pyproject.toml                 # dependencies and development/test configuration
├── configs/
│   └── experiment.json            # pre-specified assumptions and design
├── data/
│   ├── generated/                 # reproducible synthetic data; document tracking policy
│   └── reports/                   # generated tables and summaries
├── notebooks/
│   └── exploration.ipynb          # optional, exploratory work only
├── src/
│   └── ab_test/
│       ├── simulate.py
│       ├── validate.py
│       ├── metrics.py
│       └── analyze.py
├── tests/
│   ├── test_simulate.py
│   ├── test_validate.py
│   └── test_metrics.py
└── reports/
    ├── experiment_plan.md          # pre-analysis plan
    ├── decision_memo.md
    ├── ab_test_results_summary.png # exported dashboard preview
    └── dashboard/                  # native .pbix or .twb/.twbx deliverable
```

Keep the primary analysis in reusable Python modules rather than relying exclusively on notebook state. Notebooks may support exploration and visualization, but final results should run from a clean process. Do not commit large generated datasets unless their size and purpose justify it; document how to regenerate them.

## Data contract

The simulated analysis dataset should have one row per randomized user, representing that user's first eligible (index) cart. The initial schema can be extended only when the added field serves a stated experiment or validation need.

| Field | Type | Purpose |
|---|---|---|
| `user_id` | string | Stable identity for consistent assignment |
| `cart_id` | string | User's first eligible (index) cart identifier |
| `variant` | category/string | `control` or `treatment` |
| `assigned_at` | datetime | Time of randomized assignment |
| `exposed` | boolean | Whether the assigned experience was rendered |
| `eligible` | boolean | Whether the user met pre-defined eligibility |
| `converted` | boolean | At least one purchase attributed to the index cart within the window |
| `order_count` | integer | Number of attributed orders from the index cart |
| `conversion_at` | datetime or null | First attributed purchase time, when converted |
| `widget_rendered` | boolean | Whether assigned recommendation list was rendered |
| `widget_latency_ms` | numeric or null | Widget performance guardrail |
| `checkout_error` | boolean | Checkout guardrail outcome |
| `refunded` | boolean | Guardrail outcome |
| `attributed_revenue` | numeric | Total attributed revenue for the index cart; zero for non-converting carts and aggregated across attributed orders |

Assignment is the analysis basis: users remain in their assigned variant even if the widget fails to render, unless the pre-analysis plan explicitly defines a different estimand. Report exposure failures rather than quietly removing those users. Calculate AOV as total attributed revenue divided by attributed order count, and state the order denominator.

## Analysis and engineering standards

- **Reproducibility:** fixed random seed; explicit configuration; pinned or bounded dependencies; deterministic run instructions.
- **Statistical integrity:** define metrics and decision rules before outcomes; report effect sizes and confidence intervals; use a fixed horizon; distinguish confirmatory from exploratory analysis.
- **Data integrity:** validate inputs at boundaries; document exclusions; fail with actionable errors; retain an auditable sample flow.
- **Testing:** cover sample-size assumptions, assignment balance, deterministic simulation, metric denominators, edge cases (empty groups, all-zero/all-one outcomes), and invalid data.
- **Maintainability:** small single-purpose functions, clear names, type hints, formatting/lint checks, and no hidden notebook state in final outputs.
- **Communication:** show denominators and uncertainty; disclose simulation assumptions and limitations; make no causal claim beyond the stated randomized design.
- **Privacy:** use synthetic data only in this portfolio project. If real data is later introduced, require authorization, minimize fields, and follow organizational access/retention policy.

### What not to do

- Do not run a test first and decide the hypothesis, MDE, or stopping point afterward.
- Do not repeatedly check significance and stop when the result looks favorable.
- Do not use normalization or feature scaling for binary conversion; it has no role in this analysis.
- Do not add feature engineering unless it supports a pre-specified metric, eligibility rule, or clearly labeled exploratory question.
- Do not treat statistical significance as proof of business value or a non-significant result as proof of no effect.
- Do not generalize simulated results to real users.

## Definition of done

- [ ] Experiment brief, population, unit, metrics, exclusions, and attribution window are documented.
- [ ] Hypotheses, alpha, power, MDE, sample size, allocation, and stopping rule are recorded before outcome analysis.
- [ ] Synthetic data is reproducible and clearly identified as simulated.
- [ ] Data-quality checks and automated tests cover critical failure modes.
- [ ] Primary and guardrail analyses match the pre-analysis plan.
- [ ] Estimates, confidence intervals, p-values, sample flow, and limitations are reported.
- [ ] Product recommendation applies the pre-agreed statistical and practical criteria.
- [ ] Native Power BI or Tableau dashboard opens, matches the supplied reference layout, and matches the analysis outputs.
- [ ] Decision banner is conditional on the pre-specified criteria and guardrails; no result is hard-coded.
- [ ] Setup, run, and reproduction instructions work from a clean environment.

## Suggested first working session

1. Confirm the demo assumptions in the planning table.
2. Write the experiment brief and metric definitions.
3. Implement the sample-size calculation and save its output.
4. Review the design and stopping rule before generating outcomes.
5. Only then implement the seeded simulator and data checks.

The repository currently contains the project brief only. The phases above are the implementation roadmap; the scripts, data, tests, dashboard, and report are deliverables to build in sequence.
