# Customization Guide

Learn how to customize Mushu for your specific needs.

## Custom Prompts

You can customize the prompts Mushu uses by:

1. **Using `custom_prompt` input** - Add extra instructions
2. **Forking and modifying** - Full control over prompts

### Adding Custom Instructions

```yaml
- uses: zafran-io/mushu@v1
  with:
    # ... other inputs
    custom_prompt: |
      Additional context for this codebase:
      - We use Go 1.21 with generics
      - Follow the error handling patterns in pkg/errors
      - Database models are in internal/models
```

### Creating Custom Prompt Files

Fork the repo and modify files in `prompts/`:

```markdown
# prompts/bug.md

You are Mushu, our team's AI assistant.

## Team Context
- We use React 18 with TypeScript
- State management: Zustand
- Testing: Vitest + Testing Library

## Your Task
{{TICKET_KEY}}: {{TICKET_SUMMARY}}

... rest of prompt
```

## Analysis Mode Selection

### Automatic Mode Detection

Create smart workflows that detect the right mode:

```yaml
- name: Detect mode
  id: detect
  run: |
    LABELS='${{ toJSON(github.event.issue.labels.*.name) }}'
    
    if echo "$LABELS" | grep -q "mushu-fix"; then
      echo "mode=fix" >> $GITHUB_OUTPUT
    elif echo "$LABELS" | grep -q "bug"; then
      echo "mode=bug" >> $GITHUB_OUTPUT
    elif echo "$LABELS" | grep -q "enhancement"; then
      echo "mode=explore" >> $GITHUB_OUTPUT
    else
      echo "mode=explore" >> $GITHUB_OUTPUT
    fi
```

### Keyword-Based Detection

```yaml
- name: Detect from title
  id: detect
  run: |
    TITLE="${{ github.event.issue.title }}"
    
    if [[ "$TITLE" =~ ^(fix|bug|error|crash|broken) ]]; then
      echo "mode=bug" >> $GITHUB_OUTPUT
    elif [[ "$TITLE" =~ ^(feat|add|implement|create) ]]; then
      echo "mode=explore" >> $GITHUB_OUTPUT
    else
      echo "mode=explore" >> $GITHUB_OUTPUT
    fi
```

## PR Customization

### Custom PR Labels

```yaml
- uses: zafran-io/mushu@v1
  with:
    pr_labels: "ai-generated,needs-review,size/small"
```

### Draft vs Ready PRs

```yaml
- uses: zafran-io/mushu@v1
  with:
    pr_draft: "false"  # Create ready-for-review PRs
```

### Custom Base Branch

```yaml
- uses: zafran-io/mushu@v1
  with:
    base_branch: "develop"  # PR targets develop instead of main
```

## Timeout Configuration

For large codebases or complex analysis:

```yaml
- uses: zafran-io/mushu@v1
  with:
    timeout_minutes: "30"  # Increase from default 15
```

## Multi-Repo Setup

### Monorepo with Multiple Projects

```yaml
- name: Detect project
  id: project
  run: |
    TITLE="${{ github.event.issue.title }}"
    
    if [[ "$TITLE" =~ \[frontend\] ]]; then
      echo "context=Focus on frontend/ directory" >> $GITHUB_OUTPUT
    elif [[ "$TITLE" =~ \[backend\] ]]; then
      echo "context=Focus on backend/ directory" >> $GITHUB_OUTPUT
    fi

- uses: zafran-io/mushu@v1
  with:
    custom_prompt: ${{ steps.project.outputs.context }}
```

### Cross-Repo Analysis

For analyzing issues that span multiple repos:

```yaml
- uses: actions/checkout@v4
  with:
    repository: org/other-repo
    path: other-repo
    
- uses: zafran-io/mushu@v1
  with:
    custom_prompt: |
      This issue may involve code in both this repo and ./other-repo
      Check both locations for relevant code.
```

## Security Considerations

### Limiting File Access

Add instructions to limit what Mushu can modify:

```yaml
custom_prompt: |
  IMPORTANT: Do NOT modify files in:
  - .github/workflows/
  - secrets/
  - config/production/
```

### Review Requirements

Always create PRs as drafts for human review:

```yaml
pr_draft: "true"
pr_labels: "ai-generated,requires-human-review"
```

## Language-Specific Configuration

### Go Projects

```yaml
custom_prompt: |
  Go-specific guidelines:
  - Use table-driven tests
  - Follow stdlib error patterns
  - Use context for cancellation
  - Prefer composition over inheritance
```

### TypeScript Projects

```yaml
custom_prompt: |
  TypeScript guidelines:
  - Strict mode is enabled
  - Use Zod for runtime validation
  - Prefer const assertions
  - Follow React hooks rules
```

### Python Projects

```yaml
custom_prompt: |
  Python guidelines:
  - Use type hints (Python 3.10+)
  - Follow PEP 8
  - Use dataclasses for DTOs
  - Async-first for I/O operations
```

## Conditional Workflows

### Run Only on Specific Labels

```yaml
jobs:
  analyze:
    if: |
      contains(github.event.issue.labels.*.name, 'needs-analysis') ||
      contains(github.event.issue.labels.*.name, 'mushu')
```

### Skip Certain Issues

```yaml
jobs:
  analyze:
    if: |
      !contains(github.event.issue.labels.*.name, 'wontfix') &&
      !contains(github.event.issue.labels.*.name, 'duplicate')
```

### Different Behavior by Priority

```yaml
- name: Set timeout by priority
  id: config
  run: |
    if [[ "${{ contains(github.event.issue.labels.*.name, 'critical') }}" == "true" ]]; then
      echo "timeout=30" >> $GITHUB_OUTPUT
    else
      echo "timeout=15" >> $GITHUB_OUTPUT
    fi
```

## Output Processing

### Capture Analysis for Further Processing

```yaml
- name: Run Mushu
  id: mushu
  uses: zafran-io/mushu@v1
  with:
    # ... inputs

- name: Process results
  run: |
    ANALYSIS='${{ steps.mushu.outputs.analysis }}'
    
    # Check for specific patterns
    if echo "$ANALYSIS" | grep -q "security"; then
      echo "::warning::Security-related issue detected"
    fi
    
    # Store for later steps
    echo "$ANALYSIS" > analysis.md
```

### Conditional Actions Based on Results

```yaml
- name: Check if fix was made
  if: steps.mushu.outputs.has_changes == 'true'
  run: |
    echo "PR created: ${{ steps.mushu.outputs.pr_url }}"
    # Additional actions...
```

