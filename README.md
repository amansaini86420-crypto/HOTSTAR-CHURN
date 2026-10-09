# Disney+ Hotstar — Why Were Customers Leaving in 2023?
**Data Analyst portfolio project · SQL · Python · Statistics · Power BI**

> **The customer-level dataset used in this project is synthetic and created for analytical demonstration. The business context is based on publicly available information about Disney+ Hotstar's 2023 market situation.**
>
> Because the data is synthetic, the *patterns* below show how the analysis works — they are **not** real Hotstar facts. The effects were designed into the generator (e.g. Monthly plans churn more, inactivity precedes churn). What matters for a portfolio is the method, the care taken with causation, and the business translation.

---
## 1. Business problem
Why were customers leaving Hotstar in 2023, which types of customers were most likely to churn, and what could Hotstar have done to retain them? IPL loss is treated as **one hypothesis to test, not the assumed answer**.

## 2. 2023 business context (public information, not from the dataset)
| Fact | Detail (as reported by Disney earnings coverage: Business Today, TechCrunch, Inc42, Telecomtalk) |
|---|---|
| Subscriber trend | Paid subscribers peaked around 61.3 M in late 2022, then fell every quarter: −3.8 M (Oct–Dec 2022), −4.6 M to 52.9 M (Jan–Mar 2023), **−12.5 M to 40.4 M (Apr–Jun 2023, a 24% drop)**, and about 37.6 M by end-Sep 2023. |
| IPL digital rights | Viacom18 (JioCinema) won IPL digital rights for 2023-27 (~₹23,758 crore) and streamed the 2023 season **free**. Disney did not renew digital rights; it described its response as shifting from an IPL-centred product to a more balanced sports + entertainment offering. |
| Premium content | A Warner Bros. Discovery–Viacom18 deal moved HBO / Max / Warner titles to JioCinema, weakening Hotstar's premium English content. |
| Competition | JioCinema's free model drove app downloads and record concurrent IPL viewership. Hotstar answered with free-on-mobile cricket tournaments in late 2023 (e.g. ICC World Cup). |
| **2024 (separate event)** | The **Reliance–Disney transaction/JV was announced in 2024**, *after* the 2023 competitive developments. It is **not** a 2023 merger and is treated as a connected but separate part of the story. |

## 3. Dataset
~20,000 synthetic customers active at some point in 2023 (62% existing on 1-Jan-2023, 38% joined in 2023), 27 required columns. Churn = cancelled **or** did not renew in 2023 (`churn=1`); definition never changed. Two modelling notes: (a) usage columns are a baseline monthly profile; `IPL_watch_hours` measures IPL-tagged viewing *before* the 2023 rights loss (Hotstar did not stream IPL 2023 digitally), so it is an **IPL-dependency proxy**; (b) revenue = list price × 12 (−15% if a discount was used), an illustrative annual value in INR.

## 4. Data cleaning (full log: `reports/cleaning_log.csv`)
20,380 raw rows → **19,999 clean unique customers**. Nothing was removed without a reason; only 1 row was dropped for unrecoverable data.

| Problem | Treatment | Rows |
|---|---|---|
| Duplicate customers (exact + differently-cased) | Keep one record per `customer_id` (most complete) | 380 |
| Missing age / impossible age (0, 5, 120, 999) | Median (30) / set missing then median | 604 / 56 |
| Missing city | `Unknown` (not guessed) | 496 |
| Inconsistent names (gender, city, plan, content) | Mapped to one standard label | ~9,150 |
| Missing content category | Category of the customer's highest viewing hours (better than blind mode) | 395 |
| Text in numeric fields (₹299, "12.5 hrs", "14 months", Yes/No flags) | Converted to numeric / 1-0 | ~4,500 |
| Mixed date formats / impossible dates / future start dates | Parsed explicitly / set missing / set missing | 8,157 / 57 / 51 |
| Missing start date | Rebuilt from tenure & end date; 1 unrecoverable row dropped | 422 / 1 |
| End date before start date | Replaced with churn date (the reliable exit event) | 28 |
| Churn date missing | From end date (73); sampled from observed monthly pattern, flagged `churn_date_imputed` (3) | 76 |
| **Unrealistic watch hours** (>720 h/month or negative) | Rebuilt as sum of the 5 content-hour columns | 72 |
| Missing watch hours | Same rebuild | 329 |
| **Extreme but possible** (200-720 h) | **Investigated, kept** — consistent with component hours; analysed with medians/logs | 14 |
| IPL hours > sports hours | Capped at sports hours | 65 |
| Outlier sessions (2,500/month) / active days > 31 | Re-derived from watch time / sessions | 27 / 37 |

