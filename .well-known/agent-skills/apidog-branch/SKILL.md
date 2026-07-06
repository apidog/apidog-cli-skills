---
name: apidog-branch
description: "Apidog branch collaboration: regular branches, sprint branches, AI branches, pick-to, merge, merge-request, protected branches, AI write permissions, and branch-scoped resource changes. Use when modifying resources on branches, creating AI branches, or merging changes."
metadata:
  requires:
    bins: ["apidog"]
  cliHelp: "apidog branch --help; apidog merge-request --help"
---

# Branch Collaboration And AI Writes

> Prerequisite: read `../apidog-cli/SKILL.md` first. If the stable entry skill conflicts with this domain skill, use the current CLI help and this skill as the source of truth.

Always use current CLI help for exact arguments. This skill adds the branch collaboration rules most likely to be misused: branch context, AI branches, pick-to, write permissions, protected branches, and merge confirmation. After writing on a branch, agents must read back and validate with the same `--branch`; do not judge resource existence from the default branch view.

## When To Use

- The user specifies `--branch` or asks to operate on a branch.
- Create, view, update, archive, or delete branches.
- Create an AI branch and modify resources from a source branch.
- Pick source branch resources into an AI branch.
- Merge branches or create/approve merge requests.
- Encounter AI write permission restrictions.

## Command Entry Points

- Branch management: `apidog branch --help`
- Merge requests: `apidog merge-request --help`
- Resource commands usually support `--branch <branchName>`. Keep the same branch context for query, create, update, and test runs.

## Decide The Change Strategy First

Before writing to a branch, clarify which strategy the user wants:

- Directly edit the target branch: suitable when the user explicitly allows direct changes and the target branch is not protected or permission-restricted.
- Edit through an AI branch: suitable for AI-generated API, test case, test scenario, or configuration changes; also suitable when the target branch is protected, external AI direct write is disabled, or the user wants isolated validation before merge.

Before starting writes, confirm:

- Project ID.
- Source branch and target branch.
- Resource scope to modify.
- Whether creating a merge request or merging later is allowed.

## AI Branch Flow

AI branches isolate AI changes and avoid polluting source branches.

```text
Confirm source branch and target change
  -> create AI branch
  -> pick-to existing resources
  -> modify on AI branch
  -> validate with get/run/report
  -> preview merge-request
  -> after user confirmation, create merge-request or merge
```

Standard steps:

1. When creating an AI branch, specify its source with `--from <sourceBranchName>`.
2. Use a clear name, such as `ai/YYYYMMDD-from-source-feature`, for example `ai/20260312-from-main-userRegister`.
3. Before modifying an existing source branch resource, import it into the AI branch with `branch pick-to`; new resources do not need pick-to.
4. Always pass `--branch <aiBranchName>` when modifying, querying, or validating on the AI branch.
5. Validate by resource type, such as `get`, `run`, local reports, or cloud report checks.
6. Before merging, run `merge-request preview` and let the user confirm the impact.
7. After confirmation, create a merge request or merge; for protected main branches, prefer merge requests.

AI branch notes:

- AI branches start empty and do not automatically copy all resources from the source branch.
- `branch get` may return `type: SPRINT` with `isAiBranch: true`; this does not mean operations should use `--type sprint`. AI branch operations should still follow current help and use `--type ai` where required.

## Branch Parameter Rules

- Prefer branch names: `--branch <branchName>`.
- Numeric branchId may be compatible, but branch names are clearer in logs and diagnosis.
- Within one task, query, create, update, and run tests with the same branch context to avoid looking for resources from main/default by mistake.

## Pick-To Rules

- Must pass `--type ai`.
- Only supports importing from main or regular sprint branches into an AI branch.
- Cannot import into main.
- Cannot use an AI branch as the source branch.
- Use `--include-endpoint-cases` to import endpoint cases together with endpoints.

## Write Permission Rules

- Main, sprint, and regular branches may require enabling external AI edit permission.
- Typical errors include `Automation caller branch required`.
- Creating an AI branch may succeed, but writing content into the AI branch may still fail if AI editing of AI branches is disabled.
- When writes are restricted, do not switch strategy automatically; ask the user whether to enable direct edit permission or use an AI branch.
- If the target main branch is protected, prefer `merge-request preview/create`; do not directly run `branch merge`.

## Rules That Must Not Be Violated

1. Do not create an AI branch without user confirmation.
2. Do not modify existing source branch resources in an AI branch without first using pick-to.
3. Do not omit `--branch <branchName>` in branch-scoped tasks.
4. Do not merge, create merge requests, or approve merge requests without user confirmation.
5. Do not delete or archive branches unless explicitly requested.
6. When seeing `Automation caller branch required`, stop and ask the user to choose a permission strategy; do not automatically create or switch to an AI branch.

## Common Recovery

| Symptom | Action |
|---------|--------|
| `get` without branch cannot find resource | Retry with `--branch <branchName>` |
| Source resource missing in AI branch | Use `branch pick-to` first |
| Branch state has counts but AI branch page is empty | Do not assume resources were imported; AI branches still require pick-to |
| Run config returns 404 on a branch | Confirm case/endpoint/environment/branch all exist, then switch to `apidog-cli-checkup` |
| Merge is blocked by protected branch | Use `merge-request` instead |
