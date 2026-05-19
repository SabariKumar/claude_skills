---
name: research-plan
description: The user's foundational working defaults for plan-driven research / benchmarking / paper-track codebases. Combines (a) global tooling and code-style conventions the user always uses (pixi env management, Params/Returns docstrings, hands-off git workflow, branch + issue + PR summaries before major features) with (b) the rolling docs/<topic>_plan.md pattern for research threads — progress table of numbered Steps, dated Findings entries after benchmarks, deferred follow-ups with explicit trigger conditions, lever menus for hyperparameter thickets, and paper-figure specs staged before rendering code. Invoke when (1) starting any feature in the user's projects (the foundational defaults apply broadly), (2) starting a new research thread or picking up an existing plan-driven thread, (3) finishing a benchmark or experiment whose results need recording, or (4) implementing a feature large enough to deserve a numbered Step.
---

# Research-plan-driven development

The user's working defaults for codebases where the goal is a publication / model / benchmark and the work is incremental: every change either advances a Step, records a Finding, or sets up a follow-up. The plan file is the source of truth between sessions; the code is where the plan lands.

The first half of this document — *Foundational defaults* — applies to every project. The second half — *The rolling plan file* and below — applies to research threads specifically.

This skill complements project CLAUDE.md instructions. It does not override them; per-project CLAUDE.md wins on conflict.

---

## Foundational defaults

These conventions apply to every project the user works in, with or without a CLAUDE.md.

### Environment and tooling: pixi