## 5. EDA headlines (`charts/01-03`)
Median customer: ~13 h/month, 17 sessions, last login 5 days ago. Watch time is right-skewed (mean 15.7 vs median 12.9), so medians and log scales are used. Customers are 63% male, mostly 25-34, concentrated in the 7 metros plus a large Tier-2/3 group. Sports is the largest primary-content group (24%).

## 6. Churn analysis

**Overall churn rate: 40.5% (8,093 of 19,999).**

| Cut | Result | Takeaway |
|---|---|---|
| **Plan** | Monthly **48.4%** · Quarterly 38.1% · Annual **23.2%** | Commitment length is the biggest structural lever |
| Age | <20: 43.2% … 45+: 36.2% | Mild; youngest churn most |
| Gender / city | Gender 38.5-40.6%; cities 39.5-42.7% | **No meaningful difference** — not where to focus |
| Tenure at 2023 start | 39-44% across bands; 2023 joiners 39.5% | Tenure *at entry* barely matters |
| Auto-renew | OFF 46.8% vs ON 35.6% | Renewal friction matters |
| Payment failures | 0: 38.7% · 1: 46.8% · 2+: 53.1% | Involuntary-style churn is real |
| Discount used | 37.9% vs 41.4% | Small effect, not a case for blanket discounts |

### Why is Monthly highest? (tested, not assumed)
Watch time is *identical* across plans (≈15.6-15.9 h), so low engagement is **not** why Monthly churns more. What differs: Monthly has the lowest auto-renew (50.7% vs 68.4% Annual), the most 2023 joiners (45.5% vs 20.8%), and a renewal decision every month. **Within every engagement level Monthly churns most** (Low-engagement: 64.6% Monthly vs 37.7% Annual). Price sensitivity cannot be separated from commitment in this data — flagged as a limitation.

### Engagement (Q4)
| Engagement | Churn |
|---|---|
| Low | **56.4%** |
| Medium | 39.5% |
| High | 25.5% |

Watch hours <5 h/month: 67.9% churn → 40+ h: 16.4%. Sessions ≤8/month: 59.7% → 40+: 21.4%. Churned customers: median 10.5 h vs 14.9 h retained; 18.0 vs 26.7 sessions. **Yes — lighter users were much more likely to leave.**

### Inactivity (Q5) — days since last login
| 0-7 | 8-14 | 15-30 | 31-60 | 60+ |
|---|---|---|---|---|
| 23.4% | 53.4% | 74.2% | 90.4% | 95.0% |

**Churn more than doubles once a customer is silent for just 8-14 days (53%) and passes 70% after 15 days.** An intervention trigger at ~7-10 days idle has time to work; at 30+ days most have gone.

### Content (Q6, Q8)
| Primary content | Churn | | Viewing style | Churn |
|---|---|---|---|---|
| **Sports** | **47.9%** | | Focused (1 category) | 41.4% |
| Movies | 40.6% | | Two categories | 40.4% |
| TV Shows | 39.8% | | Diversified (3+) | 38.3% |
| Originals | 37.2% | | | |
| Mixed | 35.9% | | | |
| **Kids** | **32.8%** | | | |

Sports-heavy customers (≥50% of hours on sports): **48.0%** vs 38.2% others. Diversified viewers churn modestly less (a 3-point gap).

## 7. IPL analysis (the 2023 context question) — Q7
| Group | Customers | Churn | Avg watch h |
|---|---|---|---|
| IPL viewers | 5,605 | 43.6% | 18.9 |
| Non-IPL viewers | 14,394 | 39.3% | 14.5 |
| IPL dependency **Low** (<10% of hours) | 14,833 | 38.2% | 15.2 |
| **Medium** (10-30%) | 2,407 | 42.6% | 16.8 |
| **High** (>30%) | 2,759 | **51.0%** | 17.4 |

