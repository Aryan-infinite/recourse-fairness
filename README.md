# Does group fairness guarantee recourse fairness?

Independent research project testing whether standard bias-mitigation techniques (Reweighing, Prejudice Remover) that fix group fairness metrics also fix a stricter, less commonly checked notion of fairness: recourse cost, how far a denied individual sits from a favorable outcome.

Tested on two datasets from different decision domains: Adult Census Income (lending) and COMPAS (criminal justice risk assessment).

## Status
In progress. Baseline and Reweighing results computed for both datasets.

## Files
- `recourse_fairness_starter.ipynb` — full pipeline: data, baseline model, group fairness metrics, margin/recourse cost, statistical testing, mitigation techniques
- `research_log.md` — running log of decisions, results, corrections

## Built with
Python, scikit-learn, AIF360, scipy
