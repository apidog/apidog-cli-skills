---
name: apidog-cli-checkup
description: "Apidog CLI usage check and version confirmation: use when commands succeed but the client does not show resources, create results cannot be found by list/get, test runs fail, reports are missing, agentHints/help/runtime behavior disagree, or the local CLI may be outdated."
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog --help; apidog update --help; apidog cli-schema --help"
---

# CLI Usage Check And Version Confirmation

> Prerequisite: read `../apidog-cli/SKILL.md` first. If the stable entry skill conflicts with this skill, use current `apidog <command> --help` and this skill as the source of truth. Then read the corresponding domain skill based on the resource type.

This skill is for public CLI troubleshooting and does not rely on internal APIs or internal code. The goal is to confirm command, project, branch, environment, resource ID, report location, and CLI version, then return to the domain skill to fix resource structure. Agents should close the loop with current help, schema validation, get readback, and agentHints; do not trust summary text alone.

## When To Use

- CLI returns success, but the Apidog client does not show the resource or displays it incompletely.
- A resource cannot be found by `list/get` after creation.
- Test cases, test scenarios, or test suites fail to run.
- Local or cloud reports are missing or have no step details.
- `agentHints`, help, examples, or actual command behavior conflict.
- A user or tester says a parameter is missing or a command is unknown, and the CLI version needs confirmation.

## First Five Checks

1. Record the original command, projectId, branch, resourceId, environmentId, file path, and whether `--api-base-url` was used.
2. Run `apidog --version` to confirm local CLI version; if a parameter is missing, check whether an update is needed.
3. Run the matching `apidog <command> --help`; use current public help, not old docs or memory.
4. Use the matching `list/get` command to read back the resource and confirm project, branch, module, folder, or category.
5. For run or report issues, distinguish local reports from cloud reports. Without `--upload-report`, do not expect cloud report lists to contain the local run.

## Version Check

Start with version and command help:

```bash
apidog --version
apidog import --help
apidog test-case category --help
```

If a required new parameter is missing from help, update the CLI first:

```bash
apidog update
```

For non-interactive environments or confirmed direct updates:

```bash
apidog update --yes
```

If automatic update prompts interfere with diagnosis, users can disable the daily check in shell configuration. This does not disable manual `apidog update`:

```bash
export APIDOG_CLI_DISABLE_UPDATE_CHECK=1
```

Version diagnosis must include current `apidog --version`, command path such as `which apidog`, the missing parameter name, and the recommended update method.

## Help And Hint Conflicts

- When public docs, agentHints, historical examples, and actual behavior conflict, prefer current `apidog <command> --help` and real tests.
- Do not proactively recommend hidden aliases that are not public in help.
- When `success=false`, trust the actual `success` field and exit code, not a successful-sounding summary.
- If a hint suggests a next step but the parameter does not exist, record it as a CLI hint issue and use the public alternative from help.

## Resource Missing In Client

Check first:

- Correct project.
- Correct `--branch`.
- Whether the resource is on an AI branch while the client is viewing another branch.
- Expected module or folder.
- Valid `categoryId` for test cases.
- For native-format second imports, whether module import strategy placed resources into a new module.

Useful readback commands:

```bash
apidog endpoint list --project <projectId> --branch <branchName>
apidog test-case list --project <projectId> --endpoint <endpointId> --branch <branchName>
apidog test-scenario get <scenarioId> --project <projectId> --branch <branchName> --with-case-detail
```

If `get/list` can see the resource but the client cannot, first confirm client filters, branch, module, folder, and category. Do not recreate the resource immediately.

## Test Case Diagnosis

- Before creation, get a valid `categoryId` with `apidog test-case category --project <projectId>`.
- Current `test-case category` does not support `--endpoint`; to view endpoint cases, use `test-case list --endpoint <endpointId>`.
- `test-case get` seeing a structure only means it was saved; it does not prove requestBody, assertions, extractors, or scripts can run.
- On run failure, check environment, variables, request body, pre/post scripts, assertions, and report details.

## Test Scenario Diagnosis

- `test-scenario create` only creates scenario metadata; complex steps require follow-up `import-steps`, `add-ref`, or `update --file`.
- After creation or update, run `test-scenario get --with-case-detail` to confirm step tree and HTTP details expand correctly.
- If step-to-step variables are empty, check full scenario execution, step number, response path, extractor, and selected environment.
- Do not write `test-case` structures directly as `test-scenario` steps.

## Run And Report Diagnosis

- If `--environment` is not specified, the server may use project defaults. Use an explicit environment for reproducible diagnosis.
- Local reports are controlled by `--out-dir` and `--out-file`.
- Cloud `test-report list/get/download` can see the run only when execution used `--upload-report`.
- If reports have no step details, compare local JSON and cloud report first. If local details are also missing, inspect the run target and resource structure.
- Do not run side-effecting tests against production by default.

## Common Routing

| Symptom | Action |
|---------|--------|
| New parameter is unknown | Check `apidog --version`, `which apidog`, and update with `apidog update --yes` if needed |
| Create succeeded but client does not show it | Check project, branch, module, folder, category, and client filters |
| Test case is invisible in client | Check `categoryId`, endpoint, branch; read back with `test-case list --endpoint` |
| Scenario steps are not displayed | `test-scenario get --with-case-detail`; confirm steps were written after create |
| Cloud report is missing | Confirm the run used `--upload-report` |
