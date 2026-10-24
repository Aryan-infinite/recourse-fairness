# Research Log
### Does Group Fairness Guarantee Recourse Fairness? An Empirical Test Across Two Decision Domains

Newest entry at the top. Each entry: what happened, what was decided, what's still open.

---

## Entry 8 - Diagnostics before the Discussion: shared denied population, margin-gap fit, denied-distance quantiles

**What ran.** One new cell appended after the multi-split and penalty-sweep cells of the Colab notebook (208 s). Adult only, primary setup (sex is a model input), the same 30 splits and λ = 5 as Entry 7. No existing result changed: the four original CSVs were hashed before and after the run and are unchanged. New outputs are in `diagnostics/` (list at the end). Brackets are min-max over the 30 splits in A and B2 (the splits overlap, so these are ranges, not confidence intervals) and 95% bootstrap intervals in B1; Entry 7's brackets are middle-95% ranges.

**Checks.**
- 160 refits (90 primary: 3 models x 30 splits; 70 sweep: 7 penalty strengths x 10 splits) reproduce the rows of the original CSVs: counts exact, floats within 1e-4.
- Seed-42 recourse distances equal the output of the notebook's own `recourse_costs()` for all three models.
- 0 non-converged fits.
- An independent re-run in a separate environment (numpy 2.5.4, scikit-learn 1.9.1, scipy 1.18.1) printed identical output line by line, apart from timing and the file path.

