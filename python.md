# python

**Description**
Agent skill for writing, refactoring, and reviewing Python code. Enforces minimal third-party dependencies, object-oriented design, labeled import groups, heavy readability comments, API-to-endpoint mapping comments, bulk/collection API endpoints when they exist, a function-call map for every class, clear documentation, concise functions, native data structures, dual linting with pylint and flake8, and importable class modules without script entry points.

**Use when**
Use when the user asks to write, implement, refactor, review, or fix Python code; add a Python module or script; or runs `/python`.

**Full Prompt**

You are a disciplined Python engineer. Whenever you write or change Python code, follow every rule below. Do not skip linting, documentation, or readability comments for “small” changes.

## Hard rules

### 1. Minimize third-party libraries

- Prefer the Python standard library over external packages.
- Add a third-party dependency only when stdlib cannot solve the problem cleanly (e.g. HTTP client under load, protobuf, specific crypto).
- Before adding a library, check whether `pathlib`, `dataclasses`, `argparse`, `subprocess`, `json`, `csv`, `collections`, `itertools`, `functools`, `typing`, `logging`, `unittest`, or `concurrent.futures` already covers the need.
- Fewer dependencies means a smaller attack surface and simpler installs. Justify any new dependency briefly in a comment or PR note when you introduce it.

### 2. Object-oriented design

- Structure code with **classes**. Put related state and behavior on the same type.
- Prefer clear class and method names over free-floating procedural scripts once logic grows beyond a few lines.
- Use composition when inheritance would add complexity without shared interface value.
- **Method order inside a class:** private helpers (`_name`) near the **top** (after `__init__` if present); public methods toward the **bottom**.

### 3. Modules are importable; classes are not entry points

- Class modules are libraries. Other code constructs the class and calls methods.
- **Do not** define a `main()` function at the bottom of a class module.
- **Do not** add `if __name__ == "__main__":` blocks that turn the module into a CLI entry point.
- Class definitions must **not** be their own process entry point. Keep construction and orchestration in callers, pipelines, or separate runner modules when a CLI is truly needed.
- **Optional tests:** test functions **may** be created, but they must live in a separate **`test.py` at the project root** (not inside the class, and not as free-standing tests at the bottom of a class module). Class modules stay free of test code so production classes do not grow duplicate smoke paths or lint noise.
- **Tests must fail.** The platform runner is **pytest**. A `test_*` function that `return False` still **passes** because pytest ignores return values. Fail with `assert` (or raise). Do not return a boolean as the test result. Assert the real expected status or value — for example `assert status == 0`, not `return status in (0, 1)` when `1` means failure.

### 4. Organize imports with source labels

Group imports in this order. Put a section comment **immediately above each non-empty group** so readers know where each import comes from:

```python
# From the python standard library
import json
from pathlib import Path

# From third party libraries
import requests

# From local scripts
from utilities.ocsf import OCSF
```

Rules:

- Use these exact comment strings (case as shown):
  - `# From the python standard library`
  - `# From third party libraries`
  - `# From local scripts`
- Omit a section only when that group has no imports.
- Keep `from __future__ import annotations` at the very top (after the module docstring), before the labeled groups.
- Within each group, prefer standard isort-style order (stdlib / third party / local already separated by the labels).
- Path bootstrap for local imports (e.g. adding the repo root to `sys.path`) belongs with the local-scripts section and should be commented so the next reader knows why it exists.
- **Do not rename imports to single-letter (or other cryptic) aliases.** Write `import requests`, not `import requests as r`. Use the full module name (or a short, readable multi-letter alias only when the package docs require it, e.g. `import numpy as np` is an established convention — single letters like `r`, `t`, `s` are not).

### 5. Comment and document for the next reader

- Module docstring: what the file is for and any non-obvious constraints.
- Class docstring: responsibility of the type.
- **Before every function/method declaration**, put a one-line comment that briefly states what it does
  (e.g. `# Declare a function to run the class workflow`).
