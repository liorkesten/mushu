# Integrations Guide

This guide covers how to integrate Mushu with various issue trackers and automation platforms.

## Jira Integration

### Setup

1. **Create a Jira API Token**
   - Go to https://id.atlassian.com/manage-profile/security/api-tokens
   - Create a new token for Mushu

2. **Add GitHub Secrets**
   ```
   JIRA_BASE_URL: https://your-company.atlassian.net
   JIRA_EMAIL: your-email@company.com
   JIRA_API_TOKEN: your-api-token
   ```

3. **Use the Jira workflow example**
   Copy `examples/jira-integration.yml` to your repo

### Triggering from Jira

Use Jira Automation or a tool like Workato/Zapier:

#### Auto-analyze Bugs

```
Trigger: Issue created
Condition: Issue type = Bug
Action: Send web request
  URL: https://api.github.com/repos/{owner}/{repo}/dispatches
  Method: POST
  Headers:
    Authorization: token {GITHUB_PAT}
    Accept: application/vnd.github.v3+json
  Body:
    {
      "event_type": "mushu-bug",
      "client_payload": {
        "key": "{{issue.key}}",
        "summary": "{{issue.summary}}",
        "description": "{{issue.description}}"
      }
    }
```

#### Explore on Label

```
Trigger: Issue updated
Condition: Labels contain "mushu"
Action: Send web request with event_type: "mushu-explore"
```

#### Fix with PR on Label

```
Trigger: Issue updated  
Condition: Labels contain "mushu-fix"
Action: Send web request with event_type: "mushu-fix"
```

### Jira Label Flow

| Action | Before | After |
|--------|--------|-------|
| Bug analyzed | - | `mushu-analyzed` |
| Explore complete | `mushu` | `mushu-done` |
| Fix with PR | `mushu-fix` | `mushu-pr-created` |
| Fix no changes | `mushu-fix` | `mushu-no-changes` |

---

## GitHub Issues Integration

Mushu can analyze GitHub Issues natively.

### Setup

1. Copy `examples/github-issues.yml` to `.github/workflows/`
2. Ensure `ANTHROPIC_API_KEY` secret is set

### Triggers

- **Bug issues**: Auto-analyzed when created with `bug` label
- **Explore**: Add `mushu` label to any issue
- **Fix with PR**: Add `mushu-fix` label

---

## Linear Integration

### Setup

1. **Create a Linear API Key**
   - Settings → API → Personal API keys

2. **Add GitHub Secrets**
   ```
   LINEAR_API_KEY: your-api-key
   ```

3. **Create a Linear Webhook**
   Point to a middleware (AWS Lambda, Cloudflare Worker) that:
   - Receives Linear webhook events
   - Triggers GitHub repository dispatch

### Webhook Middleware Example

```javascript
// Cloudflare Worker example
export default {
  async fetch(request, env) {
    const event = await request.json();
    
    // Map Linear event to Mushu mode
    const mode = event.data.labels?.includes('bug') 
      ? 'bug' 
      : event.data.labels?.includes('mushu-fix')
        ? 'fix'
        : 'explore';
    
    await fetch(
      `https://api.github.com/repos/${env.REPO}/dispatches`,
      {
        method: 'POST',
        headers: {
          'Authorization': `token ${env.GITHUB_TOKEN}`,
          'Accept': 'application/vnd.github.v3+json',
        },
        body: JSON.stringify({
          event_type: `mushu-${mode}`,
          client_payload: {
            key: event.data.identifier,
            summary: event.data.title,
            description: event.data.description,
          },
        }),
      }
    );
    
    return new Response('OK');
  },
};
```

---

## Slack Notifications

Add Slack notifications to your workflow:

```yaml
- name: Notify Slack
  if: always()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "🐉 Mushu analyzed ${{ steps.inputs.outputs.key }}",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*${{ steps.inputs.outputs.key }}*: ${{ steps.inputs.outputs.summary }}\n\nMode: `${{ steps.inputs.outputs.mode }}`"
            }
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Workato Recipe Templates

### Recipe 1: Auto-analyze Bugs

```
Trigger: Jira - New issue
  Project: Your Project
  Issue type: Bug

Condition: Labels does not contain "mushu-analyzed"

Action: HTTP - POST request
  URL: https://api.github.com/repos/OWNER/REPO/dispatches
  Headers:
    Authorization: token [GITHUB_PAT]
    Accept: application/vnd.github.v3+json
  Body:
    {
      "event_type": "mushu-bug",
      "client_payload": {
        "key": "[Issue key]",
        "summary": "[Issue summary]",
        "description": "[Issue description]"
      }
    }
```

### Recipe 2: Explore on Label

```
Trigger: Jira - Updated issue
  
Condition: 
  - Labels contains "mushu"
  - Labels does not contain "mushu-done"

Action: HTTP - POST request
  Body: { "event_type": "mushu-explore", ... }
```

### Recipe 3: Fix with PR

```
Trigger: Jira - Updated issue

Condition:
  - Labels contains "mushu-fix"
  - Labels does not contain "mushu-pr-created"

Action: HTTP - POST request
  Body: { "event_type": "mushu-fix", ... }
```

---

## Zapier Integration

Similar to Workato, create Zaps:

1. **Trigger**: New Jira Issue (Bug type)
2. **Filter**: Labels don't contain "mushu-analyzed"
3. **Action**: Webhooks by Zapier → POST
   - URL: GitHub dispatch endpoint
   - Payload: Same as above

---

## Custom Webhooks

For any system that can send webhooks:

```bash
curl -X POST \
  -H "Authorization: token YOUR_GITHUB_PAT" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{
    "event_type": "mushu-bug",
    "client_payload": {
      "key": "TICKET-123",
      "summary": "Bug title here",
      "description": "Full description..."
    }
  }' \
  "https://api.github.com/repos/OWNER/REPO/dispatches"
```

Required permissions for the GitHub PAT:
- `repo` (for private repos)
- `public_repo` (for public repos)

