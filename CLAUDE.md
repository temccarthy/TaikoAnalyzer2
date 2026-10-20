# Taiko Analyzer

> Turn taiko drum audio into readable, editable notation, with every pipeline
> stage visible and measurable.

## Start Here

- [goals.md](goals.md) — project goals, pipeline stages, decisions, open questions. Read first.

## Git & Versioning

Version control is mandatory. Every change should be checkable and reversible.

- **Commit automatically** when a feature, fix, or doc change is complete and
  verified. One logical change per commit. Don't ask before committing.
- **Never push without asking.** Always prompt before `git push`.
- **Version** lives in [VERSION](VERSION) (single source of truth). Semver,
  pre-1.0:
  - `0.x.0` — minor feature additions (reset patch to 0)
  - `0.0.x` — hotfixes and patches
  - Docs/config-only changes don't bump the version.
- Bump VERSION in the same commit as the change it versions, and state the new
  version in the commit message.
