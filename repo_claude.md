# peptide-electrostatics — Claude instructions

## Environment

This project uses [pixi](https://pixi.sh) for dependency management. Always use pixi to resolve dependencies and run Python:

```bash
pixi run python ...
pixi run python -m <module> ...
```

Never use bare `python3` or `PYTHONPATH=...` — pixi handles the environment and module resolution automatically.

## Docstrings

All functions and methods must use the following docstring format:

```python
def example(param_a: str, param_b: int) -> None:
    """
    One-line summary of what the function does.

    Params:
        param_a: str : description
        param_b: int : description
    Returns:
        description of return value, or None
    """
```

- Always include `Params:` and `Returns:` sections, even if there are no parameters (`Params: None`) or the function returns nothing (`Returns: None`).
- Type annotations belong in the signature; the docstring repeats the type after the colon for readability.
- Keep the one-line summary concise and imperative ("Compute...", "Return...", "Load...").

## Branch and issue hygiene

Before implementing any new major feature (new model component, new training mode, new decoder head, new encoder type, new ablation study), always:

1. **Prompt the user to switch to a new branch** before writing any code. Do not begin implementation on the current branch.

2. **Write a GitHub issue summary** for the feature. Output it as raw markdown so the user can paste it directly into GitHub. Follow the same style as the READMEs (see below): imperative, explain *why* not just *what*, no line-by-line narration of code. A good issue summary covers:
   - **Background** — what problem or question motivates the feature; what the current state is and why it falls short
   - **Proposed approach** — the architecture or design, explained at the level of ideas not implementation details
   - **Key questions / unknowns** — what the feature is intended to answer or validate
   - **Relationship to other work** — upstream dependencies, downstream consumers, related ablations or benchmarks

3. **Write a pull request summary** when implementation is complete, before the user merges. Use the template in `.github/PULL_REQUEST_TEMPLATE.md`, reproduced here for reference:

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

Both outputs should be in raw markdown so the user can paste them directly into GitHub without reformatting.

---

## READMEs

Every subdirectory that contains code must have a `README.md`. A user unfamiliar with
the codebase should be able to read it and understand the code without needing to open
any source files first.

Each README must cover:

- **Purpose** — what problem this module solves and where it fits in the overall pipeline
- **Module contents** — one paragraph per file explaining what it does, key classes/functions, and non-obvious design decisions
- **Data contracts** — the shape, dtype, and meaning of inputs and outputs for the main entry points
- **Critical parameters or constraints** — anything that silently breaks correctness if misconfigured (e.g., `max_neighbors` in the encoder)
- **Dependencies on other modules** — what this module consumes from siblings and what it produces for downstream modules

Write for a reader who knows Python and ML but has never seen this codebase.
Do not summarise what code does line-by-line; explain *why* the code is structured the way it is.