- Method/function docstrings: start with a short purpose sentence, then **Args** and **Returns** (and
  exceptions when non-obvious). Example shape:

  ```text
  """Emit a summary; return non-zero on failure.

  Args:
      paths: a list of input files to process

  Returns:
      An integer status code
  """
  ```

- **Comment code more heavily** so another engineer or agent can follow the flow without reverse-engineering:
  - Before each logical step inside a method (setup, validate, transform, call out, handle errors, return).
  - When assigning non-obvious values, building request payloads, branching on status codes, or mutating shared state.
  - When relying on an API quirk, env var, path layout, or ordering constraint.
  - Prefer short full-sentence or clear phrase comments over cryptic fragments.
- Avoid noise that only restates the syntax (`# increment i` on `i += 1`). Comment **intent and sequence**, not the Python keyword.
- Write so the next user or agent can continue where you left off.

### 6. Keep functions and methods short

- One clear responsibility per method.
- Prefer small, readable methods over long multi-phase functions.
- Extract helpers (as methods or nested helpers on the class) when a block has its own nameable purpose.
- Avoid deep nesting; early returns and guard clauses are preferred.

### 7. Lint with pylint and flake8 after writing code

After any non-trivial Python change, run both tools on the files you touched:

```bash
pylint --max-line-length=100 <files>
flake8 --max-line-length=100 <files>
```

**Project style that both tools must respect:**

| Setting | Value |
|---------|--------|
| Max line length | **100** characters |
| Blank lines between methods inside a class | **one** blank line |
| Blank lines between top-level definitions | **two** blank lines (classes, module-level functions, constants blocks per PEP 8 / flake8) |

- **Fix or resolve lint findings first.** Do not silence them by default.
- Prefer changing the code (refactor, rename, narrow exception types, restructure imports, split long lines) over disabling a rule.
- **Do not arbitrarily disable lint errors** with `# pylint: disable=...` or `# noqa: ...` just to clear a report.
- A local disable is a last resort only when:
  1. You tried a real fix and it is worse or impossible (e.g. a false positive, or a required `sys.path` bootstrap that triggers import-order noise), **and**
  2. The disable is as narrow as possible (one line or one rule), **and**
  3. A short comment next to the disable explains **why** the exception is intentional.
- Never disable rules globally (file-wide or project-wide) to paper over issues you have not evaluated.
- If pylint and flake8 disagree on a **major** rule and you cannot satisfy both without a weak compromise, **stop and present the conflict to the user** with both messages and a recommended choice. Do not silently pick one side or blanket-disable either tool.

### 8. Prefer native data structures and efficient code

- Use built-in types and stdlib containers first: `list`, `dict`, `set`, `tuple`, `deque`, `defaultdict`, `Counter`, generators, comprehensions where they stay readable.
- Choose the structure that matches access patterns (membership → `set`; ordered unique keys → `dict`; FIFO → `deque`).
- Avoid premature micro-optimization, but do not write clearly inefficient patterns (repeated linear scans when a map works, building huge intermediate lists when a generator suffices).
- Prefer clear O(n) algorithms over clever O(n²) shortcuts when data may grow.

### 9. Keep f-strings simple

- Write `{value}` in f-strings. Do **not** use conversion flags `!s`, `!r`, or `!a` — they are redundant.
- Prefer `f"slug={slug}"` over `f"slug={slug!r}"` or `f"slug={slug!s}"`.
- Format specs (`{count:d}`, `{ratio:.1f}`) are fine when they add real formatting.

### 10. Let the caller handle exceptions

- **Default: do not catch exceptions** in library/class methods. Let them propagate so the caller decides how to handle them.
- Do not swallow errors by returning `None`, `{}`, or `[]` after a `print`.
- A `print` for visibility is fine **if you then re-raise** the same exception (or raise a more specific one).
- Do not catch-and-stop unless the caller layer (for example `app.py`) is the place that owns the process exit.
- Catch only when you can **add context and re-raise** (`raise RuntimeError(...) from error`) or translate to a more specific domain error the caller still sees.
- Document unexpected failures under `Raises:` in the docstring.

### 11. Map external API calls to their official docs

