# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Druxxy (formerly "sdd" — Skilld Drupal Development profile) is a **Drupal 10 installation
profile / distribution**, published as the Composer package `skilldlabs/druxxy`
(`type: drupal-profile`). It is **not a runnable site** — it contains no `web/`, no Drupal
core, and no vendor code. It is meant to be required into a host Drupal codebase, where
Composer installs it to `web/profiles/contrib/druxxy/` and a site is then installed *with*
this profile selected.

The distribution is focused on three things, which explain most design decisions here:
- **Page building** (Layout Builder / Panels / Paragraphs stack).
- **Contributor experience** (curated roles, Gin admin theme, restricted Layout Builder).
- **Strict separation of content and configuration** for clean, repeatable deployments.

## The most important thing to understand

**Behavior lives in YAML config, not PHP.** The only PHP is `druxxy.profile` (~100 lines of
install hooks). Everything else the distribution does — content types, roles, page layouts,
views, media, language settings, security hardening — is exported Drupal configuration under
`config/`. To change what the distribution *does*, you almost always edit or add a YAML file
in `config/`, not write code.

Two config directories with different semantics (standard Drupal profile behavior):
- `config/install/` — applied unconditionally during site install. The 200+ files here are
  the substance of the distribution.
- `config/optional/` — applied at install **only if its dependencies are already met**, and
  also picked up later when a matching module/dependency is enabled. Used here for the
  `seven_*` admin blocks.

Config file naming is `MODULE.config_type.MACHINE_NAME.yml` (e.g.
`node.type.basic_page.yml`, `views.view.content.yml`). Grep by prefix to find a subsystem's
config (`ls config/install | grep page_manager`).

## Install-time logic (`druxxy.profile`)

`hook_install_tasks` / `_alter` add two custom steps:
1. `druxxy_install_setup` — grants user 1 the `sysadmin` role and gives authenticated users
   `access shortcuts`.
2. `druxxy_install_import_language_config` — runs **last** (re-ordered in `_alter`), and
   imports per-language config *overrides* from `config/install/language/<langcode>/*.yml`
   into each non-default language. (Note: the keyed per-langcode subdirectories may be empty
   in the repo; the `language.*` files directly in `config/install/` are normal config, not
   these overrides.) This multilingual mechanism is adapted from the `multilingual_demo`
   project.

## Page-building architecture (read several files to grasp this)

Pages are not rendered by a normal theme region layout — they are assembled through a
**Page Manager + Panels Everywhere + Layout Builder** stack:

- `page_manager.page_variant.panels_everywhere.yml` + `page_manager.page.site_template.yml`
  define the **global page shell** (site branding, main/footer menus, content region) that
  wraps every route via `panels_everywhere`.
- `page_manager.page.node_view.yml` (+ its `*-layout_builder-*` variants) renders nodes
  through Layout Builder rather than the default node template.
- `home`, `contact`, `403_page`, `404_page` are each Page Manager pages with Layout Builder
  variants.
- `custom_pub` adds two publishing options — `promoted_to_403_page` / `promoted_to_404_page`
  — so editors *promote a node* to be shown as the site's 403/404 page (paired with the
  matching Page Manager pages). `system.site` points `403`→`/403`, `404`→`/404`.
- `layout_builder_restrictions`, `layout_library`, and `lb_ux` constrain and improve the
  contributor's Layout Builder experience; `layout_paragraphs` + `paragraphs` provide the
  in-content building blocks.
- `rabbit_hole` (`rh_node`, `rh_media`, `rh_taxonomy`, `rh_user`) controls what entity
  canonical pages do (e.g. redirect/access-deny instead of rendering a standalone page).

## Content & access model

- **Node types:** `basic_page` only (page building does the rest).
- **Block content types:** `media`, `wysiwyg`, `site_template_block`.
- **Paragraph types:** `media`, `wysiwyg`.
- **Media types:** `audio`, `document`, `image`, `remote_video`, `video`.
- **Taxonomy:** `category`.
- **Roles:** `sysadmin` (full admin, auto-assigned to user 1) and `contributor` (editorial),
  plus `authenticated`. Role delegation (`role_delegation`) and per-menu admin
  (`menu_admin_per_menu`) scope what contributors can manage.
- **Themes:** admin = `gin` (with `gin_toolbar`); default/front = `claro`.

## Security & ops posture (baked into config)

`seckit`, `username_enumeration_prevention`, `userprotect`, `password_policy*`,
`private_files_download_permission` (pfdp), `remove_http_headers`, `big_pipe_sessionless`,
and `log_stdout` (logs to stdout for containerized deployments) are all dependencies enabled
and pre-configured. Front page is `/user/login?destination=/admin/content` — an editor-first,
not anonymous-first, default.

## Patches are load-bearing

`composer.json` applies a curated set of patches to `drupal/core` and several contrib modules
(via `cweagans/composer-patches`, `composer-exit-on-patch-failure: true`). Several enable the
Layout Builder + multilingual + media/focal_point integrations this distribution depends on
(e.g. core `3101231`, `3008924`, `2942975`; focal_point media_library integration). **Do not
bump a patched dependency's version without re-checking whether its patches still apply** — a
failing patch aborts `composer install` entirely. Most version-upgrade commits in the git
history are exactly this: bump core/contrib + refresh patch URLs together.

## Working in this repo

There is **no build, lint, or test tooling in this repository** and no CI config — it is pure
profile config plus one PHP file. Validation happens by installing the profile into a real
Drupal site. The normal workflow:

- Edit/add YAML under `config/` (or `druxxy.profile`) directly, **or** make the change in a
  running Drupal site's UI and export it.
- When exporting from a live site, copy only the relevant config into `config/install/` and
  **strip site-specific keys** (`uuid:` and the `_core:` block) so the config stays portable
  across installs — this is the "separation of content and configuration" the distribution is
  built around.
- Adding a new dependency requires editing **both** `composer.json` (`require`) **and**
  `druxxy.info.yml` (`dependencies:`, so the module is enabled at install).

Typical Drupal/Drush commands used against a host site that has this profile checked out into
`web/profiles/contrib/druxxy/` (run from the host site root, not this repo):
- `drush site:install druxxy` — install a site with this profile.
- `drush config:import` / `drush config:export` — sync config (export to compare against, or
  to harvest changes for the profile).
- `drush pm:install <module>` — enable a module (then confirm `config/optional/` items apply).

## Repo facts

- Default branch: `1.x`. Remote: `github.com/skilld-labs/druxxy`.
- License: MIT.
