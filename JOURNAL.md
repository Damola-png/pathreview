# AI-201 Open Source Journal

## Repository
https://github.com/Damola-png/pathreview

## Week 7 - Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/163

**Issue title:** Review creation does not verify profile ownership

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
The `POST /reviews` flow passes the authenticated `user_id` into `create_review`, but the service currently creates a review from the provided `profile_id` without confirming that profile belongs to the same user. This creates an authorization gap where a user could potentially create reviews for another user's profile if they know the profile UUID. In contrast, `get_review` and `list_reviews` already scope access through `Profile.user_id`, so the write path is inconsistent with the read path. A successful fix should enforce ownership before review creation and add a test that covers the cross-user case.

**Is this right for me? checklist reasoning:**
- I can reproduce and explain the bug path across API route and service layers (`api/routes/reviews.py` -> `core/services/review_service.py`).
- Scope is bounded to one endpoint behavior plus test coverage, which is realistic for this module timeline.
- The acceptance criteria are clear: reject cross-user profile access on review creation and keep behavior consistent with existing ownership checks in read endpoints.
- Risk is moderate (authorization logic), but contained to review creation and should be validated with targeted tests.

**Branch name:** fix/163-review-profile-ownership

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger





## Week 8 — Reproduction & solution planning

**Reproduction commit link:**
[https://github.com/Damola-png/pathreview/commit/13aa66480b8fd02bed5ed4e4a26f43efc2a7306e]

**Reproduction summary:**
I reproduced Issue#163 by adding a targeted unit test where the authenticated user attempts to create a review for a profile owned by another user. The test failed because `create_review()` still called `db.add()` and created a pending review instead of rejecting the unauthorized request, confirming that profile ownership is not currently checked during review creation.

**PLAN.md link:**
https://github.com/Damola-png/pathreview/blob/fix/163-review-profile-ownership/PLAN.md



**Blockers or open questions:**
I still need to confirm the expected error response when a profile does not exist or belongs to another user. The existing read endpoints use ownership filtering and return a 404 when the requested resource cannot be accessed, so I will determine whether review creation should follow the same behavior. There are also unrelated existing `AsyncMock` failures in some review service tests, so I will use targeted tests for Issue #163 while implementing and validating the fix.



## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented Issue #163 by adding profile ownership validation inside `create_review()` so review creation only succeeds when the requested profile belongs to the authenticated user. Unauthorized or non-existent profiles now return a `404 Profile not found` before any review is added or committed. Updated `tests/unit/test_review_service.py` with ownership-aware setup and verification for both allowed and rejected creation paths.

**Next steps:**
Open a draft PR, request peer/mentor feedback in Slack, and finalize the PR description with baseline vs post-change check/test results. Re-run project checks before marking the PR ready and update this journal with the submitted PR link.

**Blockers:**
`make` is not available in this Windows PowerShell environment, so I ran equivalent commands via `.venv\\Scripts` (`ruff`, `black --check`, `mypy`, and `pytest`) to validate behavior.

---

### Check-in 2 (end of week)

**PR link:** [https://github.com/ascherj/pathreview/pull/699]

**PR status:** Draft (work in progress), awaiting reviewer approval and maintainer workflow approval

**Branch:** fix/163-review-profile-ownership

**What you built:**
Added an authorization guard in the review creation service that checks `(profile_id, user_id)` ownership before constructing a `Review`. This closes the cross-user creation gap by rejecting inaccessible profiles with a 404 response and preventing any database writes for unauthorized requests.

**PR description summary (submitted):**
- Summary: Fixes Issue #163 by verifying requested profile ownership before review creation.
- Issue closure: Closes #163.
- Behavior changes: rejects non-owner access, rejects missing profiles, and blocks unauthorized `db.add()`/`db.commit()` writes.
- Test updates: unit tests updated for authorized and unauthorized review creation paths; ownership regression test passes.
- Scope note: any remaining unrelated repository-wide failures were pre-existing and not introduced by this PR.

**Tests added or updated:**
Updated `tests/unit/test_review_service.py` to cover the ownership rejection path (`test_create_review_rejects_profile_owned_by_another_user`) and to keep the review service suite stable with SQLAlchemy execute-result mocks. Focused run: `pytest tests/unit/test_review_service.py -v -m unit` (20 passed).

**Self-review confirmation:** [x] `make check` equivalent executed in PowerShell (`ruff`, `black --check`, `mypy`) and results documented below  [x] `make test-unit` equivalent executed in PowerShell (`pytest tests/unit -v -m unit`) and results documented below

**Draft PR feedback received from:** none

**PR readiness notes:**
- Review required: at least 1 approving review from a maintainer/write-access reviewer.
- CI gate: 1 workflow run is awaiting maintainer approval.
- Merge state: PR remains draft until ready-for-review is submitted.

**Pre-existing failures observed:**
Project-wide `check` and `test-unit` equivalents still report existing unrelated baseline failures in other modules (for example: safety, parsing, scoring, and detector tests). For this issue scope, the focused review service run passes: `pytest tests/unit/test_review_service.py -v -m unit` (20 passed). This change did not introduce any new failures in the touched review service code path.