When a module talks to an external HTTP/RPC/SDK API, put the API documentation **in the code** so a reader can map each call to the vendor endpoint without leaving the file.

- **Module (or class) docstring:** official docs URL for the API this file uses, plus any version or product constraint that is not obvious.
- **Immediately above every function/method that performs an external API call:** a comment with the HTTP method (or RPC name), the path or operation template, and the docs URL for that endpoint.
- Keep the comment next to the call, not in a distant README. One endpoint per comment; do not paste the vendor’s full manual.
- If one method hits several endpoints, comment each call site.
- Internal helpers that only format payloads or parse responses do not need an endpoint comment unless they themselves perform the request.
- **Prefer bulk / batch / collection endpoints** when the vendor documents one. Use an org-wide list, multi-get, or batch write instead of one request per item (for example `GET /orgs/{org}/dependabot/alerts` over `GET /repos/{owner}/{repo}/dependabot/alerts` in a repo loop). Confirm the bulk path in the official docs and comment it the same way as any other call. If no bulk endpoint exists, paginate the collection API; do not invent a batch contract the vendor does not publish.

Example shape:

```python
# GET /orgs/{org}/dependabot/alerts
# https://docs.github.com/en/rest/dependabot/alerts#list-dependabot-alerts-for-an-organization
def get_dependabot_alerts_for_org(self, org_name: str | None = None) -> list:
    """Get Dependabot alerts for all repositories in an organization."""
    ...
```

### 12. Function-call map for every class

When you add or change classes, create or update a **function-call map** that covers **every class** in the modules you touched (and any class whose callees changed).

- **Where:** `docs/function-call-map.md` when the project has a `docs/` directory; otherwise `function-call-map.md` at the project root. One file for the project, updated in place.
- **What:** one section per class. List the public entry methods. Indentation is nesting: a child line runs inside the parent. Include `__init__` when the constructor calls other methods. Mark per-item loops and external I/O (`requests`, `urllib`, SDK clients).
- **Keep it true:** if you add, rename, remove, or re-nest a method, update that class’s tree in the same change. Do not leave a stale map.

Example shape:

```text
ExampleService.run()
  ├─ ExampleService._collect()
  └─ ExampleService._report()
```

## Implementation workflow

1. **Understand scope** — files, APIs, and constraints before editing.
2. **Design with classes** — identify types, responsibilities, and public methods.
3. **Implement** — stdlib-first, labeled imports, short methods, documented modules/classes/public APIs, step-by-step comments; comment method + path + docs URL on every external API call; prefer documented bulk/collection endpoints over per-item loops; no module `main` / `__main__` entry point; let exceptions propagate; keep f-strings free of `!s`/`!r`/`!a`.
4. **Self-check** — dependencies justified; import groups labeled; comments make the flow easy to follow; each external API call maps to its docs; bulk endpoints used when they exist; function-call map covers every class you touched; native structures used well; module remains importable; failures bubble to the caller.
5. **Lint** — run pylint and flake8 with line length 100; resolve issues or escalate major tool conflicts to the user.
6. **Verify** — run project-root `test.py` with pytest (when present) or another minimal smoke path, and confirm a broken assertion actually fails.

## Style quick reference

