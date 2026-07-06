---
name: apidog-test-automation
description: "Apidog test automation, suites, and CI: test-suite, scheduled-task, runner, apidog run, execution parameters, iteration data, report upload, and CI regression. Use apidog-test-scenario for complex test-scenario step modeling."
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog test-scenario --help; apidog test-suite --help; apidog run --help"
---

# Test Automation, Suites, And CI

> Prerequisite: read `../apidog-cli/SKILL.md` first. If the stable entry skill conflicts with this domain skill, use the current CLI help and this skill as the source of truth. For complex scenario step modeling, read `../apidog-test-scenario/SKILL.md`; for single-endpoint cases, read `../apidog-test-case/SKILL.md`. Environment, variables, and report commands follow current CLI help. This skill focuses on execution boundaries and common traps.

Always use current CLI help for exact arguments. Before execution, confirm resource boundaries, empty-suite risk, runner/scheduled-task constraints, CI report boundaries, and post-run diagnosis order. In agent workflows, treat run results as acceptance checks. In CI, use exit codes, report files, and upload status as gates.

## When To Use

- Create, update, or run test suites.
- Run existing test scenarios.
- Configure scheduled tasks or CI regression.
- Manage runners or check runner status.
- Work with `apidog run` parameters, reporters, iterations, variable overrides, SSL, timeouts, report upload, or `--carry-runtime-variables`.

## Resource Boundaries

| User intent | Preferred resource |
|-------------|--------------------|
| Single-endpoint test case | `test-case`; switch to `apidog-test-case` |
| Multi-step business flow | `test-scenario`; switch to `apidog-test-scenario` |
| Multi-scenario regression collection | `test-suite` |
| Scheduled execution | `scheduled-task` |
| Private execution machine | `runner` |
| View execution result | `test-report`; query or download by current CLI help |

## Command Entry Points

Use current CLI help for `test-suite`, `scheduled-task`, `runner`, and `run` parameters. `run --help` covers reporters, out-dir, upload-report, iteration, variable overrides, SSL, timeout, and `--carry-runtime-variables`.

`test-suite create --name` creates an empty suite. The client can display it, but `items: []` means it is not a usable regression suite. Unless the user explicitly wants a placeholder suite, creating a regression or automation suite must use `--file` or a follow-up update to add items, then `get` to verify `items` is non-empty.

Non-empty suites should use the frontend-compatible structure from `cli-schema get test-suite-create`, such as `STATIC_TEST_CASE` with `testCases[].id` references. Do not use legacy shorthand such as `{ testScenarioId }`; schema validation intentionally rejects it.

Runners are team-level execution resources. Confirm the team and purpose before creating one. The common practical combination is `runnerType=GENERAL` and `serverType=SELF_HOSTED`; do not treat runners as lightweight project-local resources.

## Create And Update Rules

Use `apidog-test-scenario` for complex test scenario creation or updates.

Do not create scheduled tasks from empty examples. Even if schema required fields appear to be only `name/cronExpression/runOn`, a usable task usually needs context such as a valid runner and a `TEST_SUITE` entityId. `runOn` must use values supported by current CLI help/schema, such as `APP/TSHGR/OSHGR`; do not write unsupported values like `CLOUD`.

Before updating, always `get` the original structure to avoid overwriting steps, variables, scenario references, or suite members.

## Run Parameters

Use `apidog run --help` and specific run command help for current arguments. CI workflows should explicitly confirm environment, reporters, out-dir, upload-report, iteration, variable overrides, timeout, bigint behavior, and whether `--carry-runtime-variables` is needed.

Recommended minimal CI command shape:

```bash
apidog test-suite run <suiteId> --project <projectId> --environment <environmentId> --reporters cli,json,junit --upload-report
```

## After Execution

- Local reports: check `--out-dir` and `--out-file`.
- Cloud reports: only appear after running with `--upload-report`; then use current `test-report list/get/download` help.
- Failure diagnosis: start from CLI output, JSON report, and agentHints, then locate the exact scenario, suite, or case.
- Without `--upload-report`, cloud `test-report list` will not contain this local run.
- In CI, explicitly specify `--environment`; inject tokens through CI secrets, never commit them.

## Common Recovery

| Symptom | Action |
|---------|--------|
| Scenario steps are wrong after creation | Switch to `apidog-test-scenario`; confirm whether steps were updated after create |
| Suite run is empty | `test-suite get` and confirm included scenarios/cases |
| Suite has `items: []` | It is an empty placeholder, not a valid regression suite |
| CI cannot find environment | Use `environment list/get` by current CLI help to confirm environmentId |
| Runner is unavailable | Run `runner check`, then inspect runner get/list |
| Report has no step details | Distinguish local JSON, cloud upload, and downloaded summary; switch to `apidog-cli-checkup` if needed |