**A. People denied by both Reweighing and the covariance-penalized LR** (mean over 30 splits [min, max]; r > 0: unprivileged group farther from the boundary; each model's own distance is used).

| | Reweighing | Covariance-penalized LR |
|---|---|---|
| r, own denied people (existing result) | 0.205 [0.181, 0.246] | -0.083 [-0.112, -0.050] |
| r, shared denied people (new) | 0.205 [0.180, 0.246] | -0.082 [-0.113, -0.050] |
| Denied by this model only, privileged / unprivileged | 68 / 26 | 72 / 48 |
| Mean distance on the shared set, privileged / unprivileged | 0.962 / 1.143 | 1.310 / 1.106 |

Shared set: 4,710 privileged [4,629, 4,780] and 2,528 unprivileged [2,458, 2,608] people per split. It holds 98.7% [98.5, 99.0] of Reweighing's denied people and 98.4% [98.0, 98.7] of the penalty's. Reweighing minus penalty: 0.288 [0.274, 0.301] on own sets, 0.287 [0.274, 0.299] on the shared set, positive on 30/30 splits. On the shared set r > 0 for Reweighing and r < 0 for the penalty on 30/30 splits, and Reweighing's Mann-Whitney p < .05 on 30/30.

Reading: the between-method difference does not come from which people are denied. With about 98.5% overlap the two rows could not have differed much, so this rules out the composition explanation; it is not evidence about mechanism.

**B1. Mean-margin gap vs mean recourse-cost gap** (gap = privileged minus unprivileged; margin gap over all test people, cost gap over denied people; one observation = one model fit on one split; OLS; the bootstrap resamples whole splits, B = 2000).

| Observations | n | cost gap = | R² | Margin gap at cost parity [95% bootstrap] |
|---|---|---|---|---|
| Penalty sweep, 7 strengths x 10 splits | 70 | 0.232 - 0.909 x margin gap | 0.998 | 0.255 [0.246, 0.264] |
| Sweep, strengths >= 1 only | 60 | 0.227 - 0.787 x margin gap | 0.940 | 0.289 [0.272, 0.306] |
| Primary, 3 models x 30 splits | 90 | 0.192 - 0.928 x margin gap | 0.965 | 0.207 [0.201, 0.213] |

Limits on what the association supports:
- Both quantities come from the same fitted score distribution, so the relation is partly mechanical. It is an association, not a test of mechanism or cause.
- There are only 7 (sweep) or 3 (primary) distinct model conditions; the rows within a condition are overlapping splits of the same data, so n = 70 or 90 overstates the independent information. Dropping the unpenalized fits (strengths >= 1 only) changes the slope from -0.909 to -0.787 and R² from 0.998 to 0.940.
- The cost-parity point depends on which observations are used (0.207, 0.255, 0.289) and the intervals do not overlap, so no single value should be quoted. The bootstrap covers split-to-split variability only, not the choice of model form.
- Adult only, linear models, sex only.

**B2. Distance to the boundary among denied people** (quantiles within each split, then mean over 30 splits). Gap = privileged minus unprivileged, > 0: privileged farther; in parentheses, the number of splits (of 30) with gap > 0.

| | Baseline | Reweighing | Covariance-penalized LR |
|---|---|---|---|
| mean | -0.491 (0) | -0.183 (0) | +0.206 (30) |
| q10 | -0.391 (0) | -0.213 (0) | -0.126 (0) |
| q25 | -0.604 (0) | -0.306 (0) | -0.160 (0) |
| q50 | -0.631 (0) | -0.310 (0) | +0.083 (30) |
| q75 | -0.478 (0) | -0.114 (0) | +0.528 (30) |
| q90 | -0.341 (0) | +0.049 (28) | +0.725 (30) |
| q95 | -0.309 (0) | +0.094 (30) | +0.825 (30) |
| q99 | -0.303 (0) | +0.101 (25) | +0.951 (30) |

Reading (descriptive only): the baseline gap is negative at every quantile. After the penalty, privileged denied people are closer to the boundary in the bottom quartile and farther from the median up, with the largest differences in the upper tail; the two halves have opposite signs, which is consistent with a small |r| next to a mean gap of +0.206. Reweighing's gap is negative through q75 and small at the top; its q99 sign is not stable (25/30) and rests on about 25 unprivileged people per split.

**For the write-up.**
- The penalty's mean cost gap is +0.206 (0.2063 unrounded); a draft figure of +0.207 came from subtracting rounded table entries.
- "Small overshoot" holds for the rank statistic (r = -0.083) but not for the upper tail: privileged denied people are 0.725 to 0.951 farther at q90 to q99, against 0.303 to 0.341 in the other direction at baseline.
- The shared-population check and the mechanism check, listed as future work in the draft, are done; report them in the text.

**Files** (`diagnostics/`): `diag_common_denied_by_split.csv`, `diag_common_denied_summary.csv`, `diag_mechanism_observations.csv`, `diag_mechanism_fit.csv`, `diag_denied_quantiles_by_split.csv`, `diag_denied_quantiles_summary.csv`.

**Still open.**
- Write the Discussion with these results.
- Add Friedler et al. (arXiv 1802.04422) and Karimi et al. (arXiv 2002.06278) to the references; confirm venues.
- Rebuild the paper tables from the `diag_*.csv` files.

---

## Entry 7 - Colab results: 30 splits, protected-column run, penalty sweep

**What ran.** Cells 1 and 2 of the notebook in Colab. Outputs pushed in commit f3d099e: `multiseed_results.csv` (360 fits), `multiseed_summary.csv`, `mechanism_table.csv`, `lambda_sweep.csv` (140 fits), `figure1_multiseed.png`, `figure3_lambda_sweep.png`, `requirements.txt`. Colab versions: aif360 0.6.1, pandas 2.2.3, numpy 2.1.3, scikit-learn 1.6.1, scipy 1.16.3, matplotlib 3.10.0. All 500 fits converged. Numbers were read from the pushed CSVs (the repo is public), not from the cells' printed output.

**Checks on the run.**
- Seed-42 primary rows reproduce the single-split table of Entries 3-5 (largest difference 5e-5 in SPD, AOD or r).
- Independent re-run in the sandbox (numpy 2.5.4, scikit-learn 1.9.1, scipy 1.18.1; Adult rebuilt from GitHub mirrors, Entry 6 finding 8): SPD, AOD, accuracy and denied counts identical on all 360 fits; r within 0.0002. Penalty sweep: r within 0.004 and SPD within 0.0013 (L-BFGS-B stops at slightly different points across scipy versions).
- COMPAS power recomputed from the Colab CSV with the notebook's own code (120 vs 363 denied): 12%, 38%, 70%, 92%, 98% at true r = 0.05, 0.10, 0.15, 0.20, 0.25; 80% at r about 0.17.
- Shapiro-Wilk on the seed-42 recourse costs: normality rejected for all 12 group-by-method sets (all p < 1e-10), which backs the choice of Mann-Whitney.

**Primary analysis (the audited attribute is a model input).** Mean over 30 splits [middle 95%]. r > 0: unprivileged group farther from the boundary.

| Data | Model | SPD | AOD | r | p<.05 | r<0 |
|---|---|---|---|---|---|---|
| Adult | Baseline | -0.186 [-0.198, -0.176] | -0.091 [-0.117, -0.061] | 0.426 [0.403, 0.450] | 30/30 | 0/30 |
| Adult | Reweighing | -0.087 [-0.102, -0.076] | 0.068 [0.043, 0.089] | 0.205 [0.183, 0.233] | 30/30 | 0/30 |
| Adult | Covariance-penalized LR | -0.094 [-0.109, -0.082] | 0.057 [0.035, 0.080] | -0.083 [-0.107, -0.056] | 30/30 | 30/30 |
| COMPAS | Baseline | -0.159 [-0.218, -0.101] | -0.139 [-0.207, -0.080] | 0.083 [-0.007, 0.163] | 7/30 | 2/30 |
| COMPAS | Reweighing | -0.010 [-0.059, 0.067] | 0.014 [-0.038, 0.089] | 0.059 [-0.023, 0.158] | 5/30 | 3/30 |
| COMPAS | Covariance-penalized LR | -0.078 [-0.133, -0.011] | -0.056 [-0.113, 0.014] | 0.059 [-0.006, 0.149] | 2/30 | 2/30 |

**Protected column removed from the model inputs** (the penalty and the weights still use it; the other protected column stays in).

| Data | Model | SPD | AOD | r | p<.05 |
|---|---|---|---|---|---|
| Adult | Baseline | -0.180 [-0.192, -0.169] | -0.078 [-0.104, -0.046] | 0.311 [0.289, 0.333] | 30/30 |
| Adult | Reweighing | -0.095 [-0.109, -0.085] | 0.057 [0.033, 0.075] | 0.293 [0.271, 0.320] | 30/30 |
| Adult | Covariance-penalized LR | -0.048 [-0.065, -0.035] | 0.115 [0.092, 0.140] | 0.040 [0.011, 0.066] | 27/30 |
| COMPAS | Baseline | -0.154 [-0.203, -0.096] | -0.134 [-0.181, -0.079] | 0.080 [-0.002, 0.170] | 4/30 |
| COMPAS | Reweighing | -0.147 [-0.195, -0.089] | -0.127 [-0.174, -0.072] | 0.072 [-0.007, 0.149] | 2/30 |
| COMPAS | Covariance-penalized LR | -0.148 [-0.196, -0.088] | -0.128 [-0.176, -0.070] | 0.073 [-0.004, 0.166] | 2/30 |

**Where the mitigation acts** (primary, mean over splits). Weight share: the audited attribute's coordinate over ||w||, + favors the privileged group. Margins: + is toward the favorable decision, over all test instances. Cost: |margin| among the denied.

| Data | Model | Weight share | Margin priv | Margin unpriv | SD priv | SD unpriv | Cost priv | Cost unpriv |
|---|---|---|---|---|---|---|---|---|
| Adult | Baseline | 0.119 | -0.448 | -1.214 | 1.317 | 0.977 | 0.895 | 1.386 |
| Adult | Reweighing | -0.088 | -0.560 | -0.887 | 1.313 | 0.978 | 0.949 | 1.132 |
| Adult | Covariance-penalized LR | -0.355 | -0.836 | -0.868 | 1.426 | 0.911 | 1.294 | 1.087 |
| COMPAS | Baseline | 0.004 | 0.283 | 0.023 | 0.908 | 0.912 | 0.563 | 0.599 |
| COMPAS | Reweighing | -0.093 | 0.145 | 0.091 | 0.905 | 0.909 | 0.541 | 0.607 |
| COMPAS | Covariance-penalized LR | -0.050 | 0.184 | 0.063 | 0.911 | 0.847 | 0.543 | 0.555 |

**Penalty sweep** (first 10 splits, primary setup; mean).

| lambda | Adult SPD | Adult r | COMPAS SPD | COMPAS r |
|---|---|---|---|---|
| 0 | -0.184 | 0.427 | -0.163 | 0.092 |
| 1 | -0.106 | -0.028 | -0.089 | 0.074 |
| 2 | -0.097 | -0.061 | -0.082 | 0.074 |
| 5 | -0.092 | -0.085 | -0.078 | 0.075 |
| 10 | -0.090 | -0.093 | -0.076 | 0.074 |
| 20 | -0.088 | -0.097 | -0.076 | 0.074 |
| 50 | -0.088 | -0.099 | -0.075 | 0.074 |

**Paired, same splits, Adult primary (Reweighing minus penalty).** Max |dSPD| 0.014, max |dAOD| 0.027, dr from 0.274 to 0.301 (positive on all 30 splits). Rule of thumb (|SPD| and |AOD| < 0.1, not a standard): Reweighing passes on 28/30 splits and still has |r| >= 0.1 on 28 of those; the penalty passes on 25/30 and has |r| >= 0.1 on 3. On COMPAS the two differ in SPD by up to 0.091 and show no consistent difference in r (dr from -0.063 to 0.049).

**What the data support, inside this design.**
1. On Adult, SPD and AOD did not determine the recourse gap: two mitigations with almost the same group fairness ended at r = 0.205 and -0.083.
2. Reweighing's effect on r mostly disappears when the model cannot use the protected column. With the column in, Reweighing moves r by 0.221 (0.426 -> 0.205); with it removed, Reweighing still cuts |SPD| from 0.180 to 0.095 but r only moves 0.311 -> 0.293. This compares two separate runs, so it is an inference, not an intervention on one model. The baseline model uses the column: its sex weight share is 0.119, Reweighing moves it to -0.088, the penalty to -0.355.
3. The penalty also acts through other features: with the column removed it still brings r to 0.040.
4. After the penalty the groups' mean margins are nearly equal (-0.836 vs -0.868) but their spreads differ (1.426 vs 0.911), and denied men are farther from the boundary than denied women (1.294 vs 1.087). Hypothesis, not tested: the sign of r among the denied is set by each group's lower tail, which a penalty on the mean score does not control.
5. Mean r on Adult is negative for every lambda >= 1 (the smallest nonzero value tested; where it crosses zero between 0 and 1 is not known), so the sign change is not an artifact of lambda = 5. Its size depends on lambda (-0.028 at 1, -0.099 at 50).
6. COMPAS: mean r is 0.083, 0.059, 0.059; individually significant on 7, 5, 2 of 30 splits. At this sample size 80% power needs |r| about 0.17, so these results can neither confirm nor exclude gaps of baseline size. The seed-42 split used earlier had baseline r = 0.012, lower than 26 of the 30 splits.

**Claims withdrawn or changed.**
- Entry 5, "overcorrection risk appears tied to the size of the original gap, not to the mitigation method": not supported. On Adult the method matters (item 2 vs 3), and COMPAS cannot test it (item 6). Withdrawn.
- Entry 3, "domain-dependent divergence": the datasets differ in measured r, but COMPAS is underpowered and domain is confounded with baseline gap size, group sizes and sample size. Do not write "domain-dependent".
- "Overshoot" and "overcorrection": r < 0 describes the sign of the gap. The penalty does not target r = 0, so "reverses the sign of the gap" is the accurate wording.
- Entry 3, "COMPAS: no recourse gap at baseline (p = 0.854)": that was one split, among the four with the lowest r. Replace with the power-aware statement in item 6.

**Edits to the paper draft.** `Recourse_Fairness_Sample_Paper_Template.docx` rebuilt from the CSVs with no hand-typed numbers: Tables 1-4 now show 30-split means and middle-95% ranges; new Table 5 (column removed) and Table 6 (mechanism); Figures 1 and 2 embedded; new Sections 3.7, 4.4, 4.5; Limitations rewritten; Related Work rewritten around the papers of Entry 6 finding 7. Three sentences from the old draft were removed because they could not be verified: that Ustun et al. showed recourse differing by gender (their abstract does not say it), that to our knowledge COMPAS recourse had not been examined, and that COMPAS accuracy was similar across groups (Angwin et al. sentence). Interpretation, Discussion, Conclusion and the Abstract's finding are still gray prompts in the draft.

**Citations checked on Oct 11** (title, authors, arXiv ID and abstract-level content read from ar5iv/arXiv pages): Sharma et al. 2010.06113; von Kügelgen et al. 2010.06529; Guldogan et al. 2210.06732; van Rosmalen et al. 2211.11512; Zafar et al. 1507.05259; Haldar et al. 2208.14175 (CIKM 2022 confirmed by a search result); Artelt & Hammer 2211.14858. Not verified: venues and years for von Kügelgen (AAAI 2022), Guldogan (ICLR 2023) and Zafar (AISTATS 2017) are from memory and need a check on the publisher pages; FaiR-N's Table 3 and the full texts of RAGUEL and von Kügelgen were not read; for Ustun et al. only the abstract was available.

**Still open.**
- Read FaiR-N (Table 3), RAGUEL and von Kügelgen in full, then fix any Related Work sentence a full read contradicts.
- Systematic literature sweep (Google Scholar and Semantic Scholar, "cited by" on Gupta 2019 and von Kügelgen 2022); check whether FaiR-N or RAGUEL already evaluate Reweighing.
- Write the interpretation paragraphs, Discussion, Conclusion and the Abstract's finding; the AI-assistance paragraph must describe what was actually done.
- Confirm reference venues; save the notebook copy to GitHub (File > Save a copy in GitHub) so the repo notebook contains the new cells; commit README, log, summary and paper draft once.
- Post as a preprint (OSF Preprints; arXiv would need an endorsement) labelled not peer reviewed, only after the items above.

---

## Entry 6 — Re-audit before write-up (checks designed here; Colab numbers go in Entry 7)

Reviewed every result so far against the Phase 7 rule (effect sizes and uncertainty). Nine findings, ordered by how much they change the paper.

**1. One split, no uncertainty.** Every number in Entries 2–5 is from a single 80/20 split (seed 42). Plan: same code over 30 random splits (seed 42 plus 29 others); report mean and middle-95% range of r per method, share of splits with r < 0, share with p < .05. Entry 5's line "overcorrection risk appears tied to the size of the original gap, not to the mitigation method" was an inference from one split, two datasets and two methods at one penalty strength. It is a hypothesis, not a finding. Withdrawn until Entry 7.

**2. The protected attribute is a model input in every run so far.** Source says: AIF360's `AdultDataset` drops only `fnlwgt`; `CompasDataset` lists `sex` and `race` in `features_to_keep`; `StandardDataset` always keeps the `protected_attribute_names` columns. Code does: confirmed via `feature_names` and fitted weights (Adult: sex has the 4th-largest weight, about 13% of the weight vector's length; COMPAS: race about 0.2%). Consequence: a mitigation can act by giving the model a group-specific score offset, i.e. the model treats groups differently at decision time. Separately, the recourse distance includes a coordinate nobody can change. That second point does not affect r: removing an unchangeable coordinate from the movable set rescales every margin in a model by one constant, and Mann–Whitney only uses ranks. The first point could matter a lot, so a sensitivity run removes the protected column from the model inputs (the penalty and the weights still see it). Primary analysis is unchanged.

