# Working on the agent kit

This workspace is the kit itself, not a repository where the kit is installed. `templates/private/copilot-instructions.md` is a template rendered into other repositories; this file applies only here.

For any change to the kit, prefer `/evolve-kit` (Kit Maintainer). For direct edits, these rules still apply:

- Keep `{{...}}` placeholders and `{{FILL: ...}}` tokens in `templates/` and `skeleton/`; never paste rendered output back.
- Keep `templates/roles.json`, `templates/private/agents/`, and `ROLE_CATALOG.md` consistent.
- Record every change under `## Unreleased` in `CHANGELOG.md`. Bump `kit.json` only when releasing.
- Run `conda run -n <env> python -m pytest -q -p no:cacheprovider tests` from the kit root before calling a change done.
- Add no machine-specific paths, user names, or project details. Lessons from installed repositories are sanitised first.
- Never commit, tag, or push without explicit approval.
