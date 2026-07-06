---
name: apidog-import-export
description: "Apidog import/export and quality gates: import OpenAPI, Postman, Swagger, and Apidog native formats; export OpenAPI, HTML, Markdown, Postman, and Apidog native formats; validate spec completeness, tag grouping, schema/body coverage, counts, modules, and resource visibility."
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog import --help; apidog export --help"
---

# Import, Export, And Quality Gates

> Prerequisite: read `../apidog-cli/SKILL.md` first. If the stable entry skill conflicts with this domain skill, use the current CLI help and this skill as the source of truth.

Always use current CLI help for exact arguments. Before import, check file format, OpenAPI quality gates, route-skeleton risk, tag grouping, native-format module strategy, ignoreCount, and whether a clean validation project is needed. After import, agents must verify results with agentHints, list/get, and, when relevant, run/report. Do not treat an import command returning success as complete by itself.

## When To Use

- Generate API specs from code, PRDs, requirement documents, test documents, or discussions and import them into Apidog.
- Import OpenAPI, Postman, HAR, Apidog native format, or other project data.
- Configure automatic import.
- Export OpenAPI, Markdown, HTML, Postman, or Apidog native format.
- Determine whether a generated spec is only a route skeleton.
- Migrate, back up, or copy Apidog projects, especially when module reuse or new module creation must be controlled.

## When Not To Use

- Import or copy existing endpoints, test cases, or scenario steps into a test scenario: use `apidog-test-scenario` with `test-scenario import-steps` or `test-scenario add-ref`.
- Precisely maintain scenario steps, variables, assertions, or processors: use `apidog-test-scenario`.

## Command Entry Points

Use current CLI help for `import`, `import auto-import`, `export`, and `export settings`. Do not skip the quality gates below before importing.

## Core Principles

1. Search for existing generators in the project before writing extraction scripts.
2. Do not mistake complete paths for a complete API spec.
3. Output quality metrics before import.
4. Validate both completeness and readability.
5. If import strategy is uncertain, validate in a temporary or revised project first instead of polluting an existing project.
6. Large `ignoreCount` values in import results are risk signals, not ordinary success.
7. Apidog folder grouping for OpenAPI depends on operation tags.
8. Do not assume file extension equals actual format.

## Standard Flow

### Step 0. Identify The Task Type

First decide which task the user is asking for:

| Task | Next step |
|------|-----------|
| Generate spec from code, PRD, or docs and import | Continue to Step 1 |
| Directly import an existing file | Start from Step 3 |
| Configure automatic import | Read current CLI help and schema, then create the configuration |
| Export docs or OpenAPI | Use export directly, then confirm the file was generated |
| Migrate or back up an Apidog project | Use Apidog native export/import and confirm module import strategy |
| Import existing resources into a test scenario | Switch to `apidog-test-scenario`; do not use OpenAPI import |

If the user did not specify target project, team, import strategy, or whether creating a new project is allowed, infer from context when safe. If uncertain, ask one minimal necessary question.

### Step 1. Search For Existing Generators

For requests like generating API specs, docs, or types from source code, docs, PRDs, or tests, first search the project for existing generators:

```text
openapi
swagger
routegen
route-gen
docs generator
api docs
schema generator
cmd/openapi
cmd/*openapi*
```

Correct direction:

- Run the project's own generator first.
- Prefer tools that extract handler request/response structs, DTOs, and schemas.
- Do not start by hand-parsing router files; that often produces only method + path skeletons.

If no generator exists, consider generating from framework routes, comments, DTOs, schemas, tests, or docs. Still continue through Step 3 and later quality gates.

### Step 2. Generate Spec And Confirm Format

- Run the project generator or official command.
- Save the original artifact; do not overwrite a user-owned file directly.
- Do not judge format by extension.
- Before reading a generated file, inspect the beginning or parse it with JSON/YAML-capable tooling.
- Generators may write YAML to a `.json` file; if so, rename it to `.yaml` or parse it as YAML.

