# Ansible

Follow the [Ansible community's good practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html), as enforced by ansible-lint's default profile. The [shared principles](code.md) say what to document; this page says where.

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Roles, variables | `snake_case` | `health_monitor`, `deployment_revision` |
| Role variables | prefixed with the role name | `health_monitor_config_dir` |
| Playbooks | `kebab-case.yml` | `deploy-health-monitor.yml` |
| Modules | fully qualified collection names | `ansible.builtin.systemd_service` |
| Task names | a sentence in the imperative, capitalized | `Render the host-health config for this host` |

## Where documentation goes

- **Playbooks:** a header comment saying what the playbook does, which hosts it targets, which variables it requires, and its side effects (what it installs, restarts or removes). It should also say how to preview it and how to roll it back.
- **Roles:** `meta/argument_specs.yml` documents each input variable: its meaning, type, whether it is required, and its default. Ansible validates role inputs against this file automatically, so the documentation can't drift from what is checked. Give each role a short `README.md` covering its purpose and side effects.
- **Tasks:** the `name:` is the summary, and it appears in every run's output, so make it say what the task changes. Add a YAML comment above any task whose reason isn't obvious, especially `command`/`shell` tasks, `ignore_errors`, and `changed_when`/`failed_when` overrides.

## Side effects are explicit

- Prefer modules to `command`/`shell`. When you must shell out, set `changed_when` so reported changes are honest, and add `creates`/`removes` or a guard so the task is idempotent.
- Support `--check --diff` as a safe preview, and document what it can validate. If a mutating task cannot be previewed, skip it with `when: not ansible_check_mode`. Ansible executes a task with `check_mode: false` normally even under `--check`. Reserve that override for deliberate read-only probes and explain why the probe must run.
- Production plays require an explicit confirmation variable (for example `confirm_production_deploy=true`) and a pinned revision, so nothing reaches production by accident or from an unknown commit.

## Tools

- **ansible-lint** in CI. Every entry in `skip_list`, and every inline `# noqa`, carries a comment saying why.
- **Tests:** each role has a test inventory and playbook under `tests/` that runs it against fixtures. The tests cover a second run that reports no changes (idempotence), and check mode. Before merging a production change, run it on a canary host (see [Testing](testing.md)).
