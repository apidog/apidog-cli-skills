---
name: apidog-test-scenario
description: "Apidog test scenario modeling: query, create, update, delete, and run test-scenario resources; import endpoint/test-case/other scenario steps; add scenario reference steps; orchestrate complex HTTP, condition, loop, wait, script, database, pre/post action, variable, assertion, extractor, and debugging flows."
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog test-scenario --help"
---

# Test Scenario Modeling

> Prerequisite: read `../apidog-cli/SKILL.md` first. If the stable entry skill conflicts with this domain skill, use the current CLI help and this skill as the source of truth. For single-endpoint cases, read `../apidog-test-case/SKILL.md`; for execution, CI, and report boundaries, read `../apidog-test-automation/SKILL.md`. Environment and variable commands follow current CLI help.

Always use current CLI help for exact arguments. When creating or updating test scenarios, focus on step persistence semantics, import/reference boundaries, variable passing, complex step field risks, and run-time verification. Agents should prefer import/reference commands and `get --with-case-detail` to read the real structure before making local updates.

## When To Use

- Create or update test scenarios.
- Design multi-step automated flows such as login -> create resource -> query -> assert -> cleanup.
- Orchestrate step types such as conditions, loops, waits, scripts, database operations, or external programs.
- Handle pre/post actions, variable references, variable extractors, and assertion chains.
- Diagnose scenario step linkage failures, empty variables, wrong step order, or client display issues after scenario creation.

## When Not To Use

- Single-endpoint test cases: use `apidog-test-case`.
- Only running existing scenarios, suites, or CI commands: use `apidog-test-automation`.
- Only viewing reports: use `test-report` from current CLI help; use `apidog-test-automation` for execution and report boundaries.
- Only configuring environments, variables, database connections, or similar runtime context: use environment, variables, database-connection, and related commands from current CLI help.

## Core Boundaries

| Concept | CLI resource | Notes |
|---------|--------------|-------|
| API test case | `test-case` | Case bound to one endpoint; suitable for single-endpoint validation |
| Test scenario | `test-scenario` | Multi-step workflow orchestration |
| Test suite | `test-suite` | Collection of scenarios/cases for regression |
| Environment | `environment` | Provides baseUrl, variables, and service configuration |
| Test data | `test-data` | Iteration data source for scenarios or cases |

## Command Entry Points

Use current CLI help for `test-scenario` parameters. `import-steps` and `add-ref` may not appear in the top-level command list, but each supports `--help` and can be used to import steps or add scenario references.

Important fact: `test-scenario create` only saves metadata. Even if create payload contains `steps`, those steps are not saved during creation. The correct flow is to create metadata first, then run `get --with-case-detail`, then add or modify steps with `import-steps`, `add-ref`, or `test-scenario update --file`.

Prefer high-level import commands for common imports. Do not handwrite complex HTTP binding structures when importing existing resources. Verified entry points include:

```bash
apidog test-scenario import-steps <scenarioId> --project <projectId> --source endpoint --ids <endpointIds> --sync manual
apidog test-scenario import-steps <scenarioId> --project <projectId> --source test-case --endpoint <endpointId> --ids <testCaseIds> --sync manual
apidog test-scenario import-steps <scenarioId> --project <projectId> --source test-scenario --from-scenario <sourceScenarioId> --step-ids <stepIds>
apidog test-scenario add-ref <scenarioId> --project <projectId> --scenario <sourceScenarioId>
```

Semantic boundaries:

- `import-steps` copies/imports steps into the current scenario.
- `add-ref` adds a reference to another scenario; it does not copy the source scenario's internal steps.
- `--sync manual` is the default and is suitable when imported steps need business parameters, variable references, or assertions.
- `--sync auto` applies only to endpoint/test-case sources and means the step syncs with the source API definition or single-endpoint case; test-scenario sources do not support auto sync.
- CLI commands are non-interactive and do not list all resources automatically when IDs are missing. If endpoint, case, scenario, or step IDs are missing, follow agentHints and use list/get first.

Simplified creation must include all required metadata, but it still creates an empty scenario. Unless the user explicitly wants a placeholder, creating an automation test or scenario cannot stop at an empty scenario.

