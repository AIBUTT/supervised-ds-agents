# Evolve-kit procedure

You are changing the kit itself. A kit change reaches every repository that later adopts it, so the standard is higher than for `/evolve-agents`: every change has evidence from more than one context or a stated general reason, a version impact, and the user's approval. Nothing is applied before approval.

The kit root (`<kit>`) is the workspace root: the folder that contains `kit.json`. Run every command from it.

## Rules

- Propose first, apply second. Show each change as a unified diff or a precise description of the new text.
- Installed repositories are evidence only. Read their files; never write, stage, or commit in them.
- Only generic lessons enter the kit. A lesson specific to one repository stays there and is handled by `/evolve-agents`. Remove project names, paths, data descriptions, people, and anything confidential before quoting evidence.
- Never change a model label, a role name, the handoff common fields, the marker format, or the private-by-default rule without explicit approval of that specific item.
- Templates stay templates: keep `{{...}}` placeholders and `{{FILL: ...}}` tokens; never paste rendered output back into `templates/` or `skeleton/`.
- Keep `templates/roles.json`, `templates/private/agents/`, and `ROLE_CATALOG.md` consistent; the kit tests check this.
- Do not add machine-specific paths, user names, or environment names to kit files beyond those that already exist.
- Run Python only through `conda run -n <env> python ...`, with the environment the user approves for kit work. Install nothing.
- Commits, tags, and pushes each need separate explicit approval. Updating `kit.json` or `CHANGELOG.md` does not authorise them.

## Step 1: Pre-flight

1. `git status --porcelain` is empty. If not, show the changes and ask the user to commit or discard them. Do not continue with uncommitted changes.
2. Note `git log -1 --oneline`, `git tag --list`, `kit_version` in `kit.json`, and the `Unreleased` section of `CHANGELOG.md`. Report a released version in `CHANGELOG.md` that has no tag.
3. Ask the user for the Python environment for kit work if it is not stated.
4. Run the kit tests: `conda run -n <env> python -m pytest -q -p no:cacheprovider tests`. If they fail, stop and report; do not build changes on a failing baseline.

## Step 2: Gather evidence

Ask the user which installed repositories to read, unless they named them. For each, read without editing:

- `.github/agents/.kit.json`: `kit_version`, component versions, roster;
- `.github/agents/CHANGELOG.md`: local evolutions, especially ones that recur across repositories;
- `.github/handoffs/ledger.csv`: rejected or blocked stages, failed checks, repeated blocking items;
- `lessons` and `unresolved` in recent handoff files;
- differences between the repository's agent files and the kit templates, which show where real use departed from the kit.

Also read the user's stated reason, the kit `CHANGELOG.md`, and open kit candidates the user reports from earlier `/evolve-agents` runs.

## Step 3: Propose

For each proposal, state:

| Field | Content |
| --- | --- |
| Evidence | Sanitised quote or summary, with the number of repositories or runs it came from |
| Change | Exact new text or diff |
| Files | Every kit file touched, including tests |
| Version impact | Major, minor, or patch (below) |
| Verification | Which test or render check shows it works; or "unverified until used in a run" for prompt wording |
| Risk | What could break in repositories that adopt it |

Version impact:

| Level | When |
| --- | --- |
| Major | Breaks existing workspaces: handoff common fields, marker format, role names or models, renderer CLI |
| Minor | Adds a role, component, command, stage, or rule |
| Patch | Fixes or clarifies without changing what existing workspaces must do |

A change to `components/` also changes that component's own version; state both.

List rejected candidates separately with the reason (for example, specific to one repository, or one occurrence only).

**Stop** for approval of each item.

## Step 4: Apply

1. Apply approved items only.
2. Add or update a test when the change is to code or to a checked consistency rule.
3. Record each change under `## Unreleased` in `CHANGELOG.md`: what changed and the generic lesson behind it. Do not bump `kit.json` here.
4. Run the kit tests; resolve every failure.
5. Render check, outside the kit:
   - create an empty folder `<temp>/kit-check` (the renderer refuses a missing target), where `<temp>` is a new folder under the system temporary directory;
   - `conda run -n <env> python tools/kit_render.py bootstrap --target <temp>/kit-check --mode greenfield --project-name "Kit check" --module-name kitcheck --roles "Coordinator,Data Steward,Researcher,Data Architect,Data Engineer,ML Engineer,Statistical Reviewer,Visualizer,Report Curator" --no-git`;
   - in that folder, `conda run -n <env> python -m pytest -q -p no:cacheprovider` and `conda run -n <env> python .github/scripts/audit.py`. With `--no-git` and no filling, the audit is expected to fail only "private history" and "placeholders" (`{{FILL: ...}}` tokens) and warn about the outer repository; any other failure or warning is a regression;
   - read the rendered files that the change affects and confirm they read as intended.
6. Show `git diff --stat` and the full diff of changed kit files.

## Step 5: Release (only when the user asks)

1. Choose the version from the highest impact in `Unreleased`.
2. Rename `## Unreleased` to `## v<version> (<date>)` and update `kit_version` in `kit.json`.
3. Run the kit tests again.
4. Ask separately for approval to commit, to tag `v<version>`, and to push.

## Step 6: Report

Reply with the applied changes, the test and render-check results, the temporary folder path, the version impact, the items marked unverified, and how adopting repositories pick up the change (`/evolve-agents`, or `update-components` for components).
