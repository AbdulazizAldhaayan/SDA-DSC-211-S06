# قرار التجميع

KEEP SINGLE — Logistic

The best single model is Logistic Regression (mean AP 0.3917, fold SD 0.0298), and no ensemble passed the gate: the weighted average changed AP by -0.0022, the equal average by -0.0200 and the stack by -0.0085, none exceeding the fold SD. The stack also worsened ECE by +0.0122 (limit 0.01) and Brier by +0.0028, so the decision is KEEP SINGLE (Logistic) and the extra complexity did not earn its place.

Evidence comes from nested forward OOF over 2023Q1, 2023Q3 and 2024Q1 (2,155 OOF rows), with 90-day label maturity and customer separation, and the ensemble weights and stacker were learned on inner OOF only. It is development evidence from only three overlapping periods, not an independent final test, and the fold SD is descriptive, not a confidence interval.

الدليل: artifacts/ensemble_comparison.csv وday5_ensemble_gate.json. SD وصفي، وليس اختبار دلالة.
