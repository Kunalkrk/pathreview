## Solution plan

**Issue:** [Add a `has_tests` boolean to the repo analysis output (#50)](https://github.com/ascherj/pathreview/issues/50)

### Understand

Root cause: `TechDetector` (the agent tool that turns a repo's file list into
`primary_language`/`all_languages`/`frameworks`) never had test-detection logic added.
It isn't a regression — the field simply doesn't exist yet.

- Expected: analysis output includes `has_tests: bool`, true when the repo has a
  `tests/`/`test/` directory, a `pytest.ini`, or `test_*.py` files.
- Actual: `has_tests` is absent from the output entirely, confirmed by the failing
  test added in the reproduction commit
  ([2914f8f](https://github.com/Kunalkrk/pathreview/commit/2914f8ffb18b380721c2edc256d9f0c967b426e0)).

### Map

- `agent/tools/tech_detector.py` — primary file to change. `_detect_tech()` builds
  the result dict from a pre-filtered file list (`_should_skip_file` already strips
  vendor/build noise); `has_tests` detection slots in here as a new helper method.
- `tests/unit/test_tech_detector.py` — already has `test_has_tests_detection`
  (currently failing); needs a negative-case test added alongside it.
- `ingestion/parsers/repo_analyzer.py` — has its own unrelated `_detect_tests()` for
  the ingestion/RAG pipeline. Reference only, not touched.

### Plan

1. Add `_detect_has_tests(filtered_files)` to `TechDetector`, checking each path for
   a `tests/`/`test/` directory segment, a `pytest.ini` filename, or a `test_*.py`
   filename match (case-insensitive).
2. Call it from `_detect_tech()` and add `has_tests` to the returned dict — including
   the early-return branch in `execute()` for when no files are provided.
3. Confirm the existing `test_has_tests_detection` test now passes; add a negative
   test (no test markers → `has_tests is False`).
4. Run `make check && make test-unit` to confirm lint/type/test pass.

### Inputs & outputs

- Input: same as today — `input_data["files"]`, a `list[str]` of repo paths. No new
  input required.
- Output: one new key, `has_tests: bool`, added to the existing result dict on every
  code path (including the empty-files early return). No existing fields change.

### Risks & unknowns

- Naive substring matching would false-positive on names like `latest.py` or
  `contest.js` — matching needs to check path segments/filename patterns, not a bare
  `"test" in path` check.
- Open question: the issue only lists Python markers (`pytest.ini`, `test_*.py`).
  `ingestion/parsers/repo_analyzer.py` also treats `__tests__/` and `spec/` (JS/TS
  conventions) as test markers. Unclear if `has_tests` here should match scope, or
  stay strictly to the issue's stated criteria.
- Unrelated pre-existing bug: `_should_skip_file` doesn't match root-level
  `node_modules/`/`build/` paths (only nested ones) — shouldn't affect `tests/`
  detection directly, but worth double-checking once implemented.
- `TechDetector` isn't yet wired into the live review pipeline (`review_service.py`
  currently hardcodes `_run_agent_orchestration`'s output), so this fix will only be
  verifiable via unit tests, not by exercising the running app end-to-end.

### Edge cases

- Empty file list → `has_tests: False`, no crash (matches current behavior for other
  fields).
- Mixed case paths (`TESTS/`, `Test_Main.PY`) → still detected, consistent with the
  existing case-insensitive extension matching.
- Test-like filenames outside a real test directory/pattern (`latest.py`) → must NOT
  be flagged as `has_tests`.
- Test files nested inside excluded vendor/build paths → already filtered out via
  `_should_skip_file` before detection runs.
