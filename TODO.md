# Druxxy TODO

Active, near-term checklist. Pull items here from `BACKLOG.md` as they become "next".
Ordered roughly by effort-vs-value: quick wins first, the D11 epic last.
Snapshot date: 2026-06-22 (branch `1.x`).

## Quick wins (do first)
- [ ] Close issue **#45** (ckeditor5) — already done in code, verify & close on GitHub.
- [ ] Close issue **#50** (webp requirement removed) — already done in code, verify & close.
- [x] **#62** svg_image "Eager" loading fix — the upstream fix (drupal.org `3257729`) landed
      in **svg_image 3.2.0**, so bumped the constraint to `^3.2` instead of patching (the old
      patch would fail to apply on 3.2.0). Verify on a real install, then close #62.
- [ ] **#51 / PR #52** Review & merge the `page_manager` patch (drupal.org `3398407`):
      rebase PR #52 on `1.x`, confirm the patch applies on `page_manager:^4.0@beta`, merge.

## Documentation (parallel track — see PLAN.md)
- [x] Expand `README.md` beyond the current 2 lines (purpose, install, links).
- [ ] Create `docs/` and add `architecture.md`, `installing.md`, `config-conventions.md`,
      `upgrading.md` (skeletons first, fill in iteratively).

## Upgrade-readiness groundwork (before touching core version)
- [ ] Build a dependency matrix: for each module in `composer.json`, record current
      constraint + latest D10 stable + D11-compatible release (or "none yet").
- [ ] Audit `extra.patches`: mark each patch as still-needed / merged-upstream / must-refresh.
- [ ] List which `@dev`/`@beta` deps block D11 (file_entity, lb_ux, page_manager,
      panels_everywhere, formblock, layout_library, manage_display).

## Project infrastructure
- [x] Add a CI install smoke test (`drush site:install druxxy`) on PRs —
      `.github/workflows/test.yml` (validate + PHP lint + install). Tune PHP/core matrix later.
- [ ] Add phpcs (`drupal/coder`) + YAML lint.

## Drupal 11 epic (#58 / PR #61) — start only after the groundwork above
- [ ] Decide branching strategy: continue on `1.x` vs open `2.x` for D11.
- [ ] Resolve Symfony 7 / Drush 13 compatibility.
- [ ] Confirm no removed core modules (Forum, Tracker, Action, Book, Statistics, Tour) are
      pulled in.
- [ ] Rebuild PR #61 on top of the green dependency/patch audit; do not merge the current stub.
- [ ] Run a full install + page-building smoke test on D11 before release.

---
_When an item is done, move it to a CHANGELOG/commit message and delete it here. Keep TODO.md
short — long-lived ideas live in `BACKLOG.md`._