**3. The fairness-regularized LR is not independent of the recourse measure.** Its penalty is the squared covariance between the protected attribute and the raw score, which is Zafar et al.'s (2017) decision-boundary covariance proxy in penalty form. The margin is that same raw score divided by the weight norm. The penalty pulls the groups' mean scores together, which moves margins directly. Reweighing is the cleaner test of the research question. The paper should call this method "covariance-penalized logistic regression, after Zafar et al. (2017)", not a stand-in for Prejudice Remover.

**4. Penalty strength was picked on the evaluation split.** λ = 5 came from a 4-value sweep (0 / 1 / 5 / 20) on COMPAS, judged by SPD and accuracy on the same held-out split used for reporting, then reused untouched on Adult. It was not tuned toward a recourse number, but it has to be disclosed. A 7-value sweep (0–50) on 10 splits is added so the reader can see whether the Adult sign change depends on λ = 5.

**5. COMPAS "no gap" is a low-power null.** Mann–Whitney power for 110 vs 366 denied (the seed-42 baseline group sizes), alpha .05, normal-shift simulation: true r = 0.10 → about 36%, 0.15 → about 67%, 0.20 → about 89%. Reaching 80% needs |r| of roughly 0.18. So "not significant" means "no gap larger than about 0.18 could have been detected", not "no gap".

