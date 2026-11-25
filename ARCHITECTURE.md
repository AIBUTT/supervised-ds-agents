# Architecture

How the kit is organised, what it does to a repository, and how it protects existing work. For a first look, start with the [README](README.md).

## Kit layout

| Path | Purpose |
| --- | --- |
| `kit.json` | Kit version, CCDS release followed, default environment and Python version |
| `ROLE_CATALOG.md` | The nine roles, fixed models, ownership, and when to include each |
| `CHANGELOG.md` | Kit versions and the lessons each one adopted |
| `prompts/` | Full procedures: `bootstrap-agents.md`, `evolve-agents.md`, `audit-agents.md`, and `evolve-kit.md` for the kit itself |
| `user-prompts/` | Thin VS Code slash commands that point to `prompts/`; installed with `tools/kit_render.py install-prompts` |
| `templates/roles.json` | Machine-readable role definitions |
| `templates/private/` | Everything rendered into a repository's `.github/` |
| `templates/hooks/pre-commit` | Outer-repository hook that rejects staged `.github` paths |
| `skeleton/ccds_v2_minimal/` | Minimal Cookiecutter Data Science v2 structure for empty repositories |
| `skeleton/CCDS_SOURCE.md` | CCDS release followed and the documented deviations |
| `tools/kit_render.py` | Create-only bootstrap, role addition, prompt installation, and preview-first component adoption |
| `components/portable_docs/` | Optional offline Markdown exporter, its CLI, and its guide |
| `tests/` | Kit tests |
| `.github/` | Kit-only: Kit Maintainer agent, `/evolve-kit` command, CI, rules for editing the kit; never rendered into repositories |

## Slash commands

The commands in `user-prompts/` contain a `{{kit_root}}` placeholder. `install-prompts` copies them to the VS Code user prompts folder with the kit's absolute location filled in. It creates missing files, leaves identical ones alone, and refuses to replace a differing file unless `--overwrite` is given. Each command contains a single instruction to read and follow the matching procedure in `prompts/`, so the procedures stay in one versioned place. A prompt file runs only when typed.

## Lifecycle

1. **Bootstrap** (once per repository). Pre-flight checks, discovery or interview, proposed roster and settings, your approval, deterministic render, repository-specific filling, audit, commit to the private history.
2. **Work.** Multi-stage runs through the Coordinator, with handoff files and the ledger as the record.
3. **Audit.** Read-only checks at any time.
4. **Evolve.** Evidence from the ledger, `lessons`, and repository changes becomes proposed edits; nothing is applied without approval; every change is logged and committed.
5. **Kit update.** Lessons that apply to every repository are added to the kit as a new version through `/evolve-kit`.

## Empty and existing repositories

| | Greenfield (empty folder) | Existing repository |
| --- | --- | --- |
| Discovery | Interview: goal, data, confidentiality, deliverables, modeling, names, environment | Inventory of layout, data, tests, docs, conventions |
| Skeleton | CCDS v2 minimal structure rendered | Not rendered; existing layout kept |
| Roster | Small: Coordinator, Researcher, Data Engineer, Statistical Reviewer | Chosen from evidence |
| Paths in agent files | Kit defaults; missing paths marked "(planned)" | Mapped to the repository's real paths |
| Conflicts | None expected | Written to `.github/agents/_proposals/` for review |

## Safety layers

| Layer | Protects against |
| --- | --- |
| Marker `.github/agents/.kit.json` | A second bootstrap: the procedure and the renderer both refuse |
| Create-only renderer | Overwriting files: conflicts become proposals |
| Dry run before render | Unexpected file lists |
| Pre-flight: clean outer tree, no tracked `.github` | Mixing kit files with uncommitted work; repositories that publish `.github/` |
| `.git/info/exclude` and the pre-commit hook | Private files entering the outer repository |
| Private repository in `.github/` | Losing or being unable to undo agent changes; `git clean -fdx` in the outer repository |
| Separate commands | Accidental changes: only `/evolve-agents` edits an existing workspace, and only after approval |

## Environments

`default_conda_env` in `kit.json` (`ds-shared`) is the suggested environment for repositories without their own; bootstrap asks which environment to use every time. A shared environment needs extra care: every handoff records a package-list hash (`ledger.py env-hash`), and installs require approval because they affect other projects. A repository with a dedicated environment sets it once at bootstrap (`--conda-env`); the agent files never name an environment directly. `/evolve-agents` can move a repository from a shared environment to a dedicated one.

## Undo

```powershell
git -C .github log --oneline           # private history
git -C .github revert <commit>         # undo one change
```

The private history covers `.github/` only. Code is restored through the outer repository; generated outputs are regenerated from their run record.

## Deliberate re-bootstrap

Re-bootstrapping is rarely needed; prefer `/evolve-agents`. If it is needed:

1. Commit the private history and copy `.github/` to a backup folder outside the repository.
2. Delete `.github/agents/.kit.json` by hand.
3. Run `/bootstrap-agents`. Existing files become proposals; nothing is overwritten.

## Components

Optional parts are adopted separately from bootstrap, with their own versions and a preview-first update command. The only component so far is the portable-docs exporter; see [components/portable_docs/README.md](components/portable_docs/README.md).

## Known limitations

- The kit assumes a repository publishes nothing under `.github/`. A repository that needs public `.github/` content (for example CI workflows) is refused at pre-flight and needs a separate decision.
- Model labels request a model; they do not guarantee a model snapshot. Record the actual model when the client reports it.
- Agents cannot run notebook cells with the default tools; long notebooks are run by the user.
- The skeleton follows CCDS `v2.3.0` by hand; later CCDS releases are not adopted automatically (see `skeleton/CCDS_SOURCE.md`).
- Developed and tested on Windows only.
