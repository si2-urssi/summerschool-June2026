# Storing Jupyter Notebooks in Git

*Keeping `.ipynb` files clean, diffable, and reviewable.*

A short tour of the options: strip outputs, strip metadata, or pair with a text file (Jupytext).

---

## Why notebooks fight with version control

An `.ipynb` file is JSON that stores code, results, and metadata all together; git sees all of it. That causes four recurring problems:

- **Bloated repositories.** Embedded plots and images are base64 blobs. Every rerun rewrites them, so the repo grows fast.
- **Painful merge conflicts.** Two people run the same notebook and execution counts, outputs, and metadata all collide.
- **Hard code review.** Reviewers wade through thousands of lines of output JSON to find the few lines of code that changed.
- **Huge, noisy diffs.** A one-line code edit can show as a 500-line diff once outputs and counters are regenerated.

---

## The shared idea behind every option

Commit the parts that matter; drop or separate the parts that don't. An `.ipynb` mixes three layers:

- **Code** — what you actually want to review and track. *Keep.*
- **Outputs** — plots, tables, printed results; big and volatile. *Strip or separate.*
- **Metadata** — execution counts, kernel info, widget state. *Strip or separate.*

The options below differ only in *how* they separate these layers, and in what ends up in the commit.

---

## Option 1: Strip outputs with nbstripout

**What it does**

- Removes all cell outputs (and execution counts) so only code + structure are committed.
- Runs automatically before each commit, or as a git filter on the working tree.
- Simplest, lowest-friction fix; kills the bloat and most merge noise.
- Trade-off: rendered results no longer live in the repo, so you lose the at-a-glance view on GitHub.

**Setup** (`.pre-commit-config.yaml`)

```yaml
repos:
  - repo: https://github.com/kynan/nbstripout
    rev: 0.8.1
    hooks:
      - id: nbstripout
        files: ".ipynb"
```

```bash
$ pre-commit install
```

Or install it as a git filter directly:

```bash
$ nbstripout --install
```

More: https://github.com/kynan/nbstripout

---

## Option 2: Keep outputs, strip the noisy metadata

When you *do* want results in the repo (tutorials, shared analyses), strip only the volatile bits.

**What's noisy.** Execution counts, kernelspec, `language_version`, widget state, signature, and editor metadata change constantly even when code doesn't.

**How to do it.** Use nbstripout flags to keep outputs but drop counts/metadata, or set a git filter via `.gitattributes` so it's transparent and repo-wide.

```
# .gitattributes
*.ipynb filter=nbstripout
*.ipynb diff=ipynb
```

```bash
$ nbstripout --install --attributes .gitattributes
$ nbstripout --keep-output --keep-count   # selective stripping
```

More: https://github.com/kynan/nbstripout — see the `--keep-output` / `--keep-metadata-keys` flags.

---

## Option 3: Pair with a text file using Jupytext

**What it does**

- Represents a notebook as a plain `.py` (or `.md`) script, paired and kept in sync with the `.ipynb`.
- Commit only the `.py` file: clean line-by-line diffs, easy review, trivial merges.
- The `.py` reads as normal code with `# %%` cell markers; works with black, ruff, isort.
- Trade-off: outputs aren't stored at all; collaborators regenerate them by running the notebook.

**Setup**

```bash
# one-off pairing
$ jupytext --set-formats ipynb,py:percent nb.ipynb

# keep them in sync
$ jupytext --sync nb.ipynb
```

As a pre-commit hook:

```yaml
- repo: https://github.com/mwouts/jupytext
  rev: v1.17.3
  hooks:
    - id: jupytext
      args: [--sync]
```

More: https://jupytext.readthedocs.io · https://github.com/mwouts/jupytext

---

## Which one should you reach for?

| Approach | Outputs in repo? | Diff quality | Best when… |
|---|---|---|---|
| nbstripout (strip outputs) | No | Good | Code-focused projects; you just want the bloat gone |
| Strip metadata only | Yes | Better | Tutorials / shared analyses where rendered results matter |
| Jupytext pairing | No (commit `.py`) | Best | Real code review, heavy collaboration, linting & CI |

**They combine.** A common setup: Jupytext to pair + sync, with nbstripout also in pre-commit. Pick per repo, not per career.

---

## Go home and try this

Pick one repo with notebooks and set up a pre-commit hook this week.

1. **Add pre-commit.** You already saw it for linting in this course; notebooks plug into the same `.pre-commit-config.yaml`.
2. **Strip first.** Start with nbstripout. Instant relief from bloat and most merge conflicts, near-zero effort.
3. **Pair when reviewing.** Add Jupytext when notebooks need real code review or live in CI.

### Further reading

- nbstripout — https://github.com/kynan/nbstripout
- Jupytext — https://jupytext.readthedocs.io
- pre-commit framework — https://pre-commit.com
- Jupytext pre-commit guide — https://jupytext.readthedocs.io/en/latest/using-pre-commit.html
- nbdime (notebook-aware diff/merge) — https://nbdime.readthedocs.io
