# Does Group Fairness Guarantee Recourse Fairness?

Empirical study of group-fairness metrics and geometric recourse disparities on Adult and COMPAS. This repository accompanies an **unreviewed preprint**; geometric distance to a linear decision boundary is a proxy, not feasible actionable recourse.

## Manuscript

See `manuscript/recourse_fairness_preprint_numbered.pdf` for the current reading copy and the matching DOCX for edits. Author approval and final bibliographic checks are required before public release.

## Data and analyses

- `recourse_fairness_analysis.ipynb`: cleaned analysis notebook (GitHub authentication/push cells removed; this cleaned copy has not been independently rerun).
- `multiseed_results.csv`, `multiseed_summary.csv`, `mechanism_table.csv`, `lambda_sweep.csv`: original experiment outputs already in the repository.
- `diagnostics/`: six supplementary CSVs for common-denied, margin-gap and quantile analyses.
- `research_log.md`: dated research decisions and corrections, including Entry 9.
- `requirements.txt`: recorded dependency versions.

Adult mean over 30 overlapping splits: Reweighing SPD −0.087, AOD +0.068, recourse rank-biserial r +0.205; covariance penalty SPD −0.094, AOD +0.057, r −0.083. The contrast persists on the shared denied population. COMPAS is less conclusive because of smaller denied-group samples.

The manuscript and research log describe model input choices, penalty selection, the geometric proxy's limitations, and sensitivity analyses. These splits are not independent replications.

## Reproduction

Review `requirements.txt` and run the notebook in a fresh Google Colab environment. Check the output files against the committed CSVs. The cleaned notebook omits GitHub authentication and upload commands. Do not commit tokens, secrets, or private data.

## AI assistance

Generative AI assisted code development, debugging, analytical checks, literature organization, and manuscript preparation. The authors are responsible for validating and approving the final work.
