# Source Boundary

This skill operates on user-provided local factor and label CSV files. It does not download market data, generate factor libraries, or call external research APIs.

## Data Boundary

- The bundled toy CSV files are synthetic data constructed for smoke tests and do not represent real securities, market data, or accounts.
- Inputs are local CSV files with unique (date, ticker) keys.
- Factor and label rows are inner-joined on the configured date and ticker columns.
- Labels and factor availability timing must be constructed and verified by the user.
- A missing available_date produces a point-in-time risk warning; the runtime cannot infer publication or availability lags.
- The fixed validation split is part of the selection workflow, not a locked trading holdout.

## Algorithm Boundary

- mRMR is a local, limited reproduction of regression F-statistic relevance plus absolute Pearson redundancy and greedy quotient selection.
- SAGE is a local, limited reproduction of empirical marginal imputation plus permutation-based global MSE contribution for a fixed LGBM or MLP model.
- Third-party mrmr and sage packages are optional parity-test dependencies, not runtime dependencies.
- The skill does not implement SHAP, permutation importance, native model importance, drop-one, retraining coalitions, backward elimination, or combined ranking scores.

## Research Boundary

- mRMR optimizes relevance and pairwise redundancy; it does not optimize retrained LGBM/MLP holdout performance.
- SAGE explains average fixed-model loss contribution; sorting SAGE values does not solve the best-size-K retraining problem.
- Correlated factors can share or destabilize Marginal-SAGE attribution.
- A completed SAGE run is not necessarily statistically converged; inspect sage_metadata.json and sage_std.
- Use a locked holdout outside this skill to compare the selected subset with full-factor and random-subset baselines.
- Outputs are quantitative research artifacts, not investment advice, trading signals, return guarantees, or production validation.