**6. Six primary tests, no multiplicity correction.** Bonferroni alpha = 0.05 / 6 = 0.0083 changes no conclusion (Adult p ≤ 0.00039 in all three rows; COMPAS p ≥ 0.15 in all three). Goes in Limitations.

**7. Prior work (abstract-level check of the papers below plus about a dozen searches; not a systematic review). The novelty claim in the paper draft is wrong and has to change.** Checked against the arXiv texts:
- Gupta et al. 2019: recourse = distance to the decision boundary; a regularizer that equalizes it across groups.
- Sharma, Gee, Paydarfar & Ghosh (FaiR-N, arXiv 2010.06113): Definition 1 calls a classifier recourse-fair when the average distance to the boundary of people given a negative outcome is equal across groups, which is nearly our metric. Their abstract argues recourse disparity and error-rate fairness are related; they report a comparison of existing mitigation methods on a burden gap (Adult, German Credit, MEPS). Table 3 has not been read; only the definition and the abstract-level thesis were confirmed, so read it before citing.
- Haldar, Cunningham & Ferhatosmanoglu (RAGUEL, CIKM 2022): evaluates group-fairness post-processing methods (demographic parity, equalized-odds variants) on recourse fairness (German Credit, Default of Credit Card Clients) and reports they can worsen it. Reweighing is not evaluated there.
- von Kügelgen et al. 2022: "fairness of recourse is complementary to fairness of prediction"; Adult case study.
- Guldogan et al. 2023 (Equal Improvability): relates the notion to demographic parity in theory; German Credit and ACSIncome-CA.
- van Rosmalen et al. 2022: counterfactual "burden" vs statistical parity can disagree (two synthetic sets plus Taiwan credit); no mitigation.
- Artelt & Hammer: group-fair counterfactual explanations; no standard mitigation evaluated. Zafar et al. 2017: covariance proxy.
Conclusion: "group fairness does not imply recourse fairness" is an established claim, and the empirical question of how mitigation methods move recourse gaps has been looked at already (post-processing in RAGUEL; Prejudice Remover and others in FaiR-N, on neural/other models). What this project adds, honestly stated: an exact closed-form margin for linear models, rank-based tests with effect sizes, 30-split variability, Reweighing and a covariance penalty on Adult and COMPAS, a protected-column sensitivity run, and a penalty-strength sweep. It is a replication/extension, not a first. The paper must say "we did not find" or describe the gap relative to these specific papers, never "to our knowledge no one has". A real literature sweep (Google Scholar, Semantic Scholar, "cited by" on Gupta 2019 and von Kügelgen 2022) is still owed.

