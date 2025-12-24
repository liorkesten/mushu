<p align="center">
  <img src="https://raw.githubusercontent.com/zafran-io/mushu/main/assets/mushu-logo.svg" alt="Mushu Logo" width="200"/>
</p>

<h1 align="center">🐉 Mushu</h1>

<p align="center">
  <strong>AI-powered code analyst that reads your tickets and explores your codebase</strong>
</p>

<p align="center">
  <a href="https://github.com/zafran-io/mushu/actions"><img src="https://img.shields.io/github/actions/workflow/status/zafran-io/mushu/test.yml?branch=main&style=flat-square&logo=github" alt="Build Status"></a>
  <a href="https://github.com/zafran-io/mushu/releases"><img src="https://img.shields.io/github/v/release/zafran-io/mushu?style=flat-square&logo=github" alt="Release"></a>
  <a href="https://github.com/zafran-io/mushu/blob/main/LICENSE"><img src="https://img.shields.io/github/license/zafran-io/mushu?style=flat-square" alt="License"></a>
  <a href="https://github.com/zafran-io/mushu/stargazers"><img src="https://img.shields.io/github/stars/zafran-io/mushu?style=flat-square&logo=github" alt="Stars"></a>
</p>

<p align="center">
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-features">Features</a> •
  <a href="#-analysis-modes">Modes</a> •
  <a href="#-integrations">Integrations</a> •
  <a href="#-configuration">Configuration</a>
</p>

---

## What is Mushu?

Mushu is a GitHub Action that uses Claude AI to analyze your issue tickets, explore your codebase, and even create pull requests with fixes. It bridges the gap between your issue tracker and your code.

```
📋 Jira/GitHub Issue → 🐉 Mushu → 🔍 Analysis + 🔧 Optional PR
```

### Why Mushu?

- **Save time** on initial bug investigation
- **Get context** before starting work on new features  
- **Automate simple fixes** with AI-generated PRs
- **Document findings** back to your issue tracker

---

## 🚀 Quick Start

### 1. Add the workflow

Create `.github/workflows/mushu.yml`:

```yaml
name: Mushu
on:
  workflow_dispatch:
    inputs:
      ticket_key:
        description: "Ticket key"
        required: true
      ticket_summary:
        description: "Summary"
        required: true
      analysis_mode:
        description: "Mode"
        type: choice
        options: [bug, explore, fix]

permissions:
  contents: write
  pull-requests: write

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: zafran-io/mushu@v1
        with:
          ticket_key: ${{ inputs.ticket_key }}
          ticket_summary: ${{ inputs.ticket_summary }}
          analysis_mode: ${{ inputs.analysis_mode }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Add your Anthropic API key

Go to **Settings → Secrets → Actions** and add:
- `ANTHROPIC_API_KEY`: Your Claude API key from [Anthropic Console](https://console.anthropic.com/)

### 3. Run it!

Go to **Actions → Mushu → Run workflow** and enter a ticket to analyze.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔍 **Smart Analysis** | Uses Claude to understand bugs and explore code |
| 🐛 **Bug Investigation** | Finds root causes with file:line references |
| 🗺️ **Code Exploration** | Maps relevant code for new features |
| 🔧 **Auto-Fix** | Creates PRs with minimal, focused changes |
| 📝 **Issue Tracking** | Posts findings back to Jira/GitHub |
| ⚡ **Fast** | Parallel tool calls for quick analysis |

---

## 📊 Analysis Modes

Mushu has three modes, each designed for different use cases:

### 🐛 Bug Mode

Investigates bug reports and suggests fixes.

```yaml
analysis_mode: bug
```

**Output includes:**
- Relevant files with line numbers
- Root cause analysis
- Suggested fix with code snippets
- Impact assessment
- Clarifying questions (if needed)

### 🔍 Explore Mode

Maps the codebase for new features or investigations.

```yaml
analysis_mode: explore
```

**Output includes:**
- Understanding of the request
- Relevant code map
- Existing patterns to follow
- Implementation suggestions
- Related features for reference

### 🔧 Fix Mode

Analyzes AND implements the fix, creating a PR.

```yaml
analysis_mode: fix
```

**Creates:**
- Branch: `mushu/{ticket-key}`
- Minimal, focused code changes
- Draft PR with analysis summary
- Testing notes and risk assessment

---

## 🔌 Integrations

### Jira

Auto-analyze bugs and post findings back to Jira:

```yaml
- uses: zafran-io/mushu@v1
  with:
    issue_tracker: jira
    issue_tracker_base_url: ${{ secrets.JIRA_BASE_URL }}
    issue_tracker_email: ${{ secrets.JIRA_EMAIL }}
    issue_tracker_api_token: ${{ secrets.JIRA_API_TOKEN }}