**IPL dependency was associated with churn in the analysed dataset.** It does not prove IPL loss caused churn. Reasons for caution:
1. *Effect size is moderate and smaller than plan or inactivity:* the Spearman correlation with churn is only 0.05 (most customers have ~0 dependency), versus 0.49 for inactivity; in the logistic model IPL dependency is a +27% odds uplift per standard deviation versus 3.3× for inactivity and 4.1× for Monthly plan.
2. *Confounding:* 99.7% of high-dependency customers are existing (pre-2023) customers. I checked whether the gap survives controls — it does: High vs Low dependency is 64.6% vs 54.7% within Low-engagement, 32.9% vs 23.9% within High-engagement, and 63.0% vs 45.4% within Monthly plan. So IPL dependency adds risk *on top of* engagement and plan, but it is one factor among several, not the whole story.
3. *Reasons customers gave:* only 12.0% of stated churn reasons were "IPL no longer on Hotstar"; the most common were switching to JioCinema/free alternatives (22.8%) and price (22.1%). (Synthetic exit-survey field; stated reasons ≠ proven causes.)

## 8. Monthly churn (Q: when?) — data vs context
| Jan | Feb | Mar | Apr | May | Jun | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 2.5% | 2.8% | 4.4% | 7.5% | **10.3%** | 9.4% | 4.6% | 3.7% | 4.4% | 5.0% | 5.4% | 5.8% |

(Monthly rate = churned in month ÷ customers active at start of month. **Highest month: May 10.3%. Lowest month: Jan 2.5%.** After the mid-year peak the low point is Aug at 3.7%.)

**What the data shows:** churn accelerates from March, peaks in **May (10.3%)** with Apr-Jun accounting for 3,414 of 8,093 exits (42%), then drops sharply in July and stays at 4-6%. The share of exits from *high-IPL-dependency* customers is 25-28% in Mar-May, falling to 5-9% in Q4.
**What external context suggests:** the timing matches IPL 2023 being free on JioCinema (31 Mar-29 May) and HBO/Warner content moving off Hotstar; Disney's own reporting shows its largest quarterly subscriber drop in Apr-Jun 2023. The late-year recovery of new joiners (Oct-Nov peak) coincides with free-on-mobile cricket tournaments. **These are consistent timing patterns — not proof of causation in the synthetic data.**

## 9. Cohort analysis (`charts/14_cohort_heatmap.png`)
Cohorts are start months; retention is measured from entry to the 2023 window (blank = not yet observable). **6-month retention: 2021 starters 72.2%, 2022 starters 70.1%, 2023 joiners 60.4%.** Newer customers retained **worse**, especially those who joined in Apr-Jun 2023 (3-month retention 69-73% vs 89-94% for 2022 cohorts). Caution: the Jan-2023 12-month cell and Oct-2023 3-month cell rest on few customers.

## 10. Statistical tests (`tables/statistical_tests.csv`)
| Test | Question | p | Result | Business meaning |
|---|---|---|---|---|
| Chi-square | Sports-primary viewer vs churn | <0.0001 | Reject H0 | 47.9% vs 38.1% |
| Chi-square | IPL viewer vs churn | <0.0001 | Reject H0 | 43.6% vs 39.3% |
| Chi-square | Plan vs churn | <0.0001 | Reject H0 | 48.4 / 38.1 / 23.2% |
| Chi-square | Auto-renew vs churn | <0.0001 | Reject H0 | 46.8% vs 35.6% |
| Mann-Whitney | Watch hours churned vs retained | <0.0001 | Reject H0 | median 10.5 vs 14.9 h |
| Mann-Whitney | Days since last login | <0.0001 | Reject H0 | median 12 vs 3 days |
| Mann-Whitney | Complaints / IPL dependency | <0.0001 | Reject H0 | higher for churned |

With ~20,000 rows almost everything is "significant", so **effect size matters more than p-values**: inactivity, plan and engagement are large effects; IPL, complaints and payment failures are real but smaller.

## 11. Segmentation (rule-based, mutually exclusive, applied in this order)
At-Risk = low engagement + ≥15 days idle · Heavy = top-quartile hours *and* sessions · Sports = IPL dependency ≥30% or sports ≥50% of hours · Entertainment = movies+TV+originals ≥50% · Casual = the rest.

| Segment | Customers | Churn | Share of all churn | Avg h | Annual revenue lost |
|---|---|---|---|---|---|
| **At-Risk** | 2,501 (12.5%) | **79.9%** | 24.7% | 6.6 | ₹61.3 L |
| Sports | 3,527 (17.6%) | 44.5% | 19.4% | 15.3 | ₹49.1 L |
| Entertainment | 9,151 (45.8%) | 36.4% | **41.2%** | 12.3 | **₹104.5 L** |
| Casual | 1,301 (6.5%) | 33.3% | 5.4% | 13.5 | ₹13.5 L |
| Heavy | 3,519 (17.6%) | **21.6%** | 9.4% | 32.1 | ₹24.2 L |

