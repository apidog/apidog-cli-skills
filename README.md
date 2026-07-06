# Apidog CLI Skills

Apidog CLI AI Agent Skills help AI agents use Apidog CLI correctly for project resource management, API design, test modeling, automation, import/export, and branch collaboration.

## Installation

Recommended installation with the `skills` CLI:

```bash
npx -y skills add https://github.com/apidog/apidog-cli-skills -y
```

You can also install from the Apidog official site:

```bash
npx -y skills add https://apidog.com -y
```

After installation, list installed skills:

```bash
npx -y skills list
```

## Skills

| Skill | Purpose |
| --- | --- |
| `apidog-cli` | Main Apidog CLI entry skill for general CLI usage, project resource management, and tasks that do not yet belong to a more specific domain. |
| `apidog-cli-checkup` | Checks Apidog CLI usage and version issues, including successful commands not visible in the client, missing resources, failed runs, missing reports, and help or agentHints mismatches. |
| `apidog-branch` | Handles Apidog branch collaboration, including regular branches, sprint branches, AI branches, pick-to, merge, merge-request, protected branches, and AI write permissions. |
| `apidog-import-export` | Covers Apidog import/export workflows for OpenAPI, Postman, Swagger, Apidog native formats, quality gates, folder grouping, and result verification. |
| `apidog-test-case` | Handles Apidog API test cases, including test-case, test-data, test steps, assertions, variable extraction, processors, categories, and direct runs. |
| `apidog-test-scenario` | Handles Apidog test scenario modeling, including endpoint/test-case/scenario imports, scenario references, conditions, loops, waits, scripts, database steps, variables, and debugging. |
| `apidog-test-automation` | Covers Apidog automated test execution, suites, and CI, including test-suite, scheduled-task, runner, `apidog run`, execution parameters, and report upload. |
| `apidog-workflow-api-lifecycle` | Provides an Apidog API lifecycle workflow from requirements to API design, schemas, environments, mocks, test cases, documentation export/publishing, and branch merge. |

## Usage Notes

- Use `apidog-cli` first for general Apidog CLI tasks.
- For domain-specific work, load the matching skill, such as `apidog-test-scenario` for scenario modeling or `apidog-branch` for branch collaboration.
- Before write operations, check the current CLI help and JSON Schema, then validate payloads with `cli-schema validate`.
- If a skill conflicts with current CLI output, follow the current CLI output.
