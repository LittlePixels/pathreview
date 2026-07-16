# Development Journal — PathReview

My running record of progress throughout Module 3. A new section is added each week.

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/72

**Issue title:** Add a bias audit report that runs over a sample of stored reviews

**Tier:** [ ] Tier 1  [ ] Tier 2  [x] Tier 3

**Problem summary:**
PathReview's safety layer includes a bias detector (`safety/bias_detector.py`) that
flags potentially biased language in generated reviews, but there is currently no way
to measure how well it actually performs across real data. Nothing today samples stored
reviews and evaluates the detector's accuracy, so blind spots — reviews it wrongly flags
(false positives) or biased content it misses (false negatives) — go unmeasured. This
issue asks for an offline audit script (`scripts/audit_bias.py`) that samples ~100 stored
reviews, runs them through the bias detector with detailed logging, and produces a report
of false positive/negative rates broken down by demographic signal. A successful fix gives
maintainers concrete, reproducible evidence of the detector's real-world behavior so its
thresholds can be tuned with data instead of guesswork.

**Branch name:** fix/72-bias-audit-report

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
