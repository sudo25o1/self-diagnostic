# Install Script Friction — Gap Analysis

## What Happened (Scott's Install, 2026-02-17)

Three separate failures during the install attempt on Scott's Mac mini:

**Failure 1: raw.githubusercontent.com 404**
The initial `curl` attempt returned 404. Root cause: the repo had just been made public and GitHub's CDN had not propagated the change yet. Not a code issue — a timing issue. But it created confusion and required re-explanation.

**Failure 2: npm ERESOLVE peer dependency conflict**
`npm install` failed with a peer dependency conflict between `oxlint@^1.43.0` and `oxlint-tsgolint@0.11.5`. The fix was adding `--legacy-peer-deps` to the install command. This was a real code issue — patched immediately in `install.sh` and pushed. But it required a second install attempt.

**Failure 3: "destination path already exists and is not an empty directory"**
The user had a prior `~/bernard` directory from the failed first attempt. The script didn't handle this case — it tried to `git clone` into an existing directory and failed with a fatal error. Had to manually `rm -rf ~/bernard` before retrying.

## What the Script Should Handle Better

1. **Stale directory detection:** Before cloning, check if `~/bernard` exists. If it does and has a `.git` dir, do a `git pull` instead. If it exists but has no `.git`, warn the user and exit cleanly rather than failing with a git fatal.

2. **CDN propagation:** Nothing to fix in code, but the README or install instructions should note that if the `curl` returns 404 immediately after a repo goes public, wait 60 seconds and retry.

3. **Dependency conflicts:** `--legacy-peer-deps` is already in place after the patch. This specific issue is resolved.

4. **Better error messaging:** The original script had no user-facing error context — just raw npm error output. Adding human-readable messages ("Dependency conflict detected — retrying with --legacy-peer-deps") would reduce confusion.

## Current State

The script works after the `--legacy-peer-deps` patch. The stale directory case is still unhandled — a second install attempt on a machine that already has `~/bernard` will fail unless the user manually removes it first.

## Priority

Low-Medium. Affects onboarding friction for new instances (Scott, future users). Not blocking anything current, but worth cleaning up before Bernard is distributed more widely.