## Standard Modeling Flow

1. Clarify the business goal: what flow to validate, what success means, and how failure should be cleaned up.
2. Confirm project, branch, and environment.
3. List involved endpoints, cases, environment variables, test data, database connections, or scripts.
4. If creating the scenario on an AI or sprint branch, you do not need to pick referenced endpoints or cases when only editing the scenario itself. Pick resources only when you need to modify the referenced endpoint, single-endpoint case, or source scenario.
5. If a similar scenario exists, read it with `test-scenario list/get` as a template.
6. Design the step graph in natural language before converting it to JSON; do not write by guessing fields.
7. Fetch `test-scenario-create` schema and create metadata.
8. After creation, run `test-scenario get --with-case-detail` to read the full structure.
9. When importing existing resources, prefer `import-steps` or `add-ref`; do not handwrite complex binding fields.
10. After import, run `test-scenario get --with-case-detail` again and confirm steps are written and HTTP case/detail expands correctly.
11. After importing API definitions or test cases, check whether params, headers, body, and script variables are only schema examples; update business values with `test-scenario update --file` when needed.
12. For detailed edits, fetch `test-scenario-update` schema and modify a complete readback structure.
13. After `cli-schema validate` passes, update and then run `test-scenario get --with-case-detail` to confirm `steps` is non-empty and structurally correct.

`test-scenario-update` schema includes steps, pre/post processors, assertions, extractors, and enum documentation. When writing processors for the first time, read the schema first; do not guess field names, enum values, or legacy formats.

## Step Design Best Practices

For each step, describe:

```text
stepName: what this step does
stepType: which step type to use
input: request, script, SQL, wait condition, or referenced variables
output: variables to extract
assertions: success criteria
onError: continue, stop, or cleanup behavior
cleanup: whether post-run cleanup is needed
```

For complex flows, use layers:

```text
Prepare data
Authenticate/login
Main business operation
Query and assert result
Verify side effects
Clean up resources
```

## Data Passing And Variable References

- List data sources before creation: environment variables, global variables, previous step request/response, iteration data, or script output.
- Prefer Apidog native previous-step result references for step-to-step data, such as `{{$.1.response.body.token}}` and `{{$.2.response.body.data.id}}`. These work only in full automated scenario runs; running a single step cannot resolve them.
- If the same value is reused across modules or needs a stable name, extract it into a variable and reference it as `{{token}}`.
- Random suffixes, temporary identifiers, or generated values not coming from responses are good candidates for environment variables, such as `pm.environment.set('runSuffix', suffix);`.
- In scripts, do not write `{{...}}` directly for previous-step values; use `pm.variables.get("$.1.response.body.token")`.
- Step references depend on step IDs/numbers. After inserting, deleting, or reordering steps, recheck every `{{$.step...}}` reference.
- Do not assume the response path is always `body.data.id`; verify the real response path, such as `{{$.2.response.body.id}}`, `{{$.2.response.body.data.id}}`, or `{{$.2.response.body.data[0].id}}`.
- Preserve `{{...}}` placeholders exactly in generated payloads. Do not escape, split, or rewrite them, or runtime replacement will fail.
- When variables are empty, first check whether the full scenario was run, step ID, JSONPath, response shape, execution order, and selected environment.

## Update Safety

- Always update from a full `get --with-case-detail` readback when editing existing steps.
- `update` is not JSON Patch and does not merge array entries by ID.
- Keep stable IDs on steps and processors when editing existing structures.
- For existing HTTP/custom/socket steps, preserve internal case objects from readback unless intentionally changing them.
- After update, read back and run the scenario or a targeted dry run as appropriate.

## Common Recovery

| Symptom | Action |
|---------|--------|
| Scenario created but has no steps | Expected if only `create` was used; add steps with `import-steps`, `add-ref`, or `update --file` |
| Imported step uses example values | Update the scenario with business parameters after import |
| Variable is empty | Check full-run execution, step number, response path, extractor, and environment |
| Client does not show steps | `get --with-case-detail`; confirm steps are saved and not empty |
| Need to run or inspect reports | Switch to `apidog-test-automation` |
