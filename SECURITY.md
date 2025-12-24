# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.x     | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

If you discover a security vulnerability in Mushu, please report it responsibly:

1. **Do NOT** open a public GitHub issue
2. Email security concerns to: security@zafran.io
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

## Response Timeline

- **Acknowledgment**: Within 48 hours
- **Initial assessment**: Within 1 week
- **Fix timeline**: Depends on severity

## Security Considerations

### API Keys

- Store API keys in GitHub Secrets, never in code
- Rotate keys if potentially exposed
- Use minimum required permissions

### AI-Generated Code

- Always review AI-generated PRs before merging
- Use draft PRs by default (`pr_draft: true`)
- Add required reviewers for AI-generated changes
- Consider limiting `fix` mode to trusted sources

### Repository Access

- Mushu only accesses code in the checked-out repository
- In `bug` and `explore` modes: read-only access
- In `fix` mode: writes to a new branch only
- No access to secrets or environment variables beyond what's explicitly passed

### Issue Tracker Integration

- Jira/GitHub tokens should have minimal required permissions
- Jira: Use API tokens, not passwords
- GitHub: Use fine-grained PATs when possible

## Best Practices

1. **Review all AI changes**: Never auto-merge AI-generated PRs
2. **Limit fix mode**: Only use on tickets from trusted sources
3. **Audit logs**: Monitor GitHub Actions logs for unusual activity
4. **Regular rotation**: Rotate API keys periodically
5. **Least privilege**: Grant minimum required permissions

## Acknowledgments

We appreciate security researchers who help keep Mushu safe. Contributors will be acknowledged here (with permission).

