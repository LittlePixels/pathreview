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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/LittlePixels/pathreview/commit/f047af241a5cb6732cdcf0c8e54216a82b7fc2cb

**Reproduction summary:**
Since #72 is a missing-capability issue, I reproduced the gap rather than a crash: I ran the existing `BiasDetector` over stored-review text via a throwaway script ([scripts/repro_bias_audit.py](scripts/repro_bias_audit.py)), feeding it benign snippets from the seeded reviews and a few crafted biased phrasings. The benign text was correctly unflagged (0/3 false positives), but all three crafted biased strings went undetected (3/3 false negatives) — and crucially there is no audit script, no FP/FN report, and `detect_bias` is never even called in the pipeline ([review_service.py:366](core/services/review_service.py#L366)), so these blind spots are completely unmeasured today.

**PLAN.md link:** [PLAN.md](PLAN.md)

**Walkthrough video (recommended):** [not recorded]

**Blockers or open questions:**
The `Profile` model has no demographic fields, so the issue's requested "breakdown by demographic signal" can't come from profile attributes — I plan to derive signals from a labeled fixture set of review text instead. Open question for Week 9: whether the maintainers expect the audit to run against the live DB, a fixture set, or both.