The *rate* is worst for At-Risk, but the *largest revenue loss* is the large Entertainment base — a volume problem, not just a rate problem.

## 12. Churn prediction (Q11)
Logistic regression (primary) and a depth-4 decision tree. Features: watch hours, sessions, **tenure at start of 2023** (not tenure to churn date — that would leak the outcome), inactivity, plan, IPL dependency, complaints, payment failures. 75/25 stratified split.

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Logistic Regression (0.50) | 0.754 | 0.716 | 0.647 | 0.680 | **0.818** |
| Logistic Regression (0.35) | 0.727 | 0.630 | **0.793** | **0.702** | 0.818 |
| Decision Tree | 0.747 | 0.756 | 0.551 | 0.638 | 0.799 |
| *Baseline: predict "stay" for all* | 0.595 | – | 0 | – | 0.5 |

**Which metric?** Accuracy alone is weak (59.5% by guessing "stay"). For finding at-risk customers use **recall** (how many leavers we catch) balanced with **precision** (cost of wasted offers) — F1 and ROC-AUC summarise both. Lowering the threshold to 0.35 catches 79% of churners at the cost of precision; the right threshold depends on offer cost vs customer value. Strongest drivers (odds ratios): Monthly plan 4.1× vs Annual, inactivity +1 SD 3.3×, Quarterly 2.3×, IPL dependency 1.27×, complaints 1.20×, payment failures 1.15×. The tree puts 87% of importance on inactivity. Caveat: inactivity measured near exit is partly a symptom; in production it must be scored on a rolling window *before* renewal.

## 13. Churn-risk groups and revenue at risk (Q10)
Probabilities are 5-fold cross-validated (each customer scored by a model that never saw them).

| Risk group | Customers | % | Actual churn | Annual revenue | Expected rev. at risk | Revenue actually lost |
|---|---|---|---|---|---|---|
| Low (<30%) | 8,701 | 44% | 15.6% | ₹2.33 cr | ₹0.36 cr | ₹0.39 cr |
| Medium (30-60%) | 5,702 | 28% | 41.5% | ₹1.73 cr | ₹0.77 cr | ₹0.72 cr |
| **High (≥60%)** | 5,596 | 28% | **78.0%** | ₹1.81 cr | ₹1.39 cr | ₹1.42 cr |

**Revenue at risk** (project formula: churned × average annual revenue) = 8,093 × ₹2,938 = **₹2.38 crore**; actual annual value of churned subscriptions = ₹2.52 crore of ₹5.88 crore total (43%). **By plan: Monthly = 67.5% of revenue lost, Quarterly 27.3%, Annual 5.2%.** Higher-value customers churn more (49.7% vs 35.1%) because the highest-priced plan is Monthly — so Hotstar is losing *many* mid-value Monthly customers rather than a few whales. **Immediate save list:** 1,231 customers who *stayed* in 2023 but score High Risk, carrying ₹39 lakh annual revenue.

## 14. Experience factors (Q9)
Complaints: 0 → 38.4%, 1 → 45.2%, 2 → 51.4%, 3+ → **60.5%**. Buffering 5+ issues: 53.0% vs 37.3% with none. Small in population share but a clear dose-response pattern; fixable by operations.

## 15. Business recommendations (targeted — **no blanket discounts**)
| Customer group | Evidence | Action |
|---|---|---|
| **Low-engagement / idle 7-10 days** | 53% churn at 8-14 idle days, 90%+ beyond 30 | Automated win-back trigger at day 7-10: personalised "continue watching", new-episode alerts, push/WhatsApp; stop spending after day 30 |
| **Sports-heavy / high IPL dependency** | 48-51% churn; Monthly Sports is the top-risk plan×content cell (57%) | Retain with other live sport (international cricket, football, kabaddi), highlights/analysis, tournament-based passes; communicate the sports calendar early |
| **Monthly-plan users** | 48% churn, 67.5% of revenue lost, same watch time as Annual | Upsell to Quarterly/Annual at a *usage milestone*, not at signup; turn on auto-renew; offer plan-switch savings only to engaged users |
| **Customers near renewal** | auto-renew OFF = 46.8% churn; payment failures 53% | Renewal reminders 7-10 days ahead; UPI-autopay nudges; smart payment retry |
| **Complaint / buffering customers** | 60.5% churn at 3+ complaints | Priority support queue; resolve before the next renewal date; proactive CDN fixes in Tier-2/3 |
| **High-value / At-Risk list (model High Risk)** | 78% actual churn; ₹39 L still-active revenue | Weekly scored list to CRM; loyalty perks for engaged annual members; personalised *capped* offers only here |
| **New joiners (2023 cohorts)** | 6-month retention 60% vs 70-72% | Onboarding journey: first-week content picks, habit-forming reminders |
| **Content gaps** | HBO/Warner exit; Premium-content reasons 14.5% | Re-invest in originals and regional/premium titles; Kids & Mixed viewers are already the most loyal — protect them |

