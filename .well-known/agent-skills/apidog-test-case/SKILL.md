---
name: apidog-test-case
description: "Apidog API test cases: query, create, update, delete, categorize, and run test-case and test-data resources; handle test steps, assertions, variable extractors, pre/post processors, datasets, and client display issues after CLI writes."
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog test-case --help; apidog test-data --help"
---

# API Test Cases

> Prerequisite: read `../apidog-cli/SKILL.md` first. If the stable entry skill conflicts with this domain skill, use the current CLI help and this skill as the source of truth. For API definitions, follow current CLI help for endpoint, schema, folder, and related commands. For multi-step flows, read `../apidog-test-scenario/SKILL.md`.

Always use the current CLI help for exact arguments. When creating or updating API test cases, pay special attention to categoryId display risks, the boundary between test-case and test-scenario, requestBody/processor/assertion/extractor structure, and run-time verification. Agents must validate payloads with `cli-schema validate` before writing, then use `get` to confirm the saved structure.

## When To Use

- Create automated test cases under a single endpoint.
- Update test steps, assertions, variable extractors, and pre/post processors.
- List test cases under an endpoint.
- Run API test cases by caseId, endpointId, or categoryId.
- Create or maintain test datasets.
- Diagnose issues such as missing test steps in the client, ineffective assertions, or empty extracted variables.

## When Not To Use

- Multi-endpoint workflow orchestration or scenario modeling: use `apidog-test-scenario`.
- Only running existing scenarios, suites, or CI commands: use `apidog-test-automation`.
- Only editing API definitions, request parameters, or response models: use endpoint/schema commands from current CLI help.
- Only viewing test reports: use `test-report` from current CLI help; use `apidog-test-automation` for execution and report boundaries.

## Core Concepts

| Concept | CLI resource | Notes |
|---------|--------------|-------|
| API test case | `test-case` | Test data and steps bound to an endpoint |
| Test category | `test-case category` | Test case category; use it to obtain a valid `categoryId` before creating a case |
| Test dataset | `test-data` | Data source for iteration runs |
| API definition | `endpoint` | Dependency of a case; not the case itself |
| Test scenario | `test-scenario` | Multi-step workflow orchestration with different boundaries |

## Command Entry Points

Use current CLI help for `test-case`, `test-data`, and `apidog run --test-case`. Use `test-case category` to obtain `categoryId`; current `test-case category` does not support `--endpoint`. To view cases under an endpoint, use `test-case list --endpoint <endpointId>`. `apidog run --test-case` accepts only caseId.

When importing a single-endpoint case into a test scenario, do not handwrite scenario steps in this skill. Switch to `apidog-test-scenario` and use:

```bash
apidog test-scenario import-steps <scenarioId> --project <projectId> --source test-case --endpoint <endpointId> --ids <testCaseIds> --sync manual
```

## Standard Create Flow

1. Confirm project and branch.
2. Locate the endpoint with `apidog endpoint list/get`.
3. Run `apidog test-case category --project <projectId>` to obtain a valid `categoryId`; to inspect existing endpoint cases, run `apidog test-case list --project <projectId> --endpoint <endpointId>`.
4. If a similar case exists, use `list` and then `get` one as a template.
5. Fetch and validate the `test-case-create` schema.
6. Build a complete JSON payload; do not create an empty shell with only name/endpointId.
7. After creation, immediately run `test-case get <caseId>` and confirm the saved backend structure.
8. Run once with a local JSON report to confirm requestBody, processors, assertions, and scripts work in the runner.
9. If uploading reports, use `test-report` according to current CLI help; if report details are missing, use `apidog-test-automation` for local/cloud report boundaries.

`categoryId` is a key required field for client display, not an ordinary optional category. An invalid `categoryId` may make a case visible to CLI `get/list` but invisible or unusable in the client category list. Always obtain a valid ID with `test-case category` before creating a case.

## Standard Update Flow

Before updating, always `get` the original structure and modify that complete structure. Then validate against `test-case-update` schema to avoid losing existing steps, assertions, variable extractors, or processors. `update` is not JSON Patch and does not merge array elements by id.

The `test-case-create` and `test-case-update` schemas include processor, assertion, extractor, and enum documentation. When writing processors for the first time, read the schema first; do not guess field names, enum values, or legacy formats. Current `test-case-update` validates processor `data` by `type`; for example, `extractor` validates `variableType/subject/shareScope`, `assertion` validates `subject/comparison`, and `delay` requires numeric `data`.

`test-case get` may return an empty string for `method`. This does not mean the client displays the HTTP method incorrectly; the client usually displays it from the bound endpoint. Current `test-case-update` schema accepts this readback structure. Do not guess and rewrite the method unless the user explicitly asks to change the bound endpoint or request method.

## Test Step Display Rules

If the user cares about client display, verify all of the following:

1. Run `test-case get` after create or update.
2. Confirm steps, assertions, extractors, and processors were saved by the backend.
3. Confirm fields are not empty arrays, empty objects, or written to the wrong level.
4. If CLI returns success but the client does not display the data, switch to `apidog-cli-checkup` and first confirm project, branch, endpoint, categoryId, and readback structure.

## Content And Processor Structure

- `requestBody.data` must be a string; do not write JSON Body as an object.
- Format multiline JSON bodies, pre-scripts, and post-scripts with `\n`; this improves client readability without changing execution semantics.
- `preProcessors` and `postProcessors` use the flat structure `{ id, type, data, defaultEnable, enable }`; do not use legacy nested `{ type, config }`.
- Prefer stable `id` values for processors, especially `assertion`, `extractor`, and `customScript`; missing IDs may validate but still confuse the runner or client parser.
- When extracting global variables, use `data.variableType: "globals"`; if a scope is needed, prefer `data.shareScope: "PROJECT"`. `TEAM` is team-wide and may depend on paid capabilities; do not default to it unless explicitly requested.
- Always run `cli-schema validate` before writing and `test-case get` after writing.

Example:

```json
{
  "requestBody": {
    "type": "application/json",
    "data": "{\n  \"name\": \"Demo\",\n  \"description\": \"Readable in client\"\n}"
  },
  "postProcessors": [
    {
      "id": "postProcessors.0.customScript",
      "type": "customScript",
      "data": "pm.test('returns id', function () {\n  var body = pm.response.json();\n  pm.expect(body.data.id).to.exist;\n});",
      "defaultEnable": true,
      "enable": true
    },
    {
      "id": "postProcessors.1.extractor",
      "type": "extractor",
      "data": {
        "variableName": "project_pet_name",
        "variableType": "globals",
        "shareScope": "PROJECT",
        "subject": "responseJson",
        "expression": "$.name"
      },
      "defaultEnable": true,
      "enable": true
    }
  ]
}
```

## Running And Verification

- Run the case after writing when execution behavior matters.
- Use explicit `--environment` for reproducibility.
- Generate a local JSON report when diagnosing runner behavior.
- Do not treat client display success as execution success; validate both saved structure and run result when required.

## Common Recovery

| Symptom | Action |
|---------|--------|
| Case created but missing in client | Check project, branch, endpoint, and valid `categoryId` |
| Steps missing after update | Rebuild from full `get` output; `update` does not patch arrays |
| Assertion does not run | Check processor `type`, `data`, `enable`, and generated report |
| Extracted variable is empty | Check subject, JSONPath, response shape, variableType, and shareScope |
| New CLI parameter is missing | Use `apidog-cli-checkup` to verify version and help |
