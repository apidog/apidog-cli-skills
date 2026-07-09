# Apidog CLI Installation Guide

This guide helps an AI Agent or user complete the first-time setup for Apidog CLI, including installation, login, and basic verification. After setup, the AI Agent can use Apidog CLI to manage project resources, run automated API tests, and query or edit API and testing data.

## Prerequisites

- Node.js and npm/npx are installed.
- You have an Apidog account.
- To operate a specific project, make sure the account has access to that project.

## Step 1: Install or Upgrade CLI

```bash
npm install -g apidog-cli@latest
```

Verify the installation:

```bash
apidog --version
```

## Step 2: Get an API Access Token

Get a token from the Apidog app or web console:

1. Click the user avatar.
2. Open Account Settings.
3. Go to API Access Token.
4. Create and copy a token.

Security requirement: do not print the token in logs, chat messages, or repository files.

## Step 3: Login to CLI

```bash
apidog login --with-token <TOKEN>
```

## Step 4: Record a Long-Term Working Project (Highly Recommended)

Get the project ID from: Project → Project Settings → Basic Settings → Project ID.

You can save the commonly used default `projectId` to `.apidog/settings.json`, so the AI Agent can read this file in later tasks:

```json
{
  "projectId": 123456
}
```

When the user request does not explicitly specify a project, prefer the `projectId` saved here.

Also make sure `.apidog/.gitignore` contains:

```gitignore
*.private.*
```

## Step 5: Verify Installation and Login

View available commands:

```bash
apidog --help
```

Check the current login identity:

```bash
apidog whoami
```

List projects:

```bash
apidog project list
```

If a project ID is ready, verify with a resource query command, for example:

```bash
apidog environment list --project <PROJECT_ID>
```

## Step 6: Install AI Agent Skill (!Required)

Install the Apidog AI Agent Skills so the AI Agent knows how to use Apidog CLI correctly:

```bash
npx -y skills add https://apidog.com
```

This command starts an interactive flow where you can choose the Skills, target AI Agent, and installation scope.

Or install from the GitHub repository:

```bash
npx -y skills add https://github.com/apidog/apidog-cli-skills
```

If the current environment cannot access the official installation source, manually install Apidog AI Agent Skills according to your AI Agent's rules. Use `WebFetch` to fetch https://apidog.com/.well-known/agent-skills/index.json, then download and install the required `SKILL.md` files from the `url` fields. Installing all 8 skills is recommended; at minimum, install `apidog-cli`.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `apidog: command not found` | Reinstall globally and make sure npm global bin is in PATH |
| Not logged in | Run `apidog login --with-token <TOKEN>` again |
| Project not found or permission denied | Check the project ID and make sure the token account has project access |
| Unsure command parameters | Run `apidog <command> --help` for the latest usage |
