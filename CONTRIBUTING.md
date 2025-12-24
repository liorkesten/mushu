# Contributing to Mushu 🐉

First off, thank you for considering contributing to Mushu! It's people like you that make Mushu such a great tool.

## Code of Conduct

By participating in this project, you are expected to uphold our Code of Conduct: be respectful, inclusive, and constructive.

## How Can I Contribute?

### 🐛 Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates.

When creating a bug report, include:

- **Clear title** describing the issue
- **Steps to reproduce** the behavior
- **Expected behavior** vs actual behavior
- **Screenshots or logs** if applicable
- **Environment details** (GitHub runner, etc.)

### 💡 Suggesting Features

Feature requests are welcome! Please provide:

- **Use case** - Why do you need this feature?
- **Proposed solution** - How should it work?
- **Alternatives** - What workarounds exist?

### 🔧 Pull Requests

1. Fork the repo and create your branch from `main`
2. Make your changes
3. Test your changes thoroughly
4. Update documentation if needed
5. Submit a pull request

#### PR Guidelines

- Keep changes focused and minimal
- Follow existing code style
- Write clear commit messages
- Update README if adding features
- Add examples for new functionality

## Development Setup

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/mushu.git
cd mushu

# Create a branch
git checkout -b feature/amazing-feature
```

### Testing Locally

You can test the action locally using [act](https://github.com/nektos/act):

```bash
act -j analyze -s ANTHROPIC_API_KEY=your-key
```

Or trigger via GitHub's workflow_dispatch.

## Style Guide

### YAML

- Use 2-space indentation
- Quote strings when they contain special characters
- Add comments for complex logic

### Prompts

- Keep prompts clear and concise
- Use consistent formatting
- Include examples in output format sections

### Documentation

- Use clear, simple language
- Include code examples
- Keep README sections focused

## Project Structure

```
mushu/
├── action.yml          # Main action definition
├── prompts/            # Analysis prompt templates
│   ├── bug.md
│   ├── explore.md
│   └── fix.md
├── examples/           # Example workflows
├── docs/              # Additional documentation
└── README.md
```

## Questions?

Feel free to open an issue for any questions!

---

Thank you for contributing! 🐉

