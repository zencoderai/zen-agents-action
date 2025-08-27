# GitLab CI/CD Integration

This document describes how to integrate Zencoder Autonomous Agent into your GitLab CI/CD pipelines.

## Overview

The GitLab CI/CD integration allows you to run AI-powered agents on your repositories using GitLab's CI/CD system. The agent can automatically review merge requests, perform code analysis, and execute custom tasks based on natural language prompts.

## Setup

### 1. Configure GitLab CI/CD Variables

In your GitLab project, navigate to **Settings > CI/CD > Variables** and add the following variables:

- `ZENCODER_CLIENT_ID`: Your Zencoder API client ID
- `ZENCODER_CLIENT_SECRET`: Your Zencoder API client secret (mark as protected and masked)
- `ZENCODER_GITLAB_TOKEN`: GitLab access token with API permissions (mark as protected and masked)

Make sure to set the **Minimum role to use pipeline variables** setting to the role of the person configuring the pipeline.

### 2. Add GitLab CI/CD Configuration

1. Copy the pipeline configuration to your repository:

```bash
cp docs/gitlab-ci.yml ./.gitlab-ci.yml
```

2. Make the changes to `.gitlab-ci.yml` according to your requirements.

3. Commit and push the file:

```bash
git add .gitlab-ci.yml
git commit -m "Add GitLab CI configuration for Zencoder"
git push origin main
```

## Usage Modes

### 1. Automatic Merge Request Review

The `zencoder-mr-review` job automatically runs when a merge request is created or updated. It will:

- Review the changes in the merge request
- Add constructive comments to the MR
- Focus on serious issues while ignoring minor style issues

### 2. Manual Agent Execution

The `zencoder-agent` job can be triggered manually from the GitLab CI/CD pipeline interface. To use it:

1. Go to your project's **CI/CD > Pipelines**
2. Click **Run pipeline**
3. Add the required variables:
   - `prompt`: The task you want the agent to perform
   - `agent`: (Optional) Specific agent to use
4. Run the pipeline

## Environment Variables

The following environment variables can be configured:

| Variable                 | Description | Required |
|--------------------------|-------------|----------|
| `ZENCODER_CLIENT_ID`     | Your Zencoder API client ID | Yes |
| `ZENCODER_CLIENT_SECRET` | Your Zencoder API client secret | Yes |
| `ZENCODER_GITLAB_TOKEN`  | GitLab access token with API permissions | Yes |
| `agent`                  | Specific agent to use (optional) | No |
| `VERSION`                | Zencoder binary version to download | No (defaults to "latest") |

## GitLab Token Permissions

The GitLab access token needs the following scopes:
- `api`: Full API access
- `read_api`: Read API access
- `read_repository`: Read repository content
- `write_repository`: Write repository content (for creating MRs if needed)

## Security Considerations

1. **Token Security**: Always mark sensitive variables (CLIENT_SECRET, GITLAB_TOKEN) as protected and masked
2. **Branch Protection**: Consider running agent jobs only on protected branches
3. **Bot Loop Prevention**: The configuration includes logic to prevent infinite loops when bots create MRs

## Customization

### Custom Prompts

You can customize the agent behavior by modifying the `prompt` variable in the job configuration. Examples:

```yaml
# Code quality review
prompt: "Review this merge request for code quality issues, performance problems, and security vulnerabilities."

# Documentation check
prompt: "Check if this merge request properly updates documentation for any API changes."

# Test coverage analysis
prompt: "Analyze the test coverage for changes in this merge request and suggest additional tests if needed."
```

### Conditional Execution

You can add conditions to control when jobs run:

```yaml
zencoder-mr-review:
  # ... other configuration
  rules:
    - if: '$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME =~ /^feature\/.*/'
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
```

## Troubleshooting

### Common Issues

1. **Token Issues**: Verify that all tokens are correctly set and have proper permissions
2. **Binary Download**: Check network connectivity and firewall settings
3. **Repository Access**: Ensure the GitLab token has access to the repository

### Debug Mode

To enable debug logging, add the `--debug` flag to the zencoder command:

```yaml
script:
  - ./zencoder --base-path "${CI_PROJECT_DIR}" --agent "${agent}" --prompt "${prompt}" --debug
```

## Example Integration

The configuration examples provided in this document demonstrate both automatic MR review and manual agent execution that you can implement in your own GitLab repositories.