**8. Sandbox replication.** The analysis sandbox cannot reach UCI. Adult was rebuilt from two GitHub mirrors (`shap/data/adult.data` and `jbrownlee/Datasets/adult-all.csv`): they agree on all 32,561 training rows, the test block starts with the documented first test row (25, Private, 226802, 11th, ...) and has 16,281 rows, and 3,620 rows contain "?" (the number AIF360 documents as removed). With that data, the sandbox reproduced all six seed-42 rows of the Colab table to four decimals. This verifies the Colab numbers; it does not replace them. Every number in the paper comes from the Colab run.

**9. Bug found while testing the multi-split code.** AIF360 memoizes every metric method with a cache keyed on the metric object itself, so each `ClassificationMetric` pins its dataset copies in memory forever (about 35 MB on Adult). Hundreds of them exhaust RAM and kill the session. The multi-split code computes SPD, AOD and accuracy with a few lines of numpy using the same definitions; it matched AIF360 exactly on 12 model/split combinations and matches the six seed-42 rows.

**Next:** run the multi-split cell and the penalty-sweep cell in Colab, commit the outputs, record the numbers in Entry 7.

---

## Entry 5 — Fairness-regularized LR results, both datasets (all planned results now collected)

**COMPAS:** SPD -0.064, AOD -0.038, Acc 0.657. Recourse: priv mean 0.558, unpriv mean 0.564, p=0.199, r=0.076. Matches sandbox reference exactly. Same story as Reweighing on this dataset: Question 1 improves, Question 2 stays not significant, consistent with a baseline that never had a real gap to fix.