Python projects use [pixi](https://pixi.sh) for dependency management. Always invoke Python via pixi:

```bash
pixi run python ...
pixi run python -m <module> ...
pixi run -e <env> python ...   # when multiple environments are defined
```

Never use bare `python3`, `python`, `PYTHONPATH=...`, or `pip install`. Pixi handles the environment and module resolution. If a project has multiple pixi environments (e.g. `mace` and `default`), the per-project CLAUDE.md or pixi.toml says which to default to; ask if it's not specified.

### Docstrings

Every function and method gets a docstring in this format:

```python
def example(param_a: str, param_b: int) -> SomeType:
    """
    One-line imperative summary ("Compute...", "Return...", "Load...").

    Params:
        param_a: str : description
        param_b: int : description
    Returns:
        description of return value, or None
    """
```

- Always include `Params:` and `Returns:` sections, even if there are no parameters (`Params: None`) or the function returns nothing (`Returns: None`).
- Type annotations belong in the signature; the docstring repeats the type after the colon for readability.
- Keep the one-line summary imperative and concise.

### Git workflow: hands-off

The user manages all git state by hand — commits, branches, staging, pushes, merges, resets, rebases. Never call `git add`, `git commit`, `git checkout`, `git stash`, `git reset`, `git rebase`, `git merge`, `git push`, or any other state-modifying git operation.

Read-only inspection is fine when it informs the work: `git status`, `git log`, `git diff`, `git show`, `git branch --show-current`. Stop after making file edits and let the user commit.

### Branch and issue hygiene before major features

Before implementing any new major feature — new sampling mode, new energy backend, new validation dataset, new clustering strategy, new CLI option group — always:

1. **Prompt the user to switch to a new branch** before writing any code. Do not begin implementation on the current branch.

2. **Write a GitHub issue summary** for the feature, output as a raw markdown code block (so the user can paste it directly without reformatting). Cover:
   - **Background** — what problem motivates the feature; what the current state is and why it falls short.
   - **Proposed approach** — the architecture / design at the level of ideas, not implementation details.
   - **Key questions / unknowns** — what the feature is intended to answer or validate.
   - **Relationship to other work** — upstream dependencies, downstream consumers, related ablations or benchmarks.

3. **Write a pull request summary** when implementation is complete, before the user merges. Output as a raw markdown code block. Template:

```
## Summary/Issue
Link to the relevant GitHub issue. One paragraph describing what the PR does and why.

## Key Features + Dependencies
High-level description of every change. Granular but not technical — describe what changed
and why, not how. Include any dependency additions or removals.
- Added...
- Updated...
- Fixed...

## Tests/Test Steps
Concrete, runnable steps to verify the changes work correctly.
- [ ] Step 1
- [ ] Step 2
- [ ] ...

## Story (Optional)
Design decisions, approaches tried and rejected, and why the chosen approach was selected.

## Related Issues / Branches
Upstream blockers, downstream work, related branches.

## Other Notes
Anything not addressed in this PR that should be handled in follow-up work.
```

Both the issue and PR summaries go in raw markdown code blocks, never rendered. The user pastes them directly into GitHub.

### Architecture diagrams

For any visual that explains a data pipeline, model architecture, decision flowchart, or staged process — i.e. anything more conceptual than a numerical paper-figure — **invoke the `svg-scientific-style` skill** rather than rolling matplotlib / mermaid / hand-drawn SVG. That skill handles the consistent visual style (typography, colours, spacing, arrows) the user expects across diagrams.

Architecture diagrams typically belong in `docs/` (alongside the plan they support) or `scripts/` (when auto-generated from code structure). SVG is the only acceptable output format for these.

Numerical paper figures (boxplots, heatmaps, learning curves) use matplotlib and live in `scripts/<figure_name>.py` per the *Paper figures* section below. Don't conflate the two: matplotlib for data, `svg-scientific-style` for concepts.

---

## The rolling plan file

Every research thread gets one Markdown file at `docs/<topic>_plan.md`. Examples: `docs/mcmm_plan.md`, `docs/peptide_electrostatics_ae_plan.md`, `docs/cremp_overlap_benchmark.md`. One plan per thread; do not interleave multiple unrelated threads.

The plan is the **source of truth**, not the README, not the issue tracker, not the PR description. READMEs document what the code does today; the plan documents how the code got here, why each decision was made, what didn't work, and what's still open. Future-you (or another model picking up the work cold) reads the plan first.

## Plan structure

Every plan opens with this scaffolding:

1. **Title + branch + issue link** (one line each).
2. **Progress table** — numbered Steps with `pending` / `in progress` / `✓ complete` status. Steps are coarse enough that one fits in a session or two; finer-grained subtasks become bullets *within* a Step section.
3. **Goals and constraints** — one paragraph each. What the work is trying to achieve. What's locked (compute budget, API contracts, downstream consumers, deadlines). Things you'd otherwise re-justify every session.
4. **Phase headings** group related Steps. Common phases: *Foundation* (refactors / shared primitives), *Algorithm core*, *Integration*, *Coverage and corrections*. Phases are useful when the plan grows past ~10 Steps; skip them when small.
5. **One section per Step**, in numerical order. Each section has:
   - A short description of what the Step does and why.
   - Implementation outline — bullet points, not prose. Reference the files / functions / classes that will change.
   - **Test invariants** the Step needs to preserve or establish. List them before implementing, not after.
   - **Outcome (YYYYMMDD).** Filled in only when the Step lands. Records what actually shipped, divergences from the outline (and why), test counts (`All N tests pass (was N-k; +k new)`), and any architectural decisions surfaced during implementation. Future-you reads this section to remember *why* the code looks the way it does.
6. **Findings** sections — one per major benchmark or experiment, **dated** (`### Findings 20260505`). Capture: what was run, headline numbers, what we expected vs what we got, the next decisions that fall out. Findings are append-only; never edit a past Findings entry — date a new one if interpretations change.
7. **Risks to instrument from day one** — a list of failure modes the work might hit. For each: how it would manifest, what diagnostic to add upfront, what mitigation is in reserve. Forces the team to write the diagnostic before the failure surprises them.
8. **Deferred follow-ups** — open items with explicit **trigger conditions**, each prefixed with the date added (`- (20260505) Replace metric X with Y if next benchmark shows Z`). "Replace metric X with Y" by itself is bookkeeping; the trigger condition makes it decision-ready when it fires.
9. **Lever menus** — when there's a thicket of orthogonal hyperparameters or design choices the team might pull on (e.g. "things to try if basin coverage is too low"), enumerate them once with rough cost / impact estimates instead of re-deriving the list every meeting. Mark the priority subset (`★`) so the next person knows where to start. Each entry is prefixed with the date added (`- (20260505) ★ Try lower learning rate ...`).

## Daily-work hygiene

- **Update the plan after every Step lands**, in the same session. Don't batch updates across sessions — by next session you've forgotten the load-bearing detail and the Outcome note becomes generic.
- **Mark Steps complete with `✓` in the progress table AND in the section heading** (`### Step 11: Kabsch RMSD dedup — ✓ complete`). The two should never disagree; if they do, the table is wrong.
- **When a Step's implementation diverges from the plan, update the Step section first, then implement.** Don't let the plan and the code drift: the plan's job is to be readable when the code is opaque.
- **Datestamp every addition to the plan** in `YYYYMMDD` format — not relative ("yesterday"), not `YYYY-MM-DD`. The stamp goes where natural for each element:
  - Step added mid-project: `### Step 12 (added 20260519): ...`
  - Step outcome: `**Outcome (20260519).**`
  - Findings heading: `### Findings 20260519`
  - Deferred follow-up: `- (20260519) Replace metric X with Y if ...`
  - Lever menu entry: `- (20260519) ★ Try lower learning rate ...`
- **Append-only for Findings.** A new finding that contradicts an old one gets a new dated entry; don't rewrite history. The contradiction is itself a finding.
- **Capture cross-project dependencies** when they surface. If the work breaks an assumption another project relies on (e.g. a downstream training pipeline), record it in the plan AND update the affected project's memory or plan in the same session.

## Paper figures

- **Draft figure specs in the plan before writing any rendering code.** Spec includes: panel count, what each panel plots, what axis is, what the take-home claim is. Forces the team to decide what they're trying to show.
- **One figure script per figure**, in `scripts/<figure_name>.py`. Reads a CSV emitted by upstream pipeline code; renders the figure deterministically. No "regenerate the data" inside the figure script — keep data and presentation separate.
- **SVG is the default save format.** Optional PDF / PNG via `--out_pdf` / `--out_png` flags.
- **One panel per function.** Layout tweaks (figure size, spacing, font) live in `main`; per-panel data prep + plotting lives in `_panel_a / _b / _c` so any panel can be re-rendered or extracted in isolation.
- For *concept* diagrams (architectures, pipelines, flowcharts), invoke the `svg-scientific-style` skill instead of matplotlib. See *Architecture diagrams* under foundational defaults.

## Scripts and reproducibility

- **Long benchmark runs are resume-aware.** Read the existing output CSV at start; skip rows already present. Per-row append + flush, not "build all rows, write at end". Survives interruption, lets the user kill and restart without re-doing work.
- **Per-record error handling.** Wrap the inner work loop in try/except so one bad input doesn't abort the run. Log and continue.
- **Deterministic seeds for any sampling.** Seeds belong in the CLI flags, not hardcoded inside the script. Default to a fixed seed (`42`) so first-time runs are reproducible without thinking about it.
- **Hyperparameters live in CLI flags, not in adapter code.** When a "tuning experiment" hardcodes a value into the adapter, the value gets stale and re-runs become impossible. CLI flags are the documentation of what was actually tried.

## What NOT to do

- **Do not start without a plan section.** New work that doesn't fit any existing Step gets a new Step before any code is written. "I'll plan it later" never happens; the code calcifies and the plan loses authority.
- **Do not edit past Findings entries.** Append a new dated one. Past entries reflect what was true at the time; rewriting history makes the plan untrustworthy as a historical record.
- **Do not use the README as the plan.** READMEs answer "what does this code do?" not "why does it look this way?" The plan answers the second question.
- **Do not skip the Outcome note.** Without it, the Step's section is the *intended* implementation, not the *actual* one. Six months later, no one remembers the divergences.
- **Do not let lever menus become checklists.** Levers are the menu of options when something doesn't work; they're not a list of things to do in order. The priority `★` subset is the recommendation; the rest is decision-ready inventory.
- **Do not run state-modifying git commands.** The user owns git state; you handle file edits.
- **Do not skip the issue summary before a major feature.** "I'll write it later after we agree on the approach" loses the framing the issue is meant to capture. The issue summary IS the agreement on the approach.

