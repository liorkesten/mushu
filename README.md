<p align="center">
  <img src="https://raw.githubusercontent.com/liorkesten/mushu/main/assets/mushu-logo.svg" alt="Mushu Logo" width="180"/>
</p>

<h1 align="center">🐉 Mushu</h1>

<p align="center">
  <strong>AI-powered code analyst that reads your tickets and explores your codebase</strong>
</p>

<p align="center">
  <a href="https://github.com/liorkesten/mushu/releases"><img src="https://img.shields.io/github/v/release/liorkesten/mushu?style=flat-square" alt="Release"></a>
  <a href="https://github.com/liorkesten/mushu/blob/main/LICENSE"><img src="https://img.shields.io/github/license/liorkesten/mushu?style=flat-square" alt="License"></a>
  <a href="https://github.com/liorkesten/mushu/stargazers"><img src="https://img.shields.io/github/stars/liorkesten/mushu?style=flat-square" alt="Stars"></a>
</p>

<p align="center">
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-architecture">Architecture</a> •
  <a href="#-analysis-modes">Modes</a> •
  <a href="#-integrations">Integrations</a> •
  <a href="#-examples">Examples</a>
</p>

---

## What is Mushu?

Mushu is a composable GitHub Action that uses Claude AI to analyze issue tickets, explore codebases, and create pull requests with fixes.

```
📋 Ticket → 🐉 Mushu (analyze) → 📝 Results → 🔌 Integrations (Jira/Slack/GitHub)
```

---

## 🚀 Quick Start

```yaml
- uses: liorkesten/mushu@v1
  with:
    ticket_key: "PROJ-123"
    ticket_summary: "Login button not working"
    analysis_mode: bug
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

That's it! Mushu will analyze your codebase and output the results.

---

## 🏗️ Architecture

Mushu follows a **composable architecture** - small, focused actions that work together:

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Workflow                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────────┐                                       │
│   │  liorkesten/    │──► analysis                           │
│   │  mushu@v1       │──► pr_url (fix mode)                  │
│   │  (Core)         │──► has_changes                        │
│   └────────┬────────┘                                       │
│            │                                                 │
│            ▼                                                 │
│   ┌────────┴────────┬──────────────┬──────────────┐        │
│   │                 │              │              │        │
│   ▼                 ▼              ▼              ▼        │
│ ┌─────────┐   ┌──────────┐   ┌─────────┐   ┌─────────┐   │
│ │  Jira   │   │  GitHub  │   │  Slack  │   │  Your   │   │
│ │ Action  │   │  Issues  │   │ Action  │   │ Custom  │   │
│ └─────────┘   └──────────┘   └─────────┘   └─────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Available Actions

| Action | Purpose |
|--------|---------|
| `liorkesten/mushu@v1` | Core analysis with Claude |
| `liorkesten/mushu/integrations/jira@v1` | Post results to Jira |
| `liorkesten/mushu/integrations/github-issues@v1` | Post results to GitHub Issues |
| `liorkesten/mushu/integrations/slack@v1` | Send Slack notifications |

---

## 📊 Analysis Modes

### 🐛 Bug Mode
Investigates bugs and suggests fixes.

```yaml
analysis_mode: bug
```

### 🔍 Explore Mode
Maps the codebase for new features.

```yaml
analysis_mode: explore
```

### 🔧 Fix Mode
Analyzes AND creates a PR with the fix.

```yaml
analysis_mode: fix
github_token: ${{ secrets.GITHUB_TOKEN }}  # Required for fix mode
```

---

## 🔌 Integrations

### Jira

```yaml
# Step 1: Analyze
- name: Analyze
  id: mushu
  uses: liorkesten/mushu@v1
  with:
    ticket_key: PROJ-123
    ticket_summary: "Bug title"
    analysis_mode: bug
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

# Step 2: Post to Jira
- name: Post to Jira
  uses: liorkesten/mushu/integrations/jira@v1
  with:
    jira_base_url: ${{ secrets.JIRA_BASE_URL }}
    jira_email: ${{ secrets.JIRA_EMAIL }}
    jira_api_token: ${{ secrets.JIRA_API_TOKEN }}
    ticket_key: PROJ-123
    analysis: ${{ steps.mushu.outputs.analysis }}
    analysis_mode: bug
```

### GitHub Issues

```yaml
- name: Post to Issue
  uses: liorkesten/mushu/integrations/github-issues@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    issue_number: ${{ github.event.issue.number }}
    analysis: ${{ steps.mushu.outputs.analysis }}
    analysis_mode: bug
    add_labels: mushu-analyzed
```

### Slack

```yaml
- name: Notify Slack
  uses: liorkesten/mushu/integrations/slack@v1
  with:
    slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    ticket_key: PROJ-123
    ticket_summary: "Bug title"
    analysis_mode: bug
    pr_url: ${{ steps.mushu.outputs.pr_url }}