**Adult:** SPD -0.083, AOD +0.066 (sign flip from baseline's -0.096, sharper flip than Reweighing's), Acc 0.839. Recourse: privileged mean 1.290, unprivileged mean 1.130 — **direction reversed from baseline**. At baseline and after Reweighing, unprivileged had the harder recourse; after this method, privileged does. Still statistically significant (p=0.000389) but effect size collapsed to r=-0.050 (baseline r=0.454, Reweighing r=0.246) — small in practical terms despite significance, expected given n>7,000 makes even tiny true differences detectable. Sign of r flipped along with the group means, consistent with a real direction change, not noise.

**Emerging cross-method pattern, worth building the paper's Discussion around:** on Adult, where a large real gap existed at baseline, both mitigation methods pushed past parity to the opposite side (AOD sign flip both times, recourse direction flip with this method). On COMPAS, where no real baseline gap existed, neither method overshot anything, they just correctly left an already-small effect alone. Overcorrection risk appears tied to the size of the original gap, not to the mitigation method itself.

**Limitation to add:** `fair_lambda` fixed at 5 across both datasets for comparability, not tuned per dataset — may partly explain why Adult (large starting gap) overshot while COMPAS (small starting gap) didn't. Worth naming explicitly rather than leaving implicit.

**Status:** all three methods (baseline, Reweighing, fairness-regularized LR) complete for both datasets. No further data collection planned. Next is Week 5 — synthesis and write-up, not new code.

