## Plan: FastAPI Backend Test Suite in Dedicated Directory

Add a pytest-based backend test suite under a top-level tests directory to validate all current FastAPI endpoints and key error paths, while isolating mutable in-memory activity state between tests. This approach keeps tests deterministic, aligns with repository automation checks, and establishes a scalable test structure.

**Steps**
1. Phase 1 - Test scaffolding and dependencies
2. Add `pytest` to `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` so local/dev/CI environments can run the test suite.
3. Confirm `pytest.ini` remains compatible (`pythonpath = .`) so tests can import from `src.app` without extra path hacks.
4. Create a new top-level `/workspaces/skills-getting-started-with-github-copilot/tests/` directory with initial test modules and optional package marker only if needed by tooling.
5. Phase 2 - Shared test setup and isolation (*depends on Phase 1*)
6. Add a shared fixture in `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` that provides a FastAPI `TestClient` for `src.app:app`.
7. Add a state-reset fixture that snapshots and restores `activities` before/after each test (or monkeypatches a deep-copied structure) so signup/unregister mutations do not leak across tests.
8. Phase 3 - Endpoint coverage in split modules (*depends on Phase 2*)
9. Add `tests/test_root.py` for `GET /` redirect behavior (status and target static path).
10. Add `tests/test_activities.py` for `GET /activities` happy path (status, object shape, presence of known activities).
11. Add `tests/test_signup.py` for `POST /activities/{activity_name}/signup`:
12. Success case appends new participant and returns expected message.
13. 404 for unknown activity.
14. 400 for duplicate signup.
15. Add `tests/test_unregister.py` for `DELETE /activities/{activity_name}/signup`:
16. Success case removes existing participant and returns expected message.
17. 404 for unknown activity.
18. 404 when student is not enrolled.
19. Use AAA structure in each test body for readability and maintainability.
20. Phase 4 - Run and harden (*depends on Phase 3*)
21. Run `pytest` from repo root and fix any import/path/state flakiness.
22. Ensure tests are order-independent by running the suite multiple times or with randomized ordering if available.
23. Keep assertions focused on public API behavior to avoid brittle internals.

**Relevant files**
- `/workspaces/skills-getting-started-with-github-copilot/src/app.py` - Reuse the existing `app` object and `activities` in-memory data model; verify route contracts from `root`, `get_activities`, `signup_for_activity`, and `unregister_from_activity`.
- `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` - Add `pytest` dependency.
- `/workspaces/skills-getting-started-with-github-copilot/pytest.ini` - Keep/verify root import configuration.
- `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` - Shared `TestClient` and state-isolation fixtures.
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_root.py` - Redirect behavior tests.
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_activities.py` - Activity listing tests.
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_signup.py` - Signup success/error tests.
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_unregister.py` - Unregister success/error tests.

**Verification**
1. Install dependencies: `pip install -r requirements.txt`.
2. Execute suite from repo root: `pytest -q`.
3. Confirm all tests pass repeatedly (at least two consecutive runs) to validate state isolation.
4. Confirm repository exercise checks are satisfied: `requirements.txt` contains `pytest` and a `tests` directory exists.

**Decisions**
- Include scope: backend API tests only for current endpoints in `src/app.py`.
- Exclude scope: frontend static asset tests (`src/static/*`) and performance/load testing.
- Assumption: keep current in-memory persistence model unchanged; tests adapt via fixtures rather than refactoring runtime storage.

**Further Considerations**
1. Test granularity decision: use split endpoint modules from day one (`test_root.py`, `test_activities.py`, `test_signup.py`, `test_unregister.py`) as requested.
2. Redirect assertion strictness: Option A assert redirect location only (recommended), Option B also assert exact status code behavior for trailing-slash variants.
3. Future-proofing: optionally add `pytest-cov` later once baseline tests are stable.