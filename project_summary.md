# Project Summary

> Updated Oct 11 2026 with the 30-split results, the protected-column check and the penalty sweep (research_log.md, Entries 6 and 7). Earlier versions of this file reported one train/test split and overstated novelty.

## The Question

Companies and regulators check whether an AI model is fair using group fairness: do different groups get approved, or wrongly denied, at similar rates. A separate, less commonly checked idea is recourse fairness: among the people a model denies, is it similarly hard for different groups to reach a favorable outcome, measured as distance to the decision boundary.

Formally: **when a classifier is fixed with a standard bias mitigation method so that its group fairness metrics improve, what happens to the recourse gap between groups?**

## What We Used

**Datasets**, from different decision domains:
- Adult Census Income, predicting income above $50k, protected attribute sex (male privileged, 30,527 instances; female unprivileged, 14,695).
- COMPAS, ProPublica's two-year recidivism data, protected attribute race, binarized by AIF360 as Caucasian (2,100) versus all other recorded groups (4,067).

**Models and methods**, all linear so that the distance to the decision boundary has an exact formula:
- Baseline: logistic regression on standardized features.
- Reweighing: AIF360's pre-processing method.
- Covariance-penalized logistic regression: an in-processing method written from scratch (AIF360's Prejudice Remover exposes no model weights). It adds the squared covariance between the protected attribute and the raw score to the loss, after Zafar et al. (2017), with penalty strength 5. Its penalty is tied to the quantity the recourse measure uses, so Reweighing is the cleaner test.

**Metrics**: statistical parity difference and average odds difference for group fairness; margin |w.x + b| / ||w|| of every denied test person for recourse cost, compared between groups with a Mann-Whitney U test and the rank-biserial effect size r (r > 0: the unprivileged group is farther from the boundary).

**Design for uncertainty**: every experiment repeated over 30 random 80/20 splits; a second run with the protected column removed from the model inputs; a sweep of the penalty strength (0 to 50, 10 splits); a power calculation for COMPAS.

**Tools**: Python, scikit-learn, AIF360, scipy, matplotlib, in Google Colab. Library versions are in `requirements.txt`.

## What We Expected

No predicted direction. Related work (Ustun et al. on recourse; Gupta et al., Sharma et al. and von Kügelgen et al. on fairness of recourse; Haldar et al. on group-fairness methods worsening recourse fairness) already treats the two notions as distinct. This project is a replication and extension on linear models with exact margins, not a first demonstration.

## What We Got

Mean over 30 splits [middle 95%]; source `multiseed_results.csv`.

**Adult (sex).** Baseline r = 0.426 [0.403, 0.450]. Reweighing lowers it to 0.205 [0.183, 0.233]; the covariance-penalized model reverses its sign, -0.083 [-0.107, -0.056] (negative on 30/30 splits). The two fixes end with almost the same group fairness (SPD -0.087 vs -0.094; on every split they differ by at most 0.014 in SPD and 0.027 in AOD), yet their recourse gaps differ by 0.27 to 0.30 on every split.

**Protected column removed (Adult).** Baseline r falls to 0.311. Reweighing still cuts |SPD| from 0.180 to 0.095 but leaves r at 0.293; the penalty brings it to 0.040. This suggests that Reweighing's effect on the gap runs mostly through the model using the protected column (an inference from two separate runs).

**Penalty strength (Adult).** Mean r is negative for every penalty strength from 1 to 50; its size varies (r -0.028 at 1, -0.099 at 50).

**COMPAS (race).** The gap is small: mean r 0.083 (baseline), 0.059 (Reweighing), 0.059 (penalty), individually significant on 7, 5 and 2 of 30 splits. With about 120 vs 363 denied people the test has 80% power only for |r| of about 0.17, so these results can neither confirm nor exclude a gap of baseline size. The single split used earlier (seed 42) was one of the four with the lowest baseline r.

## What This Supports, and What It Does Not

**Supports (inside this design):** on Adult, SPD and AOD did not determine the recourse gap; the same group-fairness numbers came with gaps of opposite sign, and which way a mitigation moved the gap depended on whether the model could use the protected column.

**Does not support:** that this holds for other datasets, nonlinear models or other mitigation methods; that COMPAS has no gap; that the Adult reversal is "overcorrection" (the penalty does not target r = 0); any claim of being first. The penalty method is coupled to the recourse measure, the protected column is a model input in the primary analysis, the 30 splits overlap (their spread is not a confidence interval), and the recourse gap compares different sets of denied people under different methods.

## Where This Sits in the Literature

Gupta et al. (2019) and Sharma et al. (2020) define recourse fairness as equal average distance to the boundary across groups; von Kügelgen et al. (2022) argue fairness of recourse is complementary to fairness of prediction; Haldar et al. (2022) report that traditional fair classifiers can exacerbate recourse unfairness; Guldogan et al. (2023) and van Rosmalen et al. (2022) compare recourse-style metrics with demographic parity and statistical parity. A systematic literature search is still owed.
