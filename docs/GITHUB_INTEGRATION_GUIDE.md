# GitHub Integration Guide for Autonomous Agent

This guide provides step-by-step instructions for integrating the Autonomous Agent with GitHub repositories, enabling automated code reviews, pull request management, and AI-powered development workflows.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [GitHub Access Token Setup](#github-access-token-setup)
3. [GitHub Actions Setup](#github-actions-setup)
4. [MCP Tools Configuration](#mcp-tools-configuration)
5. [Usage Examples](#usage-examples)
6. [Troubleshooting](#troubleshooting)

## Prerequisites

Before starting the integration, ensure you have:

- A GitHub account with repository access
- Zencoder client credentials (CLIENT_ID and CLIENT_SECRET)
- Repository with write permissions
- Basic understanding of GitHub Actions and YAML configuration

## GitHub Access Token Setup

### Step 1: Generate GitHub Personal Access Token

#### Classic Personal Access Token (Recommended)

For broad repository access:

1. Navigate to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click **Generate new token** → **Generate new token (classic)**
3. Name your token (e.g., "Zencoder Agent")
4. Set expiration (90 days recommended, or custom)
5. Select the following scopes:
   - `repo` - Full control of private repositories
   - `workflow` - Update GitHub Action workflows
6. Click **Generate token** and save it securely

## GitHub Actions Setup

### Step 1: Add Workflow Files

1. Create the workflows directory in your repository:

```bash
mkdir -p .github/workflows
```

2. Commit the directory structure:

```bash
git add .github/workflows
git commit -m "Add GitHub Actions workflows directory"
git push origin main
```

### Step 2: Configure Action Secrets

#### Organization-Level Configuration (Recommended)

For managing multiple repositories:

1. Navigate to **Organization Settings** → **Secrets and variables** → **Actions**
2. Click **New organization secret**
3. Add the following secrets:
   
   | Secret Name              | Value              | Description                     |
   |--------------------------|--------------------|---------------------------------|
   | `ZENCODER_CLIENT_ID`     | Your client ID     | Zencoder API client ID          |
   | `ZENCODER_CLIENT_SECRET` | Your client secret | Zencoder API client secret      |
   | `CICD_TOKEN`             | Your GitHub PAT    | Token with elevated permissions |

4. Select which repositories can access these secrets

#### Repository-Level Configuration (Alternative)

For single repository setup:

1. Go to **Repository Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add the same secrets as above

### Step 3: Create Workflow Files

#### Manual Agent Execution Workflow

1. Copy the manual agent workflow to your repository:

```bash
cp docs/zencoder-agent.yaml .github/workflows/zencoder-agent.yaml
```

#### Automated PR Review Workflow

2. Copy the PR review workflow to your repository:

```bash
cp docs/pr-review.yaml .github/workflows/pr-review.yaml
```

3. Commit and push the workflow files:

```bash
git add .github/workflows/
git commit -m "Add Zencoder GitHub Actions workflows"
git push origin main
```

### Step 4: Configure Workflow Permissions

1. Go to **Repository Settings** → **Actions** → **General**
2. Under "Workflow permissions":
   - Select **Read and write permissions**
   - Check **Allow GitHub Actions to create and approve pull requests**
3. Click **Save**

### Step 5: Test Workflow Configuration

1. **Test Manual Workflow:**
   - Go to **Actions** tab in your repository
   - Select **Run Zencoder Agent** workflow
   - Click **Run workflow**
   - Enter a test prompt: "Hello, please introduce yourself"
   - Leave agent empty for default
   - Click **Run workflow**

2. **Test PR Review Workflow:**
   - Create a test pull request
   - The workflow should automatically trigger
   - Check for agent comments on the PR

## MCP Tools Configuration

The agent automatically detects GitHub repositories and enables appropriate tools when `GITHUB_TOKEN` is configured.

### Available GitHub Tools

1. **github_create_pull_request**
   - Creates new pull requests
   - Sets title, body, and base/head branches
   - Assigns reviewers

2. **github_create_review**
   - Adds review comments to PRs
   - Supports line-specific comments
   - Includes file context and suggestions

3. **github_view_pull_request**
   - Views PR details and diff
   - Reads existing comments and reviews
   - Shows file changes with line numbers

### Tool Detection Logic

The agent automatically:
1. Detects repository type from Git remote URL
2. Enables GitHub tools when:
   - `GITHUB_TOKEN` is set (via action or environment)
   - Repository URL matches GitHub patterns
3. Falls back gracefully when tools aren't available

## Usage Examples

### Example 1: Automatic PR Reviews

Once configured, pull requests will trigger automatic reviews with prompts like:

```
Review pull request #123 and add comments. Focus on serious issues only.
```

The agent will:
- Analyze code changes
- Add inline comments for issues
- Post a summary review

### Example 2: Custom Agent Execution

Trigger via GitHub UI:
1. Go to **Actions** → **Run Zencoder Agent**
2. Enter your custom prompt:
   ```
   Analyze the codebase for security vulnerabilities and create issues for findings
   ```

### Example 3: Re-triggering PR Review

To re-run a review on an existing PR:
1. Go to the pull request
2. Request a review from `@zencoderai`
3. The workflow will trigger automatically

### Example 4: Batch Operations

Run agent with complex prompts:
```
Review all open pull requests labeled 'needs-review' and provide feedback
```

## Troubleshooting

### Common Issues and Solutions

1. **"GitHub tools not available" error**
   - Verify `GITHUB_TOKEN` or `CICD_TOKEN` is set correctly
   - Check token has required permissions
   - Ensure you're in a GitHub repository

2. **Workflow fails with authentication error**
   - Verify secrets are properly configured
   - Check token hasn't expired
   - Ensure token has necessary scopes

3. **Agent can't comment on PRs**
   - Verify workflow permissions are set to read/write
   - Check CICD_TOKEN has `pull_request:write` scope
   - Ensure PR is not from a fork (limited permissions)

4. **Workflow doesn't trigger**
   - Check workflow file syntax with GitHub's validator
   - Verify branch protection rules aren't blocking
   - Ensure workflow file is on default branch

### Debugging Steps

1. **Check Workflow Logs:**
   - Go to **Actions** tab
   - Click on the failed workflow run
   - Review step logs for detailed errors

2. **Verify Secrets:**
   - Go to **Settings** → **Secrets and variables**
   - Ensure all required secrets are present
   - Check secret names match workflow references

3. **Test Token Permissions:**
   ```bash
   curl -H "Authorization: token YOUR_GITHUB_TOKEN" \
        https://api.github.com/user
   ```

4. **Enable Debug Logging:**
   Add to workflow:
   ```yaml
   env:
     ACTIONS_STEP_DEBUG: true
     ACTIONS_RUNNER_DEBUG: true
   ```

### Getting Help

If you encounter issues not covered here:

1. Check the [GitHub Actions documentation](https://docs.github.com/en/actions)
2. Review the [GitHub API documentation](https://docs.github.com/en/rest)
3. Check workflow run logs for detailed error messages
4. Contact Zencoder support with:
   - Error messages
   - Workflow logs
   - Configuration details (without tokens)

## Security Best Practices

1. **Token Security:**
   - Never commit tokens to version control
   - Use GitHub Secrets for all credentials
   - Rotate tokens regularly (every 90 days)
   - Use minimum required permissions

2. **Workflow Security:**
   - Pin action versions (`@v1.2.3` not `@main`)
   - Review third-party actions before use
   - Limit workflow triggers to necessary events
   - Use environment protection rules for sensitive operations

3. **Repository Settings:**
   - Enable branch protection rules
   - Require PR reviews before merge
   - Limit who can approve workflow runs
   - Monitor Actions usage and billing

## Next Steps

After completing the integration:

1. Customize PR review prompts for your team's standards
2. Create specialized agents for different workflows
3. Integrate with existing CI/CD pipelines
4. Set up status checks and branch protection
5. Configure notifications for workflow results

For advanced configurations and custom workflows, refer to the main documentation or contact support.