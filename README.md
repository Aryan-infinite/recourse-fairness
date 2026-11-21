# Does Group Fairness Guarantee Recourse Fairness? An Empirical Test Across Two Decision Domains

**Authors:** Aryan Bhusal, Bishal Neupane, Rita Neupane  
**Preprint:** [Preprints.org (DOI: 10.20944/preprints202611.0608.v1)](https://doi.org/10.20944/preprints202611.0608.v1)

## Overview

When a standard bias mitigation method improves a classifier's group fairness metrics, what happens to the *recourse gap*—how far denied individuals in each group sit from a favorable decision?

- **Datasets:** Adult Census Income (protected attribute: `sex`) and COMPAS (protected attribute: `race`), loaded via AI Fairness 360.
- **Models:** Linear logistic regression, allowing exact closed-form Euclidean distance to the decision boundary. Methods evaluated: Baseline, Reweighing (Kamiran & Calders, 2012), and covariance-penalized logistic regression (after Zafar et al., 2017).
- **Evaluation:** Mann-Whitney $U$ tests with rank-biserial effect size $r$ across 30 random 80/20 train/test splits, supplemented by a protected-column sensitivity analysis, a regularization penalty sweep, a statistical power check, and shared-denied population diagnostics.

## Status

Preprint posted on Preprints.org (`10.20944/preprints202611.0608.v1`). Not yet peer reviewed. This study replicates and extends existing work on recourse fairness across decision domains (see `research_log.md` and the Related Work section of the paper).

## Main Result (Adult, Mean Over 30 Splits)

| Model | SPD | AOD | Recourse Gap ($r$) |
|---|---|---|---|
| Baseline | -0.186 | -0.091 | 0.426 |
| Reweighing | -0.087 | 0.068 | 0.205 |
| Covariance-penalized LR | -0.094 | 0.057 | -0.083 |

Reweighing and covariance-penalized LR achieve nearly identical group fairness metrics (SPD and AOD) but produce opposite-signed recourse gaps. When the protected column is removed from model inputs, Reweighing leaves the gap virtually unchanged (0.311 to 0.293). On COMPAS, the gap is small (mean $r$ between 0.059 and 0.083) and sample sizes lack statistical power to detect gaps below $|r| \approx 0.17$.

## Repository Structure

- `recourse_fairness_analysis.ipynb`: Complete, executed analysis pipeline (run top to bottom in Google Colab via `Runtime > Run all`).
- `paper/Bhusal_et_al_2026_Recourse_Fairness.pdf`: Full preprint manuscript.
- `multiseed_results.csv` (360 fits), `multiseed_summary.csv`, `mechanism_table.csv`, `lambda_sweep.csv` (140 fits), `results_summary.csv`: Raw and aggregated experimental outputs.
- `diagnostics/`: Shared-denied population overlap, OLS margin-gap fits, and denied-distance quantile tables (`diag_*.csv`).
- `figure1_multiseed.png`, `figure3_lambda_sweep.png`: Publication figures.
- `requirements.txt`: Exact Python library versions used in the primary Colab environment.
- `research_log.md`: Chronological log of experimental iterations, audits, and statistical checks.

## Reproducibility

Open `recourse_fairness_analysis.ipynb` in Google Colab and execute all cells (`Runtime > Run all`). The multi-split and diagnostic cells take roughly 15 minutes. Primary environment: `aif360==0.6.1`, `pandas==2.2.3`, `numpy==2.1.3`, `scikit-learn==1.6.1`, `scipy==1.16.3`, `matplotlib==3.10.0`. Independent replication in a newer environment (`numpy==2.5.4`, `scikit-learn==1.9.1`, `scipy==1.18.1`) matched exactly on SPD, AOD, and accuracy, and within `0.0002` on rank-biserial $r$.

## Acknowledgments

We used Claude (Anthropic) for assistance with code structuring, numerical verification, and manuscript editing; all experimental runs, statistical analyses, and final claims were executed and verified by the authors.

## Citation

```bibtex
@article{bhusal2026recourse,
  title={Does Group Fairness Guarantee Recourse Fairness? An Empirical Test Across Two Decision Domains},
  author={Bhusal, Aryan and Neupane, Bishal and Neupane, Rita},
  journal={Preprints},
  year={2026},
  doi={10.20944/preprints202611.0608.v1}
}
