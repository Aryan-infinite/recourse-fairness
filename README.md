# Does group fairness guarantee recourse fairness?

Independent research project: when a standard bias mitigation method improves a classifier's group fairness metrics, what happens to the *recourse gap*, that is, how far denied people in each group sit from a favorable decision?

- Datasets: Adult Census Income (protected attribute: sex) and COMPAS (race), through AI Fairness 360.
- Models: linear logistic regression, so the distance to the decision boundary is exact. Methods: baseline, Reweighing, and a covariance-penalized logistic regression (after Zafar et al., 2017).
- Statistics: Mann-Whitney U with rank-biserial effect size, over 30 random train/test splits, plus a run with the protected column removed from the model inputs, a penalty-strength sweep and a power calculation.

## Status

Analysis finished; paper being written. Not peer reviewed. This is a replication and extension of existing work on recourse fairness, not a first demonstration (see the research log for the papers it builds on).

## Main result (Adult, mean over 30 splits)

| Model | SPD | AOD | recourse gap r |
|---|---|---|---|
| Baseline | -0.186 | -0.091 | 0.426 |
| Reweighing | -0.087 | 0.068 | 0.205 |
| Covariance-penalized LR | -0.094 | 0.057 | -0.083 |

Reweighing and the penalty reach nearly the same group fairness but opposite-signed recourse gaps. With the protected column removed from the model inputs, Reweighing barely changes the gap (0.311 to 0.293). On COMPAS the gap is small (mean r 0.083 to 0.059) and the sample is too small to detect gaps below |r| of about 0.17. Limitations are listed in the research log and the paper draft.

## Files

- `recourse_fairness_starter.ipynb`: full pipeline, run top to bottom in Google Colab (Runtime > Run all).
- `multiseed_results.csv` (360 fits), `multiseed_summary.csv`, `mechanism_table.csv`, `lambda_sweep.csv` (140 fits): raw and summary results.
- `figure1_multiseed.png` (paper Figure 1), `figure3_lambda_sweep.png` (paper Figure 2).
- `requirements.txt`: library versions used in the Colab run.
- `research_log.md`: dated log of decisions, errors and corrections, newest first.
- `project_summary.md`: plain-language summary.

## Reproduce

Open the notebook in Google Colab and run all cells. The multi-split cells take about 10 minutes. Versions used: aif360 0.6.1, pandas 2.2.3, numpy 2.1.3, scikit-learn 1.6.1, scipy 1.16.3, matplotlib 3.10.0. An independent run with newer versions (numpy 2.5.4, scikit-learn 1.9.1, scipy 1.18.1) matched on SPD, AOD and accuracy exactly and on r to within 0.0002.

## AI assistance

The analysis code was written with the assistance of an AI model (Claude, Anthropic); the experiments were run by the repository owner in Google Colab. The research log records design decisions, errors and corrections as they happened.

## Built with

Python, scikit-learn, AIF360, scipy, matplotlib.
