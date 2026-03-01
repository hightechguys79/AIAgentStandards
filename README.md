# AI Agent Code Standards

This repository contains the **authoritative standards and instructions** for AI agents generating, modifying, or reviewing code across our projects.

---

## 📁 Whats In This Repo

| File | Purpose |
|------|--------|
| [`agents.md`](agents.md) | Defines how AI agents should behave when working with code |
| [`code-standards.md`](code-standards.md) | Defines code quality standards for Python projects |

---

## 🤖 For AI Agents

When working with a repository that references these standards:

1. **Read `agents.md` first** - this defines your authority and behavior
2. **Follow `code-standards.md`** - this defines what code is acceptable
3. **Follow the Order of Authority** - user instructions > agents.md > code-standards.md > source code > general best practices

---

## 👩‍💻 For Developers

### Using These Standards in Your Repo

To adopt these standards in your project:

1. **Copy `agents.md`** to your repository root
2. **Reference `code-standards.md`** in your PR review process
3. **Configure AI tools** (Code Puppy, Copilot, etc.) to read `agents.md`

### Example: Adding to Your Repo

```bash
# Clone this repo
git clone https://gecgithub01.walmart.com/GD-CCPA-DataEnablement/AIAgentCodeStandards.git

# Copy files to your project
cp AIAgentCodeStandards/agents.md /path/to/your/repo/
cp AIAgentCodeStandards/code-standards.md /path/to/your/repo/
```

---

## 📋 Quick Summary of Code Standards

### The Golden Rules

1. No hardcoded secrets - use environment variables
2. No hardcoded paths/URLs - use constants or config files
3. No SQL string concatenation - use parameterized queries
4. Type hints everywhere - use TypedDict/Pydantic for structured data
5. Use enums for status fields - avoid stringly-typed code
6. Tests for new code - with descriptive names
7. No files over 600 lines - split into smaller modules
8. No print() - use logging with `%s` formatting
9. Handle edge cases - nulls, empty inputs, boundaries
10. Dont swallow errors - catch specific exceptions, add context
11. **LLM outputs must use structured schemas** - no raw text parsing

See [`code-standards.md`](code-standards.md) for full details.

---

## 🛠️ Contributing

To update these standards:

1. Create a branch
2. Make your changes
3. Submit a PR with the relevant JIRA ticket (e.g., `GBSCENG-XXXXX`)
4. Get approval from the team

---

## 📞 Questions?

Reach out to the **GG-Agentic-Council** team or raise an issue in this repo.
