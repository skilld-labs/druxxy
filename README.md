# Druxxy

[![Test](https://github.com/skilld-labs/druxxy/actions/workflows/test.yml/badge.svg)](https://github.com/skilld-labs/druxxy/actions/workflows/test.yml)

_Previously known as "sdd" (Skilld Drupal Development installation profile)._

Druxxy is a Drupal 10 distribution (installation profile) focused on **page building**,
**contributor experience**, and a **strict separation of Drupal content and Drupal
configuration** for streamlined, repeatable deployments.

It ships a curated set of contrib modules pre-wired and pre-configured so a new site starts
with a working page-building stack and sensible security defaults out of the box.

## Highlights

- **Page building** — Layout Builder driven by Page Manager + Panels Everywhere, with
  Layout Paragraphs, a Layout Library, and Layout Builder UX/restriction tooling for editors.
- **Contributor experience** — Gin admin theme and toolbar, role delegation, per-menu
  administration, and a focused editorial role.
- **Content/config separation** — behaviour lives in exported YAML configuration (under
  `config/`), keeping editorial content and site configuration cleanly apart for clean
  deploys.
- **Security & ops defaults** — SecKit, username-enumeration prevention, user protection,
  password policy, private file download permission, big_pipe (sessionless), and
  stdout logging for containerised environments — all enabled and configured.
- **Media & images** — media library, focal point cropping, SVG support, and ImageMagick/
  Sophron image toolkit.
- **Multilingual** — language config import at install time, adapted from `multilingual_demo`.

## Requirements

- Drupal core `^10.6` (installed via Composer; pulled in by this profile).
- PHP and extensions per the targeted Drupal core release.
- Composer 2, with the `composer/installers` and `cweagans/composer-patches` plugins allowed
  (the profile applies several core/contrib patches — see `composer.json` → `extra.patches`).

## Installation

Druxxy is a Composer package (`skilldlabs/druxxy`, type `drupal-profile`); it is **not a
runnable site on its own**. Require it into a Drupal codebase, then install a site with it.

```bash
# In a Drupal project (e.g. created from drupal/recommended-project)
composer require skilldlabs/druxxy

# Install a site using the profile
vendor/bin/drush site:install druxxy
```

> Because the profile applies patches, ensure your project's `composer.json` allows the
> `cweagans/composer-patches` plugin (`config.allow-plugins`). A failing patch aborts
> `composer install`.

### What the install does

- Assigns user 1 the `sysadmin` role and grants authenticated users access to shortcuts.
- Imports per-language configuration overrides last in the install sequence.
- Applies the distribution's exported configuration: one `basic_page` node type;
  `media` / `wysiwyg` / `site_template_block` block types; `media` / `wysiwyg` paragraph
  types; `audio` / `document` / `image` / `remote_video` / `video` media types; a `category`
  taxonomy; the `sysadmin` and `contributor` roles; and the Page Manager / Panels Everywhere
  page-building shell (home, contact, node view, and 403/404 pages).

## Working on the distribution

Almost all behaviour is configuration, not PHP. To change what the distribution does, edit or
add YAML under `config/` (`config/install` is applied unconditionally; `config/optional` only
when its dependencies are met). When harvesting config from a running site, strip
site-specific keys (`uuid:` and the `_core:` block) so it stays portable. Adding a module
requires updating **both** `composer.json` (`require`) and `druxxy.info.yml` (`dependencies`).

See [`CLAUDE.md`](CLAUDE.md) for a deeper architecture overview, and [`PLAN.md`](PLAN.md) /
[`BACKLOG.md`](BACKLOG.md) / [`TODO.md`](TODO.md) for the roadmap and outstanding work.

## Continuous integration

The [`test` workflow](.github/workflows/test.yml) validates `composer.json`, lints PHP, and
runs an end-to-end install smoke test (`drush site:install druxxy`) on every push and pull
request.

## License

MIT — see [LICENSE](LICENSE).