```

---

## 📋 Examples

### Basic: Manual Trigger

```yaml
name: Mushu
on:
  workflow_dispatch:
    inputs:
      ticket_key:
        required: true
      ticket_summary:
        required: true
      analysis_mode:
        type: choice
        options: [bug, explore, fix]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: liorkesten/mushu@v1
        with:
          ticket_key: ${{ inputs.ticket_key }}
          ticket_summary: ${{ inputs.ticket_summary }}
          analysis_mode: ${{ inputs.analysis_mode }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Jira: Triggered by Webhook

```yaml
name: Mushu + Jira
on:
  repository_dispatch:
    types: [mushu-bug, mushu-explore, mushu-fix]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - id: mode
        run: |
          case "${{ github.event.action }}" in
            mushu-bug) echo "value=bug" >> $GITHUB_OUTPUT ;;
            mushu-explore) echo "value=explore" >> $GITHUB_OUTPUT ;;
            mushu-fix) echo "value=fix" >> $GITHUB_OUTPUT ;;
          esac

      - id: mushu
        uses: liorkesten/mushu@v1
        with:
          ticket_key: ${{ github.event.client_payload.key }}
          ticket_summary: ${{ github.event.client_payload.summary }}
          analysis_mode: ${{ steps.mode.outputs.value }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}

      - uses: liorkesten/mushu/integrations/jira@v1
        with:
          jira_base_url: ${{ secrets.JIRA_BASE_URL }}
          jira_email: ${{ secrets.JIRA_EMAIL }}
          jira_api_token: ${{ secrets.JIRA_API_TOKEN }}
          ticket_key: ${{ github.event.client_payload.key }}
          analysis: ${{ steps.mushu.outputs.analysis }}
          analysis_mode: ${{ steps.mode.outputs.value }}
          pr_url: ${{ steps.mushu.outputs.pr_url }}
          has_changes: ${{ steps.mushu.outputs.has_changes }}
```

### GitHub Issues: Auto-analyze Bugs

```yaml
name: Mushu + Issues
on:
  issues:
    types: [opened, labeled]

jobs:
  analyze:
    if: contains(github.event.issue.labels.*.name, 'bug')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - id: mushu
        uses: liorkesten/mushu@v1
        with:
          ticket_key: ${{ github.event.issue.number }}
          ticket_summary: ${{ github.event.issue.title }}
          ticket_description: ${{ github.event.issue.body }}
          analysis_mode: bug
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

      - uses: liorkesten/mushu/integrations/github-issues@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          issue_number: ${{ github.event.issue.number }}
          analysis: ${{ steps.mushu.outputs.analysis }}
          analysis_mode: bug
```

See [`examples/`](examples/) for more complete workflows.

---

## ⚙️ Configuration

### Core Action Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `ticket_key` | ✅ | - | Issue key |
| `ticket_summary` | ✅ | - | Issue title |
| `ticket_description` | ❌ | `""` | Issue description |
| `analysis_mode` | ✅ | `bug` | `bug`, `explore`, or `fix` |
| `anthropic_api_key` | ✅ | - | Claude API key |
| `github_token` | ❌ | - | Required for `fix` mode |
| `base_branch` | ❌ | `main` | Base branch for PRs |
| `custom_instructions` | ❌ | `""` | Extra prompt instructions |
| `timeout_minutes` | ❌ | `15` | Analysis timeout |
| `pr_draft` | ❌ | `true` | Create draft PRs |
| `pr_labels` | ❌ | `mushu,automated` | PR labels |

### Core Action Outputs

| Output | Description |
|--------|-------------|
| `analysis` | Claude's analysis text |
| `pr_url` | Created PR URL (fix mode) |
| `pr_number` | Created PR number (fix mode) |
| `has_changes` | Whether changes were made |
| `branch_name` | Created branch name |

---

## 📁 Repository Structure

```
mushu/
├── action.yml                      # Core action (~150 lines)
├── integrations/
│   ├── jira/action.yml            # Jira integration (~120 lines)
│   ├── github-issues/action.yml   # GitHub Issues (~100 lines)
│   └── slack/action.yml           # Slack notifications (~100 lines)
├── prompts/
│   ├── bug.md                      # Bug analysis prompt
│   ├── explore.md                  # Exploration prompt
│   └── fix.md                      # Fix implementation prompt
├── examples/
│   ├── basic-workflow.yml
│   ├── jira-integration.yml
│   ├── github-issues.yml
│   └── full-integration.yml
└── docs/
    ├── integrations.md
    └── customization.md
```

---

## 🔒 Security

- API keys are secrets, never logged
- `bug` and `explore` modes are **read-only**
- `fix` mode writes to a **new branch** only
- PRs are **drafts** by default

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📜 License

MIT License - see [LICENSE](LICENSE).

---

<p align="center">
  Made with 🐉 by <a href="https://zafran.io">Zafran</a>
</p>
