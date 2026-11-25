# engageLively Engineering Style Guide

> **Draft for discussion.** Nothing here is in force yet. Comments welcome, especially from the people who will be writing to it.

These conventions are built on John Ousterhout's [*Tcl/Tk Engineering Manual*](https://www.tcl-lang.org/doc/engManual.pdf) (1994), which grew out of the Sprite operating system project at Berkeley. Read it. It is 23 pages, and most of what follows is that manual applied to the languages we use today. The Tcl C source code is the best demonstration of the manual in practice: open any file and see how far you get without having seen it before.

## Four rules that apply everywhere

1. **We all do things the same way.** The manual says the exact conventions matter less than everyone following the same ones. Nobody here claims these are the best possible rules; they are the rules we follow.
2. **Don't be weird.** Every language community has its own conventions for naming, layout, where documentation goes, and which tools check it. We follow them. Python code should look like Python, PowerShell like PowerShell. So there is no single set of syntax rules. There is one page of shared principles, plus one page per language that maps those principles onto that language's conventions.
3. **Document the contracts callers rely on.** Public interfaces and operations with meaningful side effects need a summary, results, and side effects in the same PR as the code. A short note is enough for an obvious private helper when it adds useful context. Missing or misleading documentation for a required contract blocks review.
4. **Every branch and PR has a ticket.** The issue describes the problem the change solves, and the PR links it.

This file lives in the organization's `.github` repository, so GitHub shows it on every engageLively repository that doesn't have its own `CONTRIBUTING.md`. A repository can add its own conventions by adding a `CONTRIBUTING.md` that links back here.

## Pages

| Page | Covers |
| --- | --- |
| [Code](style-guide/code.md) | Shared principles: interface documentation, naming, comments, clean code |
| [Testing](style-guide/testing.md) | Shared principles: what tests must do, fault injection, canaries, CI |
| [Python](style-guide/python.md) | PEP 8, Google-style docstrings, ruff, pytest |
| [Bash](style-guide/bash.md) | Google Shell Style Guide, shellcheck, bats |
| [PowerShell](style-guide/powershell.md) | Verb-Noun naming, comment-based help, PSScriptAnalyzer, Pester |
| [Ansible](style-guide/ansible.md) | ansible-lint, argument specs, check mode |
| [JavaScript](style-guide/javascript.md) | Community naming, JSDoc, ESLint |

## Enforcement

A rule that is not enforced is a suggestion. Each language page names the linter that checks what can be checked mechanically, and CI must pass before merge. Whether documentation explains the *why* cannot be linted; that is the reviewer's job.

## Working with coding assistants

Keep a short checklist in the repository's `AGENTS.md` and link to this guide. Include the rules and checks that matter in that repository, such as the lint and test commands. Use CI for mechanical checks and review to judge whether the contracts and tests tell readers what they need to know.

## Applying the guide to existing code

Apply these rules to new code and to the behavior a PR changes. Add or update the contract when the change affects a documented interface or its side effects. Keep existing filenames and command names when callers depend on them. Renaming a published entry point needs its own compatibility plan.

## Open questions

- Do we adopt a dated `CHANGES` log per repository (manual §9.1), with incompatible changes flagged so the next person deploying sees them?
