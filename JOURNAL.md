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