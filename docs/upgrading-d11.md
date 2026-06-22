# Drupal 11 upgrade notes

Captured from a CI-driven exploration (the `2.x` branch).
**Verdict: D11 is achievable.** This is the verified work-list, not a finished upgrade.

Current release (`1.x`) targets Drupal 10.6. The `2.x` branch is the D11 work-in-progress.

## Branch & version policy

`1.x` and `2.x` are maintained **in parallel** — `2.x` does not replace `1.x`:

- **`1.x` → Drupal 10** (`^10.6`). Stays stable; existing projects depend on it. Security + bugfixes.
- **`2.x` → Drupal 11**. A clean D11 major (D11-only deps), released as `v2.x.y`.

Consumers pin `skilldlabs/druxxy:^1.0` (D10) or `^2.0` (D11). Both branches run the install CI.

## How this was tested

The install smoke test (`.github/workflows/test.yml`) was pointed at `drupal/core-recommended:^11`
and used two things specific to consuming not-yet-D11 modules:

- **`mglaman/composer-drupal-lenient`** — relaxes the `drupal/core` constraint for an allow-list
  of packages that don't have a D11 release yet (lb_ux, big_pipe_sessionless,
  block_content_permissions).
- During exploration only, `composer-exit-on-patch-failure: false` to collect the full inventory
  of patch failures in one run. **This must be `true` for release.**

Important constraint discovered: `cweagans/composer-patches` v1 cannot reliably resolve
**relative (vendored) patch paths declared by a dependency** (it resolves them from the root
project). So every D11 patch the profile ships must be a **URL** (commit/MR diff), never a repo file.

## Dependency change-set (verified to resolve on core 11.3.x)

`drupal/core-recommended: ^11`, profile `core_version_requirement: ^11`, plus these major bumps —
each because the currently-pinned major is Drupal-10-locked:

| package | 1.x pin | D11 pin |
|---|---|---|
| coffee | `^1.3` | `^2.0` |
| imagemagick | `^3.7` | `^5.0` |
| sophron | `^2.0` | `^3.0` |
| file_mdm | `^3.1` | `^3.1` (already covers D11) |
| webform | `^6.1` | `^6.3` |
| layout_builder_restrictions | `^2.7` | `^3.0` |
| log_stdout | `^1.5` | `^3.0` |
| manage_display | `^2.0@beta` | `^3.0` |
| remove_http_headers | `^1.0 \|\| ^2.0` | `^2.0` |

Everything else already supports D11 at its current constraint.

## The three no-D11-release modules (lenient + URL patch)

Allow-list these in `extra.drupal-lenient.allowed-list`, and patch each with a URL:

| module | patch (URL) | notes |
|---|---|---|
| **big_pipe_sessionless** | [MR!11](https://git.drupalcode.org/project/big_pipe_sessionless/-/merge_requests/11.diff) (issue [3428235](https://www.drupal.org/project/big_pipe_sessionless/issues/3428235)) | applies cleanly; only fix is `MASTER_REQUEST` to `MAIN_REQUEST` |
| **lb_ux** | commit [`55a8b5e8`](https://git.drupalcode.org/project/lb_ux/-/commit/55a8b5e8.diff) (issue [3464547](https://www.drupal.org/project/lb_ux/issues/3464547)) | use the **commit**, not MR!3 — the MR's last commit adds a `composer.json` that conflicts with the dist. Applies after the 2 existing lb_ux patches. |
| **block_content_permissions** | [MR!9](https://git.drupalcode.org/project/block_content_permissions/-/merge_requests/9.diff) (issue [3555336](https://www.drupal.org/project/block_content_permissions/issues/3555336)) | **conflicts** with our existing [2920739](https://www.drupal.org/project/block_content_permissions/issues/2920739) patch (see below) |

> **Patch-URL stability.** `lb_ux` is pinned to a single **immutable commit** diff on purpose —
> `…/-/merge_requests/N.diff` URLs are **mutable** and change as the MR evolves (rebases,
> automated update-bot commits), which can break the build without warning. `big_pipe_sessionless`
> (MR!11, 5 commits) and `block_content_permissions` (MR!9, 2 commits) above still use mutable MR
> diffs because their fix spans several commits — they apply today, but **before tagging `v2.0.0`
> re-verify they apply and pin them to a stable form** (a specific commit/range diff), or better,
> wait for an actual D11 release of each module and drop the patch entirely.

## Known blockers still to resolve

1. **lb_ux resolves to `1.0.0-beta2`, not the dev branch.** `lb_ux: 1.x@dev` + `prefer-stable`
   selects beta2, whose `info.yml` differs from the dev branch the `55a8b5e8` patch targets, so the
   patch silently skips and install fails with `The 'core_version_requirement' key must be present`.
   **Fix: pin `drupal/lb_ux: 1.x-dev`.**

2. **block_content_permissions 2920739 ↔ MR!9 conflict.** Both edit `src/Routing/RouteSubscriber.php`.
   The exploration branch temporarily **drops** the 2920739 "access Custom block library without
   Administer blocks" patch to proceed. For release, reroll 2920739 onto MR!9 and **host it as a URL**
   (the relative-path limitation above), or drop the feature deliberately.

3. **Core patches not yet applied/validated on D11.** The lenient pre-install pulls `drupal/core`
   before the profile's patches are known, so the 5 core patches are skipped; `composer reinstall
   drupal/core` is a composer-patches dead-end. Use a single-install flow (e.g. seed the core patches
   into the host root before install) and confirm the 5 LB/multilingual core patches still apply on D11.

4. **Config-schema layer untested.** Install hasn't passed module-enable yet (blocked by #1), so the
   D11 config compatibility of the bumped v3/v5 modules (log_stdout, manage_display, imagemagick,
   layout_builder_restrictions) is unverified.

## Shortest path to a green D11 install

1. Pin `drupal/lb_ux: 1.x-dev` → clears blocker #1.
2. Re-run CI → hit and fix the config-schema layer (#4).
3. Reroll 2920739 onto MR!9, host as URL, restore the bcp patch (#2).
4. Fix the core-patch install flow; verify the 5 core patches (#3).
5. Flip `composer-exit-on-patch-failure` back to `true`, then tag the first `v2.0.0` release
   (`2.x` is D11-only — the bumped deps drop D10 — per the branch policy above).

## Status of related issues/PRs

- **#58** (Prepare to 11.0 core) — this is the tracking epic.
- **#61** — obsolete stub (partial dep bumps already covered here; its CKEditor5 config already in `1.x`). Closed in favour of the `2.x` exploration.
