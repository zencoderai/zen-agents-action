# Zen Agents Action

> [GitHub Action](https://github.com/features/actions) for [Zencoder Agents](https://zencoder.ai/product/zen-agents-ci)

[![GitHub Release][release-img]][release]
[![License][license-img]][license]

## Authentication

To get Zencoder client id and secret go to the https://auth.zencoder.ai/profile and click on **Settings** -> **Personal Tokens**.

## Workflow examples

### Pull request reviews

```yaml
name: Run Zencoder Agent on PR review

on:
  pull_request:
    branches:
      - main
jobs:
  agent-run:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Running Zencoder Agent
        uses: zencoderai/zen-agents-action@main
        with:
          prompt: "Please review the pull request with number ${{ github.event.pull_request.number }} and add comment to it. Please address only the serious issues, not the minor ones. If you didn't find any serious issues, please write 'Looks good to me!'."
          zencoder_client_id: "${{ secrets.ZENCODER_CLIENT_ID }}"
          zencoder_client_secret: "${{ secrets.ZENCODER_CLIENT_SECRET }}"
          github_token: "${{ secrets.GITHUB_TOKEN }}"
```

### Workflow to be triggered from API call or manually

```yaml
name: Run Zencoder Agent

on:
  workflow_dispatch:
    inputs:
      prompt:
        type: string
        description: "The input prompt for the Agent"
        required: true
      agent:
        type: string
        description: "Alias of the agent to run (will run default agent if not provided)"
        required: false
        default: ""
jobs:
  agent-run:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Running Zencoder Agent
        uses: zencoderai/zen-agents-action@main
        with:
          prompt: "${{ inputs.prompt }}"
          agent: "${{ inputs.agent }}"
          zencoder_client_id: "${{ secrets.ZENCODER_CLIENT_ID }}"
          zencoder_client_secret: "${{ secrets.ZENCODER_CLIENT_SECRET }}"
          github_token: "${{ secrets.GITHUB_TOKEN }}"
```

## Inputs

| Name                     | Type   | Required | Default    | Description                                                        |
| ------------------------ | ------ | -------- | ---------- | ------------------------------------------------------------------ |
| `zencoder_client_id`     | String | true     |            | Zencoder client id for authentication                              |
| `zencoder_client_secret` | String | true     |            | Zencoder client secret for authentication                          |
| `github_token`           | String | true     |            | GitHub token used by the action to access GitHub APIs              |
| `prompt`                 | String | true     |            | The input prompt for the Agent                                     |
| `version`                | String | false    | `"latest"` | Version of the zencoder to use                                     |
| `agent`                  | String | false    | `""`       | Alias of the agent to run (will run default agent if not provided) |
| `base_path`              | String | false    | `"."`      | Base path to run agent from (should be repository root)            |
| `model`                  | String | false    | `"default"`| LLM model to use (e.g., `opus-4-5`, `sonnet-4`, `gemini-25-pro`). Use `default` for server default. |

[release]: https://github.com/zencoderai/zen-agents-action/releases/latest
[release-img]: https://img.shields.io/github/release/zencoderai/zen-agents-action.svg?logo=github
[license]: https://github.com/zencoderai/zen-agents-action/blob/main/LICENSE
[license-img]: https://img.shields.io/github/license/zencoderai/zen-agents-action
