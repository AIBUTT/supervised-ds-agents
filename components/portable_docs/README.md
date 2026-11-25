# Portable exporter source

Canonical core: `portable_docs.py`, component **1.1.0**, Python 3.11 or later.
`build_portable_docs.py` is a thin public CLI, and `PORTABLE_DOCS.md` is the deployed
public guide. No project data, project paths or private runtime imports belong here.

The kit's `update-components` command vendors the core byte-for-byte into public
`scripts/portable_docs.py` and records both upstream and installed SHA-256.
The `vendor-core` command provides a guarded path for reviewed legacy adapters.
Make core fixes here first, then preview/adopt them in approved targets; do not
maintain a second divergent exporter implementation. Adapter-specific eligible
figure roots remain in the target's public entry point, not in this core.

Changes to this component must update its version, the kit's component version,
changelog, and focused tests as appropriate. No automated update scans or writes
other repositories. Rebuilding documents is separate from updating source.

## Adopting the component (1.1.0)

This is engineering maintenance, not a scientific release. Deployment copies the
canonical core's bytes unchanged to `scripts/portable_docs.py`; there are no separate
scientific/project-specific forks. Public runtime code imports no private `.github`
modules. The existing private `scripts/docs_site.py` renderer and `build_site` callers
remain unchanged. The exporter runtime requires Python 3.11 or later; the bootstrap
tool itself remains standard-library-only and Python 3.10 compatible.

After bootstrap in **either greenfield or existing mode**, deliberately install the
optional component using the same update command. Bootstrap alone does not install
public exporter files. From the kit root, with the target repository's approved
environment as `<env>` and its path as `<repo>`:

```powershell
conda run -n <env> python tools/kit_render.py update-components --target <repo>
conda run -n <env> python tools/kit_render.py update-components --target <repo> --stage
# Review the plan/proposals; run the next command only after explicit approval.
conda run -n <env> python tools/kit_render.py update-components --target <repo> --apply
```

If conda is not on `PATH`, call `conda.exe` by its full path. Do not install or
upgrade packages automatically.

The default command is read-only. `--stage` creates only review proposals under
`.github/agents/_proposals/portable-docs-1.1.0/`, including `plan.json` with
base/local/new SHA-256. It refuses to overwrite differing prior proposals.
`--apply` creates missing files and updates only unchanged managed files.
Unknown/customized conflicts block the entire apply before target writes.
For a conflict, manually merge the proposal while preserving local content, then
explicitly acknowledge each reviewed result:

```powershell
conda run -n <env> python tools/kit_render.py update-components --target <repo> --apply --accept-local docs/PORTABLE_DOCS.md
```

`--accept-local` is an approval, not an automatic merge or content validation; inspect
the resulting file before using it. Custom files remain custom on subsequent upgrades.
Reapplying an unchanged component is a no-op. The four managed paths are:

- `scripts/portable_docs.py` and `scripts/build_portable_docs.py`;
- `docs/PORTABLE_DOCS.md`;
- `.github/instructions/chat-equations.instructions.md`.

The chat rule applies to every role: write equations in KaTeX (`$...$` inline,
`$$...$$` for blocks) and define terms. Preserve LaTeX in source reports.

The exporter writes packages and its asset cache only under `build/`. Greenfield
projects ignore `/build/` through the skeleton `.gitignore`; `update-components` does
not edit `.gitignore`, so in an existing repository add `/build/` to its `.gitignore`
before the first build. Newly bootstrapped central instructions reference that shared rule.
Existing custom `copilot-instructions.md` and role files are never replaced.

Only `components.portable_docs.version` and its managed hashes in the target marker
advance after every component file succeeds or is explicitly acknowledged. The
top-level `kit_version` remains the workspace's bootstrap version: this command does
not pretend to migrate all private roles/templates. Interrupted writes do not advance
the marker; preview and reapply safely. No commits, tags or external-repository changes
are performed by these component commands.

### Build and share after deployment

From the **target repository root**, create `reports/presentation.md` and its approved
figures, then run:

```powershell
conda run -n <env> python scripts/build_portable_docs.py --include reports/presentation.md --name study-summary --version 1 --date 2026-11-05 --fetch-assets
```

Omit `--fetch-assets` on subsequent builds. Repeat `--include` for each additional
document. No flag means all public docs for compatibility, not implicit link traversal.
Greenfield `scripts/build_docs.py --portable` delegates to the same installed CLI;
existing builders need not be overwritten and can use `build_portable_docs.py` directly.
Use `--cache` to reuse a verified public MathJax cache. The public guide documents
eligible paths, failures, metadata and the optional reproducible `SOURCE_DATE_EPOCH`.
Only existing selected Markdown/figures are consumed; no data analysis runs.

Output: `build/study-summary_v1_2026-11-05.zip`, containing the identically named root
folder. Peers extract it and open that folder's `index.html` in a JavaScript-enabled
browser, with no server, Python or network. Unselected links are visibly scoped out,
not added to navigation. Source updates require rebuilding; never patch generated HTML.
The manifest records exporter/core hashes, versions, selection, UTC build time and
separate date label, input/output hashes and text-line counts.

### Legacy projects without a kit marker

`update-components` explicitly refuses pre-kit workspaces. Do not bootstrap over a
custom private team merely to obtain the exporter. A reviewed public adapter can
instead vendor just the canonical core, with no marker/private-file mutation:

```powershell
conda run -n <env> python tools/kit_render.py vendor-core --target <repo> --path src/example/portable_docs.py
# After review, supply the current file hash printed by preview:
conda run -n <env> python tools/kit_render.py vendor-core --target <repo> --path src/example/portable_docs.py --expected-sha256 <local-sha256> --apply
```

Missing destinations are create-only; existing differing files require their exact
reviewed SHA-256. Preserve the project's adapter, eligible figure roots and CLI tests;
verify the vendored bytes against the canonical core and rebuild. For an entirely
new public entry point, review the standalone CLI and guide as create-only additions.
Legacy chat instructions require a manual approved merge of the shared rule; no tool
claims to migrate arbitrary custom instructions safely.

### Optional real-asset test

`tests/test_components.py::test_generic_deployed_export_with_pinned_cached_assets` runs
only when `PORTABLE_DOCS_TEST_CACHE` (a verified public MathJax asset cache) and
`PORTABLE_DOCS_TEST_CONDA_ENV` (an environment with `markdown` and `pymdown-extensions`)
are set. `CONDA_EXE` selects the conda executable if `conda` is not on `PATH`.
