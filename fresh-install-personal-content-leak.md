# Gap: Personal Content Leaking Into Fresh Installs

**Date Identified:** 2026-02-18
**Status:** Fixed (commit `1ce2bd0c5`)
**Severity:** High — privacy risk + broken onboarding for new users

---

## What Was Wrong

The `bernard/` workspace directory in the public repo contained two problems:

**1. Personal RELATIONAL.md baked in**
`bernard/RELATIONAL.md` contained the original developer's (Derek's) personal relational data:
- Named "Bernard & Derek" explicitly
- Contained Derek's specific communication markers ("really important", "I want to be clear")
- Contained dated growth markers from actual conversations (2026-01-22, 2026-01-27, etc.)
- Contained observed friction patterns from real sessions

Any new user cloning the repo would inherit this context in their workspace. Bernard would effectively believe it was already in a relationship with "Derek" — wrong user, wrong patterns, wrong history.

**2. Missing BOOTSTRAP.md**
`bernard/BOOTSTRAP.md` was not present in the repo's workspace directory. BOOTSTRAP.md is the trigger for first-contact onboarding — without it, Bernard has no signal to ask who the new user is or how they want to work together.

New users would get a Bernard that has someone else's relationship context loaded and no mechanism to replace it.

---

## What Was Fixed

1. Replaced `bernard/RELATIONAL.md` with the clean template from `bernard/templates/RELATIONAL.md`
   - Generic "Bernard & USER" header
   - All pattern fields empty / marked for population through conversation
   - No personal content

2. Added `bernard/BOOTSTRAP.md` from `docs/reference/templates/BOOTSTRAP.md`
   - Triggers onboarding sequence on first session
   - Guides Bernard to ask one question at a time, discover through conversation
   - Routes learned context to USER.md, RELATIONAL.md, SOUL.md appropriately
   - Self-deletes when onboarding is complete

`bernard/USER.md` was already clean (empty template). `bernard/SOUL.md` was also clean (generic Bernard identity, no personal data).

---

## Root Cause

The workspace directory (`bernard/`) was seeded during early development from the developer's live workspace — personal content was committed alongside the structural files. No review process caught that RELATIONAL.md was personal before the repo went public.

---

## Prevention

Before sharing the repo publicly or with new users:
- Audit `bernard/` for any file referencing real names, dates, or conversation history
- `RELATIONAL.md` specifically should always be the template version in the repo — personal versions live only in `~/.openclaw/workspace/` (outside the repo)
- `BOOTSTRAP.md` must be present to ensure clean onboarding
- Consider adding a CI check or pre-push hook that warns if RELATIONAL.md contains non-template content (e.g., grep for specific name patterns)

---

## Files Changed

- `bernard/RELATIONAL.md` — replaced with clean template
- `bernard/BOOTSTRAP.md` — added (was missing)

**Commit:** `1ce2bd0c5` on `sudo25o1/bernard`
