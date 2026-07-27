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
