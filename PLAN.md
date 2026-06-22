# Druxxy — Documentation & Upgrade-Readiness Plan

A plan to (A) document the distribution so contributors and AI tools can be productive, and
(B) make Drupal core/contrib upgrades a safe, repeatable routine instead of a one-off scramble.
Companion files: `BACKLOG.md` (full work list) and `TODO.md` (active checklist).

Snapshot: 2026-06-22, branch `1.x`, pinned to `drupal/core-recommended: ^10.6`.

---

## Why now

- The repo is config-driven with essentially no prose docs (`README.md` is 2 lines) and **no
  CI/lint/tests**. Upgrades are currently done by hand and verified only by installing.
- Drupal 11 is GA; Drupal 10 moves toward end-of-life. The D11 prep (issue #58 / PR #61) has
  stalled as a stub since 2024 and needs real groundwork.
- Several open issues are actually already fixed in code (#45, #50) — a sign the backlog has
  drifted from reality. This plan re-anchors it.

## Goals

1. A contributor can understand the architecture, install the profile, and make a
   config change correctly **from the docs alone**.
2. Bumping Drupal core (minor or major) follows a written, checklist-driven procedure with an
   automated install smoke test.
3. The GitHub backlog reflects the true state of the code.

---

## Part A — Documentation

`CLAUDE.md` (architecture + working notes) is done. Remaining work, in priority order:

1. **`README.md`** — first impression for drupal.org/GitHub visitors. Cover: what Druxxy is,
   who it's for, the three pillars (page building / contributor UX / content-config
   separation), how to require it via Composer, the one-line install, and links into `docs/`.
2. **`docs/architecture.md`** — the page-building stack and how a page is actually assembled:
   Page Manager `site_template` + Panels Everywhere shell → Layout Builder node view → error
   pages via `custom_pub` (promote-to-403/404). Content model (node `basic_page`, block/
   paragraph/media types, `category` taxonomy) and roles (`sysadmin`, `contributor`).
3. **`docs/installing.md`** — host-site setup: `composer require skilldlabs/druxxy`, installer
   paths, `drush site:install druxxy`, what the install tasks do (user-1 `sysadmin`,
   per-language config import).
4. **`docs/config-conventions.md`** — the rules that keep the distribution portable:
   `config/install` vs `config/optional`; strip `uuid:` and `_core:` from exported config;
   add a new module to **both** `composer.json` and `druxxy.info.yml`; never commit
   site-specific content as config.
5. **`docs/upgrading.md`** — see Part B; the procedure lives here once written.

Approach: commit skeletons first (headings + TODOs) so the structure is reviewable, then fill
in. Keep each doc focused; link rather than duplicate `CLAUDE.md`.

## Part B — Upgrade readiness

The fragile part of every upgrade here is the **patch set** (`composer.json` →
`extra.patches`) — a failing patch aborts `composer install` outright. Make this systematic.

### B1. Build the dependency & patch matrix (do this first)
For every module in `composer.json` record: current constraint · latest stable for current
core · D11-compatible release (or "none yet"). For every patch record: still-applies /
merged-upstream / needs-refresh. This single table tells you exactly what blocks D11.
Known soft spots: `@dev`/`@beta` pins — `file_entity:dev-2.x`, `lb_ux:1.x@dev`,
`page_manager`, `panels_everywhere`, `formblock`, `layout_library`, `manage_display`.

### B2. Add a safety net before changing versions
- CI install smoke test: GitHub Actions job that runs `drush site:install druxxy` against the
  target core version on every PR. This is the single highest-leverage addition — it turns
  "did the upgrade break install?" into an automatic check.
- `drupal/coder` + phpcs (on `druxxy.profile`) and a YAML lint for config.

### B3. Write the repeatable bump procedure (`docs/upgrading.md`)
Codify what the git history already does informally on every core bump:
1. Bump `drupal/core-recommended` (and affected contrib) in `composer.json`.
2. Refresh/remove each patch per the matrix; run `composer update` and confirm all patches
   apply.
3. Update `druxxy.info.yml` dependencies/themes to match any module changes.
4. Run the install smoke test + a manual page-building pass (create a Layout Builder page,
   media, a 404-promoted node).
5. Tag/release.

### B4. Drupal 11 epic (#58 / PR #61)
Only after B1–B3 are in place:
- Decide branching: continue `1.x`, or cut `2.x` for D11 and keep `1.x` on D10 (recommended if
  D10 host sites still need support).
- Resolve Symfony 7 / Drush 13; confirm no removed core modules (Forum, Tracker, Action, Book,
  Statistics, Tour) are pulled in transitively.
- Rebuild PR #61 on the green matrix — treat the existing branch as reference, not a mergeable
  change.

---

## Phasing

- **Phase 1 — Reconcile & quick wins (days):** close #45 & #50; merge svg_image (#62) and
  page_manager (#51/#52) patches; expand `README.md`. Result: backlog matches reality.
- **Phase 2 — Foundations (1–2 weeks):** CI install smoke test + phpcs; `docs/` skeletons;
  dependency & patch matrix (B1).
- **Phase 3 — D11 (project):** branching decision; D11-compat dependency/patch work; rebuild
  PR #61; release.

## Definition of done

- `README.md` + four `docs/` files exist and are accurate.
- CI runs an install smoke test and phpcs on every PR.
- `docs/upgrading.md` procedure has been used for at least one real core bump.
- All GitHub issues reflect true code state (stale ones closed).
- A documented decision exists on D10-vs-D11 branch support.