```

**Label flow:**
| Action | Label Added |
|--------|-------------|
| Bug analyzed | `mushu-analyzed` |
| Explore complete | `mushu-done` |
| PR created | `mushu-pr-created` |

See [Jira Integration Guide](docs/integrations.md#jira-integration) for automation setup.

### GitHub Issues

Analyze GitHub Issues directly:

```yaml
on:
  issues:
    types: [opened, labeled]

jobs:
  analyze:
    if: contains(github.event.issue.labels.*.name, 'bug')
    steps:
      - uses: zafran-io/mushu@v1
        with:
          ticket_key: ${{ github.event.issue.number }}
          ticket_summary: ${{ github.event.issue.title }}
          issue_tracker: github
```

### Linear, Notion, etc.

Trigger via `repository_dispatch` from any webhook-capable system:

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{"event_type":"mushu-bug","client_payload":{"key":"LIN-123","summary":"Bug title"}}' \
  "https://api.github.com/repos/OWNER/REPO/dispatches"
```

---

## ⚙️ Configuration

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `ticket_key` | ✅ | - | Issue key (e.g., PROJ-123) |
| `ticket_summary` | ✅ | - | Issue title/summary |
| `ticket_description` | ❌ | `""` | Issue description |
| `analysis_mode` | ✅ | `bug` | `bug`, `explore`, or `fix` |
| `anthropic_api_key` | ✅ | - | Claude API key |
| `github_token` | ❌* | - | Required for `fix` mode |
| `base_branch` | ❌ | `main` | Base branch for PRs |
| `issue_tracker` | ❌ | `none` | `jira`, `github`, `linear`, or `none` |
| `custom_prompt` | ❌ | `""` | Additional instructions |
| `timeout_minutes` | ❌ | `15` | Analysis timeout |
| `pr_draft` | ❌ | `true` | Create PR as draft |
| `pr_labels` | ❌ | `mushu,automated` | PR labels (comma-separated) |

### Outputs

| Output | Description |
|--------|-------------|
| `analysis` | Full Claude analysis text |
| `pr_url` | URL of created PR (fix mode) |
| `pr_number` | Number of created PR (fix mode) |
| `has_changes` | Whether code changes were made |
| `branch_name` | Created branch name (fix mode) |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `ANTHROPIC_API_KEY` | ✅ | Claude API key |
| `JIRA_BASE_URL` | For Jira | e.g., `https://company.atlassian.net` |
| `JIRA_EMAIL` | For Jira | Jira account email |
| `JIRA_API_TOKEN` | For Jira | Jira API token |

---

## 📁 Repository Structure

```
mushu/
├── action.yml              # Main action definition
├── prompts/
│   ├── bug.md              # Bug analysis prompt
│   ├── explore.md          # Exploration prompt
│   └── fix.md              # Fix implementation prompt
├── examples/
│   ├── basic-workflow.yml  # Simple setup
│   ├── jira-integration.yml
│   └── github-issues.yml
└── docs/
    ├── integrations.md     # Integration guides
    └── customization.md    # Customization options
```

---

## 🎨 Customization

### Custom Prompts

Add project-specific context:

```yaml
- uses: zafran-io/mushu@v1
  with:
    custom_prompt: |
      Codebase context:
      - Go 1.21 with generics
      - gRPC + Protocol Buffers
      - PostgreSQL with sqlc
```

### Forking for Full Control

1. Fork this repository
2. Modify prompts in `prompts/`
3. Use your fork: `uses: YOUR_ORG/mushu@main`

See [Customization Guide](docs/customization.md) for more options.

---

## 🔒 Security

- Mushu only has access to code in the checked-out repository
- In `bug` and `explore` modes, Mushu has **read-only** access
- In `fix` mode, changes are committed to a **new branch**
- All PRs are created as **drafts** by default
- API keys are passed as secrets, never logged

### Best Practices

1. Always review AI-generated PRs before merging
2. Use draft PRs (`pr_draft: true`)
3. Add required reviewers to AI-generated PRs
4. Limit `fix` mode to trusted ticket sources

---

## 🤝 Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

```bash
# Clone the repo
git clone https://github.com/zafran-io/mushu.git

# Create a branch
git checkout -b feature/amazing-feature

# Make your changes and submit a PR
```

---

## 📜 License

MIT License - see [LICENSE](LICENSE) for details.

---

## 💬 Support

- 📖 [Documentation](docs/)
- 🐛 [Report a Bug](https://github.com/zafran-io/mushu/issues/new?template=bug_report.md)
- 💡 [Request a Feature](https://github.com/zafran-io/mushu/issues/new?template=feature_request.md)
- 💬 [Discussions](https://github.com/zafran-io/mushu/discussions)

---

<p align="center">
  Made with 🐉 by <a href="https://zafran.io">Zafran</a>
</p>

