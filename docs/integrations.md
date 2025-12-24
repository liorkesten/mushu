# Integrations Guide

Mushu uses a composable architecture. The core action analyzes your code, and integration actions post results to your tools.

## Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    Trigger   │────►│   Analyze    │────►│  Integrate   │
│  (webhook)   │     │   (mushu)    │     │  (jira/etc)  │
└──────────────┘     └──────────────┘     └──────────────┘
```

---

## Jira Integration

### Setup

1. **Create Jira API Token**
   - Go to https://id.atlassian.com/manage-profile/security/api-tokens
   - Create token for Mushu

2. **Add GitHub Secrets**
   ```
   JIRA_BASE_URL: https://your-company.atlassian.net
   JIRA_EMAIL: your-email@company.com
   JIRA_API_TOKEN: your-token
   ```

### Action Reference

```yaml
- uses: liorkesten/mushu/integrations/jira@v1
  with:
    # Required
    jira_base_url: ${{ secrets.JIRA_BASE_URL }}
    jira_email: ${{ secrets.JIRA_EMAIL }}
    jira_api_token: ${{ secrets.JIRA_API_TOKEN }}
    ticket_key: PROJ-123
    analysis: ${{ steps.mushu.outputs.analysis }}
    analysis_mode: bug
    
    # Optional (for fix mode)
    pr_url: ${{ steps.mushu.outputs.pr_url }}
    has_changes: ${{ steps.mushu.outputs.has_changes }}
```

### Label Updates

The Jira action automatically updates labels:

| Mode | Label Added | Label Removed |
|------|-------------|---------------|
| bug | `mushu-analyzed` | - |
| explore | `mushu-done` | `mushu` |
| fix (with changes) | `mushu-pr-created` | `mushu-fix` |
| fix (no changes) | `mushu-no-changes` | `mushu-fix` |

### Triggering from Jira

Use Jira Automation or Workato/Zapier:

```javascript
// Jira Automation Rule
Trigger: Issue created
Condition: Issue type = Bug

Action: Send web request
  URL: https://api.github.com/repos/OWNER/REPO/dispatches
  Method: POST
  Headers:
    Authorization: token ${GITHUB_PAT}
    Accept: application/vnd.github.v3+json
  Body: {
    "event_type": "mushu-bug",
    "client_payload": {
      "key": "{{issue.key}}",
      "summary": "{{issue.summary}}",
      "description": "{{issue.description}}"
    }
  }
```

---

## GitHub Issues Integration

### Action Reference

```yaml
- uses: liorkesten/mushu/integrations/github-issues@v1
  with:
    # Required
    github_token: ${{ secrets.GITHUB_TOKEN }}
    issue_number: ${{ github.event.issue.number }}
    analysis: ${{ steps.mushu.outputs.analysis }}
    analysis_mode: bug
    
    # Optional
    pr_url: ${{ steps.mushu.outputs.pr_url }}
    has_changes: ${{ steps.mushu.outputs.has_changes }}
    add_labels: "mushu-analyzed"
    remove_labels: "mushu,needs-triage"
```

### Auto-trigger on Bug Issues

```yaml
on:
  issues:
    types: [opened, labeled]

jobs:
  analyze:
    if: |
      (github.event.action == 'opened' && contains(github.event.issue.labels.*.name, 'bug')) ||
      (github.event.action == 'labeled' && github.event.label.name == 'mushu')
```

---

## Slack Integration

### Setup

1. Create Slack Incoming Webhook
2. Add `SLACK_WEBHOOK_URL` secret

### Action Reference

```yaml
- uses: liorkesten/mushu/integrations/slack@v1
  with:
    # Required
    slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    ticket_key: PROJ-123
    ticket_summary: "Bug title"
    analysis_mode: bug
    
    # Optional
    ticket_url: https://jira.company.com/browse/PROJ-123
    pr_url: ${{ steps.mushu.outputs.pr_url }}
    has_changes: ${{ steps.mushu.outputs.has_changes }}
    analysis_preview: ${{ steps.mushu.outputs.analysis }}
```

---

## Custom Webhooks

Trigger Mushu from any system:

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_PAT" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{
    "event_type": "mushu-bug",
    "client_payload": {
      "key": "TICKET-123",
      "summary": "Issue title",
      "description": "Full description..."
    }
  }' \
  "https://api.github.com/repos/OWNER/REPO/dispatches"
```

---

## Building Custom Integrations

Use Mushu outputs to build your own:

```yaml
- id: mushu
  uses: liorkesten/mushu@v1
  with:
    # ... inputs

- name: Custom integration
  run: |
    # Access outputs
    ANALYSIS="${{ steps.mushu.outputs.analysis }}"
    PR_URL="${{ steps.mushu.outputs.pr_url }}"
    HAS_CHANGES="${{ steps.mushu.outputs.has_changes }}"
    
    # Post to your system
    curl -X POST https://your-api.com/webhook \
      -d "analysis=$ANALYSIS"
```

---

## Parallel Integrations

Run integrations in parallel for faster workflows:

```yaml
jobs:
  analyze:
    outputs:
      analysis: ${{ steps.mushu.outputs.analysis }}
    steps:
      - uses: liorkesten/mushu@v1
        id: mushu
        # ...

  jira:
    needs: analyze
    steps:
      - uses: liorkesten/mushu/integrations/jira@v1
        # ...

  slack:
    needs: analyze
    steps:
      - uses: liorkesten/mushu/integrations/slack@v1
        # ...
```
