# Bash

Follow the [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html). The [shared principles](code.md) say what to document; this page says where.

If a script grows past a couple of hundred lines, or needs data structures beyond simple arrays, consider writing it in Python.

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Script files | `kebab-case.sh` (executables may drop `.sh`) | `install-health-monitor.sh` |
| Functions | `snake_case` | `check_sync_timer` |
| Local variables | `snake_case`, declared with `local` | `local next_trigger` |
| Constants and exported variables | `UPPER_SNAKE_CASE`, `readonly` where possible | `readonly CONFIG_ROOT` |
| Library-internal functions | a package prefix, `::` or `_` | `hm::emit_metric`, `hm_emit_metric` |

Indent with 2 spaces, never tabs. Keep lines under 80 characters, and break long pipelines one stage per line.

## File header

After the shebang and `set -euo pipefail`, every script starts with a comment saying what it does, and a `usage` function if it takes arguments.

```bash
#!/usr/bin/env bash
#
# Installs the Ubuntu health monitor, its systemd timers and its
# logrotate policy. An existing /etc/health-monitor/config.json is kept
# unless --force-config is given.

set -euo pipefail
```

## Function header

Google's function-comment sections (`Globals:`, `Arguments:`, `Outputs:`, `Returns:`) match the manual's Results almost exactly. We add `Side effects:`.

```bash
#######################################
# Report whether a path is a responsive mount point.
# Globals:
#   OPERATION_TIMEOUT_SECONDS
# Arguments:
#   Path that should be a mount point.
# Outputs:
#   Nothing on success; a one-line reason on stderr otherwise.
# Returns:
#   0 if mounted and responsive, 1 if not a mount point, 2 if the
#   mount did not answer within the timeout (usually a stale NFS mount).
# Side effects:
#   None. Probes run under timeout(1), so a hung mount cannot hang the
#   caller.
#######################################
mount_status() {
  ...
}
```

Exit codes are part of the interface. If a function or script uses more than "zero for success", list every code.

## Practices that matter in our scripts

- Quote every expansion: `"$var"`, `"${array[@]}"`.
- Use `[[ ... ]]` and `(( ... ))`, not `[ ... ]` or `expr`.
- Put any command that can hang (anything touching a mount, a network service, or `virsh`) under `timeout`.
- Make side-effecting scripts support `--dry-run`, and test it.
- Never parse `ls`. Use globs, `find -print0`, or `stat`.
- Wherever the system state is tested, it can be faked: call commands by name, not absolute path, so tests can put fakes first on `PATH`.

## Tools

- **shellcheck**: every warning is fixed or disabled at that one spot, with a comment saying why.
- **bats** for tests. Keep tests in a `tests/` directory next to the scripts, with one `.bats` file per script or library. Use fake commands on `PATH` to simulate `systemctl`, `mountpoint` and similar commands.