## 16. Answers to the 12 final business questions
1. **Overall churn:** 40.5% (8,093 / 19,999).
2. **Segments churning most:** At-Risk (79.9%), then Sports users (44.5%); Entertainment drives the most *volume* (41% of churners).
3. **Highest-churn plan:** Monthly (48.4%) vs Quarterly 38.1% vs Annual 23.2%. Driven by commitment/auto-renew and renewal frequency — not lower watch time (equal across plans).
4. **Low engagement vs churn:** Yes — 56.4% (Low) vs 25.5% (High); churned customers' median watch time is 10.5 h vs 14.9 h for retained (about 30% lower).
5. **Inactivity vs churn:** Yes, strongly: 23% (≤7 days) → 53% (8-14) → 74% (15-30) → 90% (31-60). The risk jump begins at 8-14 days.
6. **Content vs churn:** Yes — Sports 47.9% highest, Kids 32.8% lowest; diversified viewers 38.3% vs focused 41.4%.
7. **IPL dependency vs churn:** *IPL dependency was associated with churn in the analysed dataset* — High 51.0% vs Low 38.2%, persisting after controlling for engagement and plan, but modest relative to inactivity and plan, and not proof of causation.
8. **Sports-heavy behaviour:** Yes — higher churn (48.0% vs 38.2%); they watch more in total (17.2 vs 15.2 h) but in fewer, much longer sessions (16 sessions of ~67 min vs 25 of ~42 min — match-style viewing); and 60% of their exits fall in Mar-Jun vs 47% for other customers.
9. **Complaints vs churn:** Yes — 38.4% with none to 60.5% with 3+.
10. **Highest revenue at risk:** Monthly-plan customers (67.5% of revenue lost), the Entertainment segment (₹104.5 L lost), and the High-Risk group (₹1.4 cr).
11. **Can we predict churn?** Yes — ROC-AUC 0.82; at threshold 0.35 the model catches 79% of churners with 63% precision.
12. **Retention actions:** See §15.

## 17. Limitations (say these in interviews — it shows maturity)
- **Synthetic data:** effect sizes reflect how it was generated; real data would need confirming.
- **IPL proxy:** IPL hours are pre-loss baseline viewing, not 2023 viewing.
- **Association ≠ causation:** no experiment or control group; price vs commitment effects are confounded.
- **Snapshot inactivity** is partly a symptom of leaving; a production model needs time-windowed features.
- **Stated churn reasons** are self-reported-style and can't establish causes.
- Cohort cells with few customers are noisy.

## 18. Project structure & how to run
```
data/raw|clean/     raw (messy) and cleaned CSVs
python/             01_generate_data · 02_data_cleaning · 03_analysis · 04_sql_build
notebook/           hotstar_churn_analysis.ipynb (same code as cells + markdown; run from notebook/)
sql/                queries.sql (16 queries) · hotstar.db (SQLite) · query_results/
charts/             20 PNG charts       tables/  all result tables as CSV
powerbi/            data + theme + PowerBI_build_guide.md (DAX, 5 page layouts)
reports/            cleaning_log.csv
```
`pip install -r requirements.txt`, then run `python python/01_…py` through `04_…py` in order (seeded, fully reproducible).

## 19. Skills demonstrated
**SQL:** 6-table model, joins, CTEs, CASE, 8+ window functions (LAG, RANK, NTILE, running totals). **Python:** cleaning log, EDA, feature engineering, chi-square / Mann-Whitney / Welch tests, logistic regression vs decision tree, leakage awareness, cross-validated scoring. **Power BI:** KPI cards, DAX measures, 5-page design. **Business:** churn, retention, segmentation, revenue-at-risk, and disciplined causal language around the IPL/JioCinema story.
