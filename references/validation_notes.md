# Validation Notes And Boundaries

- Verify factor and label timing before running. `available_date` is not required, so the runtime cannot prove point-in-time correctness.
- Use an embargo at least as long as the overlapping label horizon.
- mRMR uses only training rows and ignores validation rows.
- SAGE trains the fixed model on training rows and evaluates global contribution on sampled validation rows.
- SAGE uses the local empirical marginal imputer; correlated factors may receive unstable or shared attribution.
- The Top-K choice is supplied by `selection_count`; neither method proves that K is optimal.
- Keep a locked holdout outside the skill. Do not use test results to revise K, model parameters, or SAGE settings.
- The skill does not generate factors, tune hyperparameters, optimize portfolios, estimate transaction costs, run a full backtest, or issue investment advice.