---

## Entry 4 — Second mitigation built: fairness-regularized LR replaces Prejudice Remover

AIF360's `PrejudiceRemover` confirmed unusable (Entry 3 follow-up): wraps a 2012 subprocess, no accessible model weights, would have blocked the margin calculation entirely. Built a transparent replacement instead — logistic regression with a covariance-based fairness penalty added to the training loss (correlation between protected attribute and raw decision score, pushed toward zero), optimized with `scipy.optimize.minimize` (L-BFGS-B, analytic gradient — numerical gradient was hitting function-evaluation limits before converging, analytic fixed it). Same family as Prejudice Remover (in-processing, training-time regularization), still linear, margin formula applies unchanged. Wrapped in a small class exposing `coef_`, `intercept_`, `decision_function()` so it plugs into the existing `recourse_costs` function with no changes needed there.

`fair_lambda` (penalty strength) swept 0/1/5/20 before picking 5 — real tradeoff confirmed (SPD -0.174→-0.064 on COMPAS as lambda increases, accuracy stays ~flat), 5 chosen as a reasonable middle ground, not searched for a specific target result.

**COMPAS result:** SPD -0.064, AOD -0.038, Acc 0.657. Recourse: priv mean 0.558, unpriv mean 0.564, p=0.199, r=0.076 — not significant, consistent with both baseline (p=0.854) and Reweighing (p=0.154) on this dataset. Third method, same story: Question 1 improves, Question 2 was never broken here.

**Next step:** run Adult through the same cell (code handles both datasets already), then Week 5 — lay all three methods next to each other across both datasets and write up the actual pattern.

---

## Entry 3 — Reweighing results, both datasets (Phase 6, first mitigation complete)

**Correction first:** an earlier run produced a corrupted Adult recourse result (p = 0.000000 with unusual magnitude) because `run_baseline` had reverted to an unscaled version while the margin cell still referenced a scaler object — a leftover from stale Colab kernel state, not a real finding. Fixed at the source (`run_baseline` now scales and returns its scaler, matching `run_reweighing`) and reran clean from a fresh runtime. COMPAS baseline reproduced Entry 2's numbers exactly (SPD −0.170, AOD −0.150, p = 0.854), confirming this run is trustworthy.

**Baseline, confirmed clean:**
- Adult (sex) — SPD −0.182, AOD −0.096, Acc 0.847. Recourse: privileged mean 0.902 (n=4487), unprivileged mean 1.457 (n=2686). Mann-Whitney p < 0.000001, r = 0.454 — large, highly significant gap, unprivileged group faces harder recourse.
- COMPAS (race) — SPD −0.170, AOD −0.150, Acc 0.661. Recourse: privileged mean 0.652 (n=110), unprivileged mean 0.603 (n=366). p = 0.854, r = 0.012 — no meaningful gap.

