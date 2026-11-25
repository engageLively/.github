# Python

Follow [PEP 8](https://peps.python.org/pep-0008/) and [PEP 257](https://peps.python.org/pep-0257/). Write docstrings in [Google style](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings). The [shared principles](code.md) say what to document; this page says where.

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Modules, packages | `snake_case` | `sync_timer.py` |
| Functions, methods, variables | `snake_case` | `next_trigger_usec` |
| Classes, exceptions | `CapWords` | `CheckResult`, `ProbeTimeout` |
| Constants | `UPPER_SNAKE_CASE` | `DEFAULT_TIMEOUT_SECONDS` |
| Internal to a module | leading `_` | `_parse_show_output` |

## Module docstring

Every file starts with a docstring that names what the module is for. If the module holds design decisions that several functions share, explain them here once (principle 5).

```python
"""Health check: the sync timer is scheduled and firing.

A timer is healthy when it is enabled and either waiting with a valid next
trigger or running. While its service runs, systemd reports SubState=running
and clears NextElapseUSecRealtime, so a missing next trigger is only a fault
in the waiting state.
"""
```

## Function docstring

The docstring goes in the first line of the body. Types go in the type hints, not in the docstring. `Args:` gives each argument's meaning, and `Returns:` and `Raises:` cover the Results. Side effects go in a `Side Effects:` section, which Google style doesn't define. Configure Sphinx's napoleon to recognize it with `napoleon_custom_sections = ["Side Effects"]`.

```python
def restart_sync_service(runner: CommandRunner, unit: str, *, dry_run: bool = False) -> bool:
    """Restart the sync service so that it picks up a changed configuration.

    Called by the installer after it writes a new sync config. Not for use
    as a recovery action: a failing service should be diagnosed, not bounced.

    Args:
        runner: Runs commands with a time limit; injected so tests can fake it.
        unit: Full systemd unit name, e.g. "data-sync.service".
        dry_run: Report what would happen without restarting anything.

    Returns:
        True if the unit is active after the restart (or would be, under
        dry_run); False if systemd reports it failed to start.

    Raises:
        ProbeTimeout: systemctl did not answer within the runner's limit.

    Side Effects:
        Restarts the unit unless dry_run is set, interrupting any copy in
        progress. The service resumes from its queue on its next run, so no
        data is lost, but a large file may be copied twice.
    """
```

Short private helpers whose purpose is obvious from their name and signature can have a one-line docstring. Public functions, and anything that has side effects, get the full form.

These docstrings are also read by tools: `help()`, Sphinx, IDEs, and MCP servers, which use a function's docstring and type hints as the description a model reads before calling it as a tool. A model deciding whether to call `restart_sync_service` needs the Side Effects section more than any human reader does.

## Comments in code

Comments go on their own line above the code they explain, written as full sentences. Don't put explanations at the ends of lines; keep end-of-line comments for short notes such as units (`timeout = 30  # seconds`).

## Tools

- **ruff** for linting and formatting. Select at least `E`, `F`, `I`, `B`, `UP` and `D`, with `[tool.ruff.lint.pydocstyle] convention = "google"`, so that a missing docstring fails CI.
- **Type hints** on all public functions. Running a type checker (mypy or pyright) is encouraged; pick one per repository.
- **pytest** for tests. Use names such as `test_<module>.py` or a feature name that readers can find, and use fixtures for setup. Use `tmp_path` and `monkeypatch` rather than touching the real filesystem or environment.