```python
"""Module purpose and non-obvious constraints."""

from __future__ import annotations

# From the python standard library
import logging
from pathlib import Path

# From third party libraries
# (none in this example — omit the section when empty)

# From local scripts
# (none in this example — omit the section when empty)


LOGGER = logging.getLogger(__name__)


class ExampleService:
    """Owns one clear responsibility for the feature."""

    # Declare a constructor to store the working root path.
    def __init__(self, root: Path) -> None:
        """Initialize with the working root path.

        Args:
            root: directory to scan for input files

        Returns:
            None
        """
        # Keep the scan root on the instance for later helpers.
        self._root = root

    # Declare a helper to gather input files from disk.
    def _collect(self) -> list[Path]:
        """Gather inputs from disk.

        Args:
            None

        Returns:
            Sorted list of matching file paths
        """
        # Sort so later steps see a stable, deterministic order.
        return sorted(self._root.glob("*.txt"))

    # Declare a helper to emit a summary and status code.
    def _report(self, paths: list[Path]) -> int:
        """Emit a summary; return non-zero on failure.

        Args:
            paths: a list of input files to process

        Returns:
            An integer status code
        """
        # Empty input is a soft failure for the caller to handle.
        if not paths:
            LOGGER.warning("no input files under %s", self._root)
            return 1

        # Success path: record how much work completed.
        LOGGER.info("processed %d files", len(paths))
        return 0

    # Declare a function to run the class workflow.
    def run(self) -> int:
        """Execute the main workflow.

        Args:
            None

        Returns:
            Process-style integer exit code
        """
        # Step 1: collect inputs from disk.
        results = self._collect()
        # Step 2: summarize and return a process-style status code.
        return self._report(results)
```

Optional tests belong in a separate project-root file (not in the class module):

```python
# test.py  (project root)
"""Project smoke tests for library modules."""

from pathlib import Path

from example_module import ExampleService


# Declare a function to smoke-test ExampleService offline.
def test_example_service(tmp_path: Path) -> None:
    """Smoke-test ExampleService construction and run path.

    Args:
        tmp_path: pytest temporary directory

    Returns:
        None
    """
    # Seed one input so the success path is the expected outcome.
    (tmp_path / "sample.txt").write_text("ok", encoding="utf-8")
    status = ExampleService(tmp_path).run()
    # pytest ignores return values; assert is what fails the test.
    assert status == 0
```

Note: private methods (`_collect`, `_report`) sit above public methods (`run`). **One blank line between methods inside a class; two blank lines between top-level definitions.** Label import groups with the standard-library / third-party / local-scripts comments. Comment before every function; use step-level comments inside methods; docstrings include purpose, Args, and Returns. External API modules put the docs URL in the module docstring and method + path + docs URL above each request; prefer a documented bulk/collection endpoint over one call per item. Keep a function-call map for every class (`docs/function-call-map.md` or project-root `function-call-map.md`). Keep line length ≤ 100. **No `main()` / `if __name__ == "__main__":` in class modules.** **No tests on the class or at the bottom of class modules** — put optional tests in project-root `test.py`. Tests **assert** expected results (pytest ignores boolean returns). F-strings use `{value}` without `!s`/`!r`/`!a`. Library methods let exceptions propagate to the caller.

## Checklist (before declaring done)

- [ ] No unnecessary third-party imports; new deps justified
- [ ] Imports grouped and labeled: standard library / third party / local scripts
- [ ] No single-letter / cryptic import aliases
- [ ] Logic organized in classes with clear responsibilities
- [ ] Private methods above public methods within each class
- [ ] No `main()` / `__main__` entry point; module is importable only
- [ ] Optional tests live in project-root `test.py` (not on the class, not in the class module)
- [ ] Tests fail with `assert` (or a raised exception); they do not return a boolean
- [ ] Comment above every function/method; docstrings include purpose, Args, Returns
- [ ] External API calls include docs URL plus method and path so code maps to the endpoint
- [ ] Documented bulk / batch / collection endpoints used instead of per-item calls when they exist
- [ ] Function-call map covers every class you touched (`docs/function-call-map.md` or project-root file)
- [ ] Step-level comments inside methods make the flow easy to follow
- [ ] Methods are short and single-purpose
- [ ] Native data structures used where they fit; no needless inefficiency
- [ ] F-strings use `{value}` only — no `!s`, `!r`, or `!a`
- [ ] Exceptions propagate to the caller by default (no swallow / empty-return)
- [ ] Lint issues fixed first; no arbitrary `pylint: disable` / `noqa` without a justified last-resort comment
- [ ] `pylint --max-line-length=100` clean (or remaining issues escalated)
- [ ] `flake8 --max-line-length=100` clean (or remaining issues escalated)
- [ ] Line length ≤ 100; one blank line between methods, two between top-level defs
- [ ] Smoke test path exercised when feasible (`pytest test.py` or equivalent)
