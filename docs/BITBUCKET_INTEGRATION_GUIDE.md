# Bitbucket Integration Guide for Autonomous Agent

This guide provides step-by-step instructions for integrating the Autonomous Agent with Bitbucket repositories, enabling automated code reviews, pull request management, and AI-powered development workflows.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Bitbucket Access Token Setup](#bitbucket-access-token-setup)
3. [Bitbucket Pipelines Setup](#bitbucket-pipelines-setup)
4. [MCP Tools Configuration](#mcp-tools-configuration)
5. [Usage Examples](#usage-examples)
6. [Troubleshooting](#troubleshooting)

## Prerequisites

Before starting the integration, ensure you have:

- A Bitbucket account with repository access
- Zencoder client credentials (CLIENT_ID and CLIENT_SECRET)
- Repository with write permissions
- Basic understanding of environment variables and YAML configuration

## Bitbucket Access Token Setup

### Step 1: Generate Bitbucket Access Token

#### Workspace Access Token (Recommended)

For organizations managing multiple repositories:

1. Navigate to your Bitbucket workspace
2. Go to **Workspace settings** → **Access tokens**
3. Click **Create access token**
4. Name your token (e.g., "Zencoder Agent")
5. Select the following permissions:
   - `pipeline:write` - To trigger pipelines
   - `pullrequest:write` - To read, create and comment on pull requests
   - `repository:write` - To access repository content
6. Click **Create** and save the token securely

#### Repository Access Token (Alternative)

For single repository setup:

1. Go to your repository in Bitbucket
2. Navigate to **Repository settings** → **Access tokens**
3. Follow the same permission selection as above

## Bitbucket Pipelines Setup

### Step 1: Add Pipeline Configuration

1. Copy the pipeline configuration to your repository:

```bash
cp docs/bitbucket-pipelines.yml ./bitbucket-pipelines.yml
```

2. Commit and push the file:

```bash
git add bitbucket-pipelines.yml
git commit -m "Add Bitbucket Pipelines configuration for Zencoder"
git push origin main
```

### Step 2: Configure Pipeline Variables

#### Workspace-Level Configuration (Recommended)

For organizations managing multiple repositories:

1. Navigate to **Workspace settings** → **Pipelines** → **Workspace variables**
2. Add the following variables:
   
   | Variable Name | Value | Secured |
   |--------------|-------|---------|
   | `ZENCODER_CLIENT_ID` | Your client ID | No |
   | `ZENCODER_CLIENT_SECRET` | Your client secret | Yes |
   | `ZENCODER_BITBUCKET_TOKEN` | Your Bitbucket token | Yes |

#### Repository-Level Configuration (Alternative)

For single repository setup:

1. Go to **Repository settings** → **Pipelines** → **Repository variables**
2. Add the same variables as above

### Step 3: Enable Pipelines

1. Go to **Repository settings** → **Pipelines** → **Settings**
2. Toggle **Enable Pipelines** to ON
3. Verify you have sufficient build minutes in your plan

### Step 4: Test Pipeline Configuration

1. **Test Manual Pipeline:**
   - Go to **Pipelines** → **Run pipeline**
   - Select branch: `main` (or your default branch)
   - Pipeline: **Custom: zencoder-agent**
   - Variables:
     - `prompt`: "Hello, please introduce yourself"
     - `agent`: (leave empty for default)
   - Click **Run**

2. **Test PR Review Pipeline:**
   - Create a test pull request
   - The pipeline should automatically trigger
   - Check for agent comments on the PR

## MCP Tools Configuration

The agent automatically detects Bitbucket repositories and enables appropriate tools when `BITBUCKET_TOKEN` is configured.

### Available Bitbucket Tools

1. **bitbucket_create_pull_request**
   - Creates new pull requests
   - Supports draft/WIP PRs
   - Configurable reviewers

2. **bitbucket_create_review**
   - Adds review comments to PRs
   - Supports line-specific comments
   - Includes file and line context

3. **bitbucket_view_pull_request**
   - Views PR details and diff
   - Reads existing comments
   - Shows file changes with line numbers

### Tool Detection Logic

The agent automatically:
1. Detects repository type from Git remote URL
2. Enables Bitbucket tools when:
   - `BITBUCKET_TOKEN` is set
   - Repository URL matches Bitbucket patterns
3. Falls back gracefully when tools aren't available

## Usage Examples

### Example 1: Automatic PR Reviews

Once configured, every pull request will trigger an automatic review:

```yaml
# The pipeline automatically runs with this prompt:
prompt: "Please review the pull request with number $BITBUCKET_PR_ID 
         and add comment to it. Please address only the serious issues, 
         not the minor ones."
```

### Example 2: Creating a Pull Request

When running through Bitbucket Pipelines, the agent can create PRs using the available tools.

### Example 3: Custom Pipeline Execution

Trigger via Bitbucket UI:
1. Go to **Pipelines** → **Run pipeline**
2. Select **Custom: zencoder-agent**
3. Enter your custom prompt:
   ```
   Review the codebase for security vulnerabilities and create 
   issues for any findings
   ```

### Example 4: Reviewing Specific PR

Trigger a custom pipeline with a prompt like:
```
View pull request #123 and summarize the changes and review comments
```

## Troubleshooting

### Common Issues and Solutions

1. **"Bitbucket tools not available" error**
   - Verify `BITBUCKET_TOKEN` is set correctly
   - Check token has required permissions
   - Ensure you're in a Bitbucket repository

2. **Pipeline fails with authentication error**
   - Verify pipeline variables are set correctly
   - Check token hasn't expired
   - Ensure variables aren't overridden at different levels

3. **Agent skips PR review**
   - This is intentional for bot-created PRs
   - Check commit author isn't "bitbucket-pipelines"
   - Verify PR was created by a human user

4. **Tools not detecting Bitbucket repository**
   - Check Git remote URL: `git remote -v`
   - Ensure URL matches Bitbucket patterns:
     - `https://bitbucket.org/owner/repo`
     - `git@bitbucket.org:owner/repo.git`

### Debugging Steps

1. **Check Pipeline Variables:**
   - Go to **Pipelines** → **View logs**
   - Verify variables are being loaded correctly

2. **Test Token Permissions:**
   ```bash
   curl -H "Authorization: Bearer $BITBUCKET_TOKEN" \
        https://api.bitbucket.org/2.0/user
   ```

3. **Check Pipeline Logs:**
   - Go to **Pipelines** in Bitbucket
   - Click on failed pipeline
   - Review step logs for errors

### Getting Help

If you encounter issues not covered here:

1. Check the [Bitbucket API documentation](https://developer.atlassian.com/cloud/bitbucket/rest/)
2. Review agent logs for detailed error messages
3. Contact Zencoder support with:
   - Error messages
   - Pipeline logs
   - Configuration details (without sensitive tokens)

## Security Best Practices

1. **Token Security:**
   - Never commit tokens to version control
   - Use secured variables in Bitbucket
   - Rotate tokens regularly
   - Use minimal required permissions

2. **Pipeline Security:**
   - Review pipeline configuration before committing
   - Limit who can modify pipeline files
   - Monitor pipeline usage and costs

3. **Code Review Settings:**
   - Configure agent to focus on serious issues
   - Set up branch protection rules
   - Require human approval for merges

## Next Steps

After completing the integration:

1. Customize the PR review prompt for your team's needs
2. Create custom agents for specific workflows
3. Integrate with your existing CI/CD pipeline
4. Set up monitoring and alerts for pipeline failures

For advanced configurations and custom workflows, refer to the main documentation or contact support.