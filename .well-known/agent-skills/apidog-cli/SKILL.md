---
name: apidog-cli
description: Manage Apidog project resources through Apidog CLI. Trigger scenarios - running API automation tests/test suites, querying/creating/updating/deleting endpoints, environments, schemas, mocks, branches and other project resources, importing/exporting API documentation, viewing test reports, managing runners, scheduled tasks, notifications and CI/CD configuration. CLI output is structured JSON and often includes agentHints.nextSteps; all commands support --help.
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog --help"
---

# Apidog CLI

Use Apidog CLI to complete the user's request. Do not guess payloads from memory; prefer CLI `--help`, `cli-schema`, `cli-schema validate`, and `agentHints.nextSteps` to drive the next action.

## New Session Check

- When first handling an Apidog CLI task, lightly verify CLI availability: `apidog --version`, `apidog --help`.
- If `apidog` is missing or cannot output a version, install the CLI first: `npm install -g apidog-cli@latest`.
- After installation, run `apidog login --with-token <TOKEN>`. Get the API access token from the Apidog app or web console: user avatar → Account Settings → API Access Token.
- Do not upgrade by default every time; suggest upgrade only when a command returns version-too-low / unknown command / parameter issues likely caused by an old CLI, or when the user's task depends on a new capability.

## Basic Usage

```bash
apidog --help
apidog <command> --help
apidog <command> <subcommand> --help
```

Common global options:

```text
--project <projectId>     Project ID
--branch <branchName>     Branch name; pure numeric values are compatible with legacy branchId
--access-token <token>    Override current login token
--api-base-url <url>      Private deployment URL
```

## Login And Project

- If not logged in, ask the user for an API access token and run: `apidog login --with-token <TOKEN>`.
- Apidog CLI token is stored in `~/.apidog/config.toml`; do not print the token in logs, commit it to the repo, or include it in chat summaries.
- If the user does not specify a project, first check whether `.apidog/settings.json` exists in the current workspace. If it contains a commonly used default `projectId`, prefer that project.
- Recommend that the user get the project ID from Project Settings - Basic Settings - Project ID; or first run `apidog project list` and ask the user to confirm the target project for the command.
- Ask the user before writing any local config file.

## Write Workflow

Required: before running `create` or `update`, first get the complete resource definition and data format, then run `cli-schema validate` before the real write:

1. Get schema: `apidog cli-schema get <schemaKey>`
2. Generate a JSON data file.
3. Validate: `apidog cli-schema validate <schemaKey> --file <path>`
4. Only run the real `create` or `update` command after `validate` passes.

After command output, prefer reading `agentHints.nextSteps` in the JSON output to continue or recover.

## CLI Facts First

- Concrete commands, options, and schema keys must follow the current CLI output: `apidog <command> --help`, `apidog cli-schema list`, and `agentHints.nextSteps`.
- If this skill conflicts with the current CLI output, follow the current CLI output and update the factual description in this skill.

## CLI Write Permission Statement

- When writes are blocked by AI (CLI source) permission, do not choose for the user unless they already specified a preference. Ask whether to enable direct edit permission on the target branch, or edit target branch data through an AI branch.
- To directly edit main branch, sprint branch, or general branch data, ask the user to enable direct edit permission in Apidog client 2.8.32+: Project Settings - Feature Settings - AI Feature Settings - External AI Edit Permissions.
- If the user chooses an AI branch, the workflow is:
  step1: create an AI branch and specify the source branch;
  step2: import (pick-to) source branch resources as needed;
  step3: edit the AI branch according to the user request;
  step4: after completion, ask the user to confirm whether to create a merge request or merge.
- When directly running merge or merge-request from CLI, require direct edit permission switches to be enabled for both the source branch and target branch. Otherwise, remind the user to enable the permission switches for the corresponding branch types, or manually trigger merge in the client.

## AI Branch

- AI branch is an isolated branch for AI/automation to modify project resources without directly polluting the source branch.
- An AI branch with no difference from its source branch within 24 hours will be auto-archived.
- AI branch starts empty and does not automatically clone source resources; before editing/deleting existing source resources, import them into the AI branch.
- Resources newly created in the AI branch do not need import first.
- AI branch changes are not written back automatically; after completion, ask the user to confirm whether to merge immediately or create a merge request.
- If the target main branch is protected, prefer `merge-request`; do not direct merge.

## Must Ask User

- Login token, local config writes, private deployment URL.
- Creating/switching AI branch, importing source resources into AI branch.
- Destructive operations such as delete, archive, overwrite import, batch update.
- Merging AI branch changes back through merge / merge-request.
- Whether to upgrade CLI.

## Recovery

| Issue | Action |
|-------|--------|
| Not logged in | `apidog login --with-token <TOKEN>` |
| Unknown project | `apidog project list` |
| Unknown parameters | `apidog <command> --help` |
| Parameter or schema error | Run `cli-schema get` / `cli-schema validate` first |
| AI write blocked | Explain AI branch / permission switch and let user choose |
| Private deployment | Add `--api-base-url https://your-server` |
