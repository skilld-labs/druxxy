# Druxxy Backlog

Curated, prioritized view of outstanding work, reconciled against the actual repo state
on 2026-06-22 (branch `1.x`). Each item links to its GitHub issue/PR. This is the
"someday/maybe + planned" list; `TODO.md` holds the active near-term checklist.

Status legend: 🔴 not started · 🟡 in progress (open PR) · ✅ done in code (issue can be closed)

---

## 0. Housekeeping — close already-resolved issues

These issues are fixed in the current code but still open upstream. No code change needed;
just verify and close on GitHub.

- ✅ **[#45] Replace ckeditor with ckeditor5 in info.yml** — `druxxy.info.yml` already
  depends on `ckeditor5`; no legacy `ckeditor` reference remains. Close as done.
- ✅ **[#50] Get rid of webp requirement in info file** — no `webp` dependency in
  `druxxy.info.yml` or `composer.json` (removed in commit `3963b6c`). Close as done.
  (The optional core WebP/AVIF quality patches listed in the issue can be split into a new,
  separate enhancement issue if still wanted — see §3.)

---

## 1. Drupal 11 readiness (epic)

The headline upgrade (current release targets D10.6). **Exploration done on the `2.x` branch** — D11
is achievable; the verified dependency change-set, the lenient + URL-patch approach, and the
remaining blockers are written up in [`docs/upgrading-d11.md`](docs/upgrading-d11.md). PR **#61 is
obsolete** (superseded by the `2.x` exploration).

- 🟡 **[#58] Prepare to 11 core** — the tracking epic. Exploration done on the `2.x` branch
  (write-up in [`docs/upgrading-d11.md`](docs/upgrading-d11.md)); the old `58-up-for-11` stub
  (PR #61) is **closed/superseded**. Remaining work to ship `v2.0.0`:
  - Symfony 7 + Drush 13 compatibility.
  - Deprecated/removed core modules (Forum, Tracker, Action, Book, Statistics, Tour) —
    confirm none are pulled in transitively.
  - Per-dependency D11 compatibility audit (see §2) — the real bulk; mostly mapped in the doc.
  - PHPUnit 11 if/when a test suite is added.

## 2. Dependency & patch hygiene (prerequisite for D11)

Every patched dependency is a potential upgrade blocker. Before/while moving to D11:

- 🔴 Audit each of the ~30 contrib modules in `composer.json` for a D11-compatible release.
- 🔴 Re-validate every entry in `composer.json` → `extra.patches`. Each patch must either
  still apply on the target version or be dropped because it landed upstream. A failing
  patch aborts `composer install` (`composer-exit-on-patch-failure: true`).
- 🔴 Flag modules with no stable D11 release (currently several are on `@beta`/`@dev`:
  `file_entity:dev-2.x`, `lb_ux:1.x@dev`, `formblock`, `layout_library`,
  `manage_display`, `page_manager`, `panels_everywhere`) — these gate the upgrade.
- 🔴 Image stack & D11: pinned at `imagemagick ^3.7`, `file_mdm ^3.1`, `sophron ^2.0` for
  D10.6 (imagemagick 3.7 needs `file_mdm ^3` + `sophron ^2.0.2`). `file_mdm ^3.1` and
  `sophron ^2.x` already allow D11 — the D11 blocker in this stack is `imagemagick` (bump
  `^3.7`→`^5.0`; imagemagick 4.x/5.x also pull `file_mdm ^3.1+`).

## 3. Open bugs & smaller enhancements

- ✅ **[#62] svg_image breaks image loading setting** — "Eager" loading was ignored, images
  stayed `lazy`. The fix (drupal.org `svg_image` `3257729`) shipped in **svg_image 3.2.0**, so
  the constraint was bumped to `^3.2` (a patch would now fail to apply). Verify on install and
  close #62.
- 🟡 **[#51 / PR #52] Add upgrade path for page_manager** — PR #52 adds the
  `page_manager` patch (drupal.org `3398407`). No `page_manager` patch is in `composer.json`
  today, so PR #52 is still unmerged. Review, rebase, verify the patch applies on the pinned
  `page_manager:^4.0@beta`, and merge.
- 🔴 **[#4] Add content locking to druxxy profile** — add `drupal/content_lock`, enable in
  `druxxy.info.yml`, ship default config. Improves contributor experience (core goal). Scope
  which entity types/forms it should lock.
- 🔴 **(from #50) Optional WebP/AVIF tuning** — core patches for quality config / default
  conversions / AVIF (`3320689`, `3406267`, `3202016`). Nice-to-have; create a dedicated
  issue if pursued.

## 4. Documentation (epic) — see PLAN.md

- ✅ `CLAUDE.md` — architecture & working notes for AI/devs (done this session).
- ✅ Expand `README.md` — purpose, highlights, requirements, install, working notes, CI badge,
  links to `CLAUDE.md`/`PLAN.md`/`BACKLOG.md`/`TODO.md`.
- 🔴 `docs/architecture.md` — the page-building stack (Page Manager + Panels Everywhere +
  Layout Builder), content model, roles.
- 🔴 `docs/installing.md` — host-site Composer + `drush site:install druxxy` walkthrough.
- 🔴 `docs/config-conventions.md` — config/install vs config/optional, stripping
  `uuid:`/`_core:`, keeping content/config separated.
- 🔴 `docs/upgrading.md` — the repeatable bump-core-and-refresh-patches procedure.

## 5. Project infrastructure (no tooling exists today)

The repo has **no CI, no lint config, no tests**. To make upgrades safe and repeatable:

- ✅ Add an install smoke test in CI — `.github/workflows/test.yml` runs `composer validate`,
  PHP lint, and `drush site:install druxxy` (Drupal 10.6 / PHP 8.3) on each push/PR. Extend
  the matrix (more PHP/core versions, D11) as upgrade work proceeds.
- 🔴 Add config validation / coding standards (`drupal/coder` + phpcs on `*.profile` and
  YAML lint) so contributions stay consistent.
- ✅ Supported-branch policy decided: `1.x` = Drupal 10 (maintained for existing projects),
  `2.x` = Drupal 11 (new major). Documented in `README.md` + `docs/upgrading-d11.md`.
