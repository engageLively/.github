# Testing: shared principles

Section numbers (§) refer to the [Engineering Manual](https://www.tcl-lang.org/doc/engManual.pdf). Each language page names its test framework. This page says what the tests must do.

## 1. Tests live in files and run on every change (§7)

If you checked something by hand, turn it into a test. A test typed into a terminal is lost once it has run. Every PR that changes behavior adds or updates tests, and CI runs them all.

## 2. Make tests easy to find (§7.2)

A test file named after its source file is a good default (`sync_timer.py` → `test_sync_timer.py`). Group tests by feature or workflow when that makes the behavior easier to find and understand. Use descriptive names and follow the repository's existing layout.

## 3. Tests must tell correct from incorrect, not just run every line (§7.3)

Test the behavior a caller or operator relies on, including failure paths. Use these cases where they can uncover a bug:

- Exercise branches that change results, side effects, or recovery behavior.
- Try zero, one, and multiple items when those cases can behave differently.
- Test boundary positions in a collection when position matters.
- Every test asserts something specific. A test that only checks "did not crash" is rarely enough.

Coverage is reported on each PR, but there is no minimum percentage. A percentage target rewards tests that run code without checking it. Instead, reviewers look at the lines in the diff that no test covers and ask why.

## 4. Reproduce the bug before fixing it (§7.5)

A bug means the test suite was incomplete. Write a test that fails because of the bug, watch it fail, then fix the code and watch it pass. The ticket describes the bug, and the PR includes the test. Then ask what kind of test would have caught it earlier.

## 5. Protect tricky code (§7.6)

If code is deliberately not the obvious version, write a test that fails when someone "simplifies" it back to the obvious version. The test explains the code to the next reader, and the comment says why.

## 6. Tests are independent (§7.7)

Each test can run alone and be understood alone. Set up once, then run a group of tests against that setup. Don't build a chain where each test changes the setup for the next one.

## 7. Give hard-to-reach code a way in (§7.3)

If a code path can't be reached from outside, add a hook that exists only for testing, the way Tcl's `tclTest.c` adds commands only to the test build. Today this usually means injection. Pass in the command runner, the clock, the filesystem root and the config path rather than reaching for globals. Code designed this way is easier to test and easier to reason about.

## Beyond unit tests

Our code controls real machines, and most of our failures come from the environment: a hung mount, a timer in an unexpected state, a full disk, a corrupt cache. Unit tests alone don't catch those. Use these as well:

### Fault injection

Put fake versions of external commands (`systemctl`, `mountpoint`, `virsh`, `rclone`) first on `PATH`, and run the real code against sandboxed configs and directories. Test each way the environment can fail, not just the healthy case. Keep the harness in the repository next to the tests, so the next change can reuse it.

### Comparison against the previous version

When a change claims to preserve behavior (a refactor, a port, a rewrite), run the old and new versions on the same inputs and compare their outputs. Pin the old version so the comparison doesn't drift.

### Canary host before merge

A change to production monitoring, deployment or data shipping runs on one real host before merge. The steps:

1. Choose a host with no active tests, and tell the owner.
2. Record a baseline from the current version.
3. Deploy the PR's exact commit with the normal deploy tooling.
4. Compare the results, leave it running for a day or so, and post the results on the PR.

The host is now running code that isn't on main. Record that, so the next fleet-wide deploy doesn't silently roll it back, and redeploy the host when the PR merges.

### CI blocks the merge

A test that can't stop a bad change from merging doesn't protect anything. All tests and linters run in CI, and a failure blocks the merge.

## What tests can't cover

Some things can't be fully faked: lab instruments, VMs under real load, storage-appliance firmware. Fault injection and canary hosts reduce that risk; they don't remove it. When a PR's behavior depends on something its tests can't exercise, the PR description says so.
