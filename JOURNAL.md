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

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I resolved the Week 8 open question by splitting the design in two: the ground-truth FP/FN measurement runs over a labeled fixture set, and an optional live scan runs over stored DB reviews for a flag rate. From PLAN.md, the Understand/Map/Inputs sub-tasks are done, and the core scoring module (`safety/bias_audit.py`) is implemented — text extraction from a review's `sections` JSON, and a `ConfusionMatrix` computing precision/recall and FP/FN rates overall and per demographic signal. I also captured the repo's pre-existing baseline first: `make test-unit` already shows 52 failed / 31 errors on `main` (chunker SSL errors + 9 `test_bias_detector` failures that are the exact blind spots this audit measures), and `make check` is red repo-wide (ruff 182, black 52 files, mypy 5).

**Next steps:**
Build the labeled fixture set, wire up the `scripts/audit_bias.py` CLI (console + JSON report, optional `--scan-db`), and write unit tests for the scoring core. Then confirm my changes introduce no new failures against the baseline, and open the PR.

**Blockers:**
None. (Peer review happens in Slack; I'll request a draft-PR review there.)

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/583

**Branch:** `fix/72-bias-audit-report`

**What you built:**
An offline bias audit for the `BiasDetector`. `safety/bias_audit.py` scores the detector against a labeled sample set and reports false-positive / false-negative rates overall and by demographic signal (education, age, origin); `scripts/audit_bias.py` is a thin CLI that prints the report, writes `bias_audit_report.json`, and can optionally scan stored DB reviews for the detector's live flag rate (degrading gracefully when no database is reachable). Running it surfaces a real blind spot immediately — an overall 75% false-negative rate, 100% on education bias.

**Tests added or updated:**
`tests/unit/test_bias_audit.py` — 16 unit tests covering text extraction from `sections` JSON (including `None`, string, and malformed inputs), confusion-matrix tallying with divide-by-zero-safe rates, per-signal breakdown, empty-sample skipping, labeled-sample loading plus its validation errors, and report formatting. Added `tests/fixtures/bias_audit_samples.json` as the labeled ground-truth set.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

> Note on "passes": the codebase has documented pre-existing failures in both `make check` and `make test-unit` (see the PR description). My changes introduce **no new failures** — `make test-unit` goes from `52 failed / 345 passed / 31 errors` to `52 failed / 361 passed / 31 errors` (my 16 new tests, all green), and my new files are individually ruff-, black-, and mypy-clean (enforced by pre-commit on every commit).

**Draft PR feedback received from:** none
