---
name: apidog-workflow-api-lifecycle
description: "Apidog API lifecycle workflow: from requirements to API design, schemas, environments, mocks, test cases, documentation export/publishing, and branch merge. Use when creating a group of APIs and completing tests, mocks, or docs."
metadata:
  requires:
    bins: ["apidog"]
---

# API Lifecycle Workflow

> Before starting, read `../apidog-cli/SKILL.md`. If the stable entry skill conflicts with this workflow, use current CLI help and this workflow as the source of truth. Read `apidog-branch`, `apidog-import-export`, and `apidog-test-case` as needed. For complex test flows, read `apidog-test-scenario`; for execution and CI, read `apidog-test-automation`.

Use current CLI help for endpoint, schema, environment, test-report, and related commands. This workflow covers end-to-end delivery from requirements to API assets, tests, reports, and branch merge. Focus on resource order, quality gates, and delivery verification. Agents should build a `help/schema -> get -> validate -> write -> run/report` loop instead of writing from old docs or memory.

## When To Use

- Create or update a set of APIs from requirements.
- Import OpenAPI/Postman and then organize APIs, schemas, folders, and tests.
- Generate an API spec from code, PRDs, requirement documents, test documents, or discussions and import it into Apidog.
- Add mocks, test cases, documentation, and publishing settings to APIs.
- Complete API changes on an AI branch and prepare for merge.

## Workflow

```text
Confirm project/branch
  -> if importing from code/docs, generate spec and run quality gates
  -> design endpoints and schemas
  -> configure environments and variables
  -> configure mocks
  -> create API test cases
  -> run tests and inspect reports
  -> export/publish docs
  -> merge or create MR
```

## Step 1. Confirm Context

Use current CLI help to confirm identity, project, target branch, and whether an AI branch is needed.

- If the user wants to edit main or sprint branches directly, confirm AI write permission or use an AI branch.
- If editing existing resources through an AI branch, import source resources first with `branch pick-to`.

## Step 2. Design API Resources

Use current CLI help for endpoint, schema, response-component, security-scheme, folder, and related API design commands.

Minimum boundary: endpoint is the API definition; test-case is a test case under an endpoint. Do not mix them. Reusable resources such as schemas, response components, and security schemes should be created first, then referenced by endpoints. Environment variables should not be written into common parameters.

If the task is to generate and import an API spec from code, PRDs, or docs, read `../apidog-import-export/SKILL.md` first. Complete generator search, OpenAPI metrics, tag grouping, and temporary project validation before moving into API design or test additions.

- Create reusable `schema`, `response-component`, `security-scheme`, and similar resources before referencing them.
- After creation, always run `endpoint get` to verify the real saved structure.

## Step 3. Configure Environment

Use current CLI help for environment, variables, database-connection, vault, and related runtime context commands.

- Create or update an environment if no suitable one exists.
- Do not expose sensitive variables in final responses.
- For test runs and CI delivery, explicitly pass `--environment` to avoid non-reproducible default environment changes.

## Step 4. Add Test Cases

Read `../apidog-test-case/SKILL.md`.

- Do not create empty-shell cases.
- After creation, run `test-case get` to verify steps, assertions, and extractors were saved.

## Step 5. Validate And Deliver

- For report issues, first distinguish local reports, cloud upload, and downloaded summaries; switch to `apidog-cli-checkup` when needed.
- For client display issues, switch to `apidog-cli-checkup`.
- For merge needs, switch to `apidog-branch`.
- If tests for a new API return 404 or assertions fail, first check mock/environment configuration. A mock configuration issue is not the same as an unsaved API design asset.

## Rules That Must Not Be Violated

1. Do not skip project/branch confirmation.
2. Do not write directly to protected main branches.
3. Do not create endpoints without verifying saved structure.
4. Do not create empty test cases.
5. Do not treat test failure as API creation failure; diagnose API definition, environment, test case, and execution report separately.
