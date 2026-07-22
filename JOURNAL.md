# Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/50

**Issue title:** Add a `has_tests` boolean to the repo analysis output

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The agent's repo analysis pipeline currently has no way to tell whether a scanned
repository includes any testing infrastructure, so that signal is missing from
the output profile it builds for a candidate's project. A successful fix adds
detection logic that looks for common markers of tests — a `tests/` or `test/`
directory, a `pytest.ini` file, or files matching `test_*.py` — and exposes the
result as a new `has_tests` boolean field on the analysis output. This affects
the agent/tooling side of the codebase (likely a new or extended tool under
`agent/tools/`), and once done it lets downstream scoring and profile-building
logic account for whether a project demonstrates testing practices.

**Branch name:** feat/50-has-tests-boolean

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

**"Is this right for me?" checklist reasoning:**
- Understanding: clear — add `has_tests` to analysis output based on `tests/`, `pytest.ini`, or `test_*.py`.
- Scope note: issue names `agent/tools/repo_analyzer.py`, which doesn't exist. Real fit is `agent/tools/tech_detector.py` (already scans a `files` list the same way).
- Tier: confirmed Tier 1 via GitHub labels; matches "first contribution" fit.
- Codebase: read `tech_detector.py` and its 26-test suite — new logic/test slot in with the existing pattern.
- Time/blockers: no assignees/comments/blockers on the issue; 2–4 hr estimate fits the Week 8–9 window.
