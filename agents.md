# AI Agent Authority (agents.md)

This file defines the **authoritative instruction set** for all AI agents
generating, modifying, or reviewing code in this repository.

AI agents MUST read and follow the documents referenced below.

---

## File Location

This file MUST be named `agents.md` (or `AGENTS.md`) and placed in the repository root.
AI agents should automatically look for this file when working with a repo.

---

## Available Skills

| Task | Instruction File | Description |
|------|------------------|-------------|
| PR Review | `code-standards.md` | Review PRs against team standards |
| Code Generation | `code-standards.md` | Generate code following repo patterns |
| Architecture Context | `README.md` | Understand project purpose and setup |

---

## Order of Authority

When generating or modifying code, follow instructions in this order:

1. User instructions (if explicit and unambiguous)
2. This file (`agents.md`)
3. Referenced instruction files (`code-standards.md`, `README.md`)
4. Repository source code
5. General best practices

If instructions conflict:
- Higher-ranked instructions win
- If ambiguity remains, stop and ask

---

## Mandatory Instruction Files

AI agents MUST load and comply with the following files:

### 1. Code Authoring & Review Rules
→ `code-standards.md`

Defines:
- How to contribute code in this repo
- What reviewers will reject
- Review checklist
- Quality bar for acceptance

If review rules conflict with other files, **code-standards.md wins**.

#### PR Review Format (for AI agents)

When reviewing PRs, AI agents should:
- **Use inline comments only** - post comments directly on the relevant lines of code
- **Do NOT post summary comments** - avoid repetitive overview comments that duplicate inline feedback
- Keep comments concise and actionable
- Reference the relevant section from `code-standards.md` (e.g., "Section 13")

---

### 2. Repository Context & Invariants
→ `README.md`

Defines:
- What business outcome is this repo trying to solve
- Any system architecture background
- How to setup the repo and run the code

---

## Non-Negotiable Global Rules

These apply regardless of which file is being followed:

- Do not hallucinate APIs, files, or behavior
- Do not guess intent
- Do not duplicate logic
- Do not introduce unused code
- Do not silently swallow errors

If unsure, stop and ask.

---

## Self-Check Requirement

Before submitting code, the agent must verify:

- [ ] All referenced instruction files were followed
- [ ] No conflicting rules were violated
- [ ] No assumptions were made without confirmation
- [ ] Changes are minimal and focused

Failure to meet these requirements means the code should NOT be submitted.

---

## Enforcement

This file is authoritative for all AI-generated code.

If an AI agent cannot comply, it must:
1. Explain why
2. Ask for clarification
3. Produce no code

---