**After Reweighing:**
- Adult — SPD −0.078 (roughly halved), AOD flipped sign (−0.096 → +0.075, worth a Discussion note, not just a number), Acc 0.840. Recourse: privileged mean 0.955, unprivileged mean 1.201. p < 0.000001, r = 0.246 — gap shrank (r roughly halved) but remains highly significant. Reweighing helped, did not fix.
- COMPAS — SPD 0.0015 (near-perfect parity), AOD 0.027, Acc 0.649. Recourse: privileged mean 0.575, unprivileged mean 0.625. p = 0.154, r = 0.081 — still not significant, consistent with baseline.

**The actual pattern so far:** the two datasets are diverging, not agreeing, which is itself a real result. On COMPAS there was never a significant recourse gap to fix, and Reweighing's near-perfect Question 1 fix left Question 2 exactly where it started. On Adult, a real, large recourse gap existed at baseline, and Reweighing partially closed it (effect size roughly halved) without eliminating it, even while substantially improving Question 1. Domain-dependent divergence, not a uniform answer either way.

**Next step:** second mitigation method (Phase 6 continued). Choice of implementation still open — AIF360's `PrejudiceRemover` wraps an inaccessible legacy subprocess with no usable weights; recommended fix is a transparent, self-implemented fairness-regularized logistic regression instead. Awaiting confirmation before building.

---

## Entry 2 — Baseline finalized with scaling (Phase 3 complete)

Ran the notebook. Unscaled Adult model failed to converge (real evidence, not just the theoretical concern from Entry 1) — resolves that entry's open question in favor of scaling both datasets now, rather than deferring to Week 2.

**Final baseline (scaled), reference numbers for Table 1's Baseline row:**

Adult (protected attribute: sex) — SPD −0.182, AOD −0.096, Accuracy 0.847
COMPAS (protected attribute: race) — SPD −0.170, AOD −0.150, Accuracy 0.661

Scaled vs. unscaled results are close for both (expected — scaling mainly fixes optimizer reliability, not what a converged logistic regression expresses). COMPAS accuracy dropped ~1 point after scaling (0.672 → 0.661); within normal variation for a linear refit, noted rather than treated as a problem.

`adult_scaler` / `compas_scaler` saved alongside each fitted model — required for Week 2, new points need the identical transform before the margin calculation is meaningful.

**Next step:** margin function (Phase 5).

---

## Entry 1 — Project start, notebook audit (Phase 1)

**Status:** Starter notebook (`recourse_fairness_starter.ipynb`) uploaded to Colab, not yet executed.

**Implemented:**
- Environment: pandas pinned to `2.1.4` (required — aif360 is incompatible with pandas ≥3.0's default string dtype handling; confirmed by direct testing, not assumed)
- Data acquisition: COMPAS via ProPublica's GitHub-hosted CSV, download tested and confirmed working. Adult via the three UCI files aif360's source code itself requires; URLs are correct per aif360, not yet confirmed reachable from a live Colab run
- Data loading: `AdultDataset()`, `CompasDataset()` via `aif360.datasets`
- Baseline model: L2-regularized logistic regression, 80/20 split, `seed=42`, one per dataset, no feature scaling applied
- Group fairness metrics: statistical parity difference, average odds difference, accuracy, via `aif360.metrics.ClassificationMetric`
- Verified reference output (COMPAS only, from an actual run): SPD −0.176, AOD −0.154, accuracy 0.672

**Not yet implemented:** margin/recourse function, Reweighing, Prejudice Remover, Mann-Whitney U testing, results tables/figures, repo structure and version control.

**Open decision (unresolved):** baseline model has no feature scaling. Irrelevant to the group fairness metrics already computed (label-based). Will matter for the margin function (Week 2): unscaled, high-magnitude features could dominate the Euclidean distance-to-boundary independent of actual importance. Options: (a) add `StandardScaler` now and rerun baseline, or (b) keep current numbers as the documented pre-scaling reference and scale starting with the margin work. Not yet decided.

**Infrastructure note:** No GitHub connector active. Checked directly against the connector directory, nothing installed or connected. Repo commits to be made by running `git` from within Colab, authenticated with the project owner's own credentials, once repo structure is set up.

**Next step:** run the notebook, record real Adult and COMPAS baseline numbers below in Entry 2, resolve the scaling decision.