### Step 3. Collect Pre-Import Metrics

Before import, parse the generated OpenAPI file and report real metrics. Do not use example, default, or placeholder values.

Required metrics:

| Metric | Meaning | Purpose |
|--------|---------|---------|
| `paths` | Number of OpenAPI paths | Estimate API scale |
| `operations` | Actual operation count | Estimate import scale |
| `schemas` | Number of `components.schemas` | Estimate model completeness |
| `writes` | POST/PUT/PATCH and similar write APIs | Body coverage target |
| `withBody` | Write APIs with requestBody | Body coverage |
| `emptyObjectBodies` | Request body schema is empty object | Route-skeleton risk |

Output real statistics from the current file, as a table or JSON. Every value must come from parsing the file.

### Step 4. Judge Spec Completeness

- Do not use a fixed schema count as the quality threshold.
- Small, pure-GET, health check, webhook passthrough, or non-JSON-body projects can be acceptable with `schemas=0/1`.
- Large projects with many APIs and write operations but very few schemas are likely route skeletons.
- Many POST/PUT/PATCH bodies missing or represented as `{}` are strong risk signals.
- Method + path alone does not prove requestBody, response, or schema completeness.

Risk signals:

- Many operations but almost no schemas.
- Write APIs without requestBody.
- Request bodies are empty objects.
- Responses are only generic strings or empty objects.
- Tags are missing or too generic, causing poor folder grouping.

When risk is high, explain the risk and recommend improving the generator/source annotations before import, or import into a temporary project for visual validation.

### Step 5. Import

Use current CLI help for the exact format and arguments. Examples of command shapes:

```bash
apidog import --project <projectId> --format openapi --file <openapiFile>
apidog import --project <projectId> --format postman --file <postmanFile>
apidog import --project <projectId> --format apidog --file <nativeFile> --module-import-mode match-name
```

For native Apidog project files, choose module strategy deliberately:

- Default `match-name`: reuse a target module only when the source module name uniquely matches one target module; otherwise create a new module.
- `new`: create new modules for source modules not covered by `--module-map`.
- `--module-map`: explicitly map a source module to a target module ID, `default`, or `new`.

Prefer target module IDs in `--module-map` to avoid same-name ambiguity.

### Step 6. Verify Import Results

After import, verify more than the success flag:

- Check `success` and exit code.
- Inspect imported counts, ignored counts, and warnings.
- Use `endpoint list`, `schema list`, folder/module views, or relevant commands to confirm resource visibility.
- Confirm APIs landed in expected modules/folders.
- If importing test resources, verify test cases, scenarios, suites, and categories in their own domain commands.
- If the client display matters, open the same project/branch/module/category context or use CLI readback to confirm exact placement.

If `ignoreCount` is high or resources are missing, do not repeat the same import blindly. Diagnose format, module strategy, unsupported resource types, and warning details first.

## Export Rules

Use current CLI help for supported formats and options. After export:

- Confirm the output file exists.
- Inspect whether it is valid JSON/YAML/Markdown/HTML as expected.
- For OpenAPI export, confirm whether extension properties are required.
- For native export, use it for migration/backup and pair it with an explicit import strategy.

Example command shapes:

```bash
apidog export --project <projectId> --format openapi --output <file>
apidog export --project <projectId> --format markdown --output <file>
apidog export --project <projectId> --format apidog --output <file>
```

## Common Recovery

| Symptom | Action |
|---------|--------|
| Import succeeds but APIs are missing | Check warnings, ignoreCount, module strategy, and list/get results |
| APIs are grouped poorly | Check OpenAPI operation tags |
| Generated spec has only paths | Search for a better generator or enrich DTO/schema extraction |
| Native import creates duplicate modules | Use `--module-map` or `--module-import-mode match-name` carefully |
| Export file is unusable | Confirm format, output path, and whether the file is JSON/YAML/HTML/Markdown as expected |
