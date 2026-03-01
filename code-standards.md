# Code Standards

**Applies to:** Python only 🐍

This document defines our code quality standards. All code - whether human or AI-generated - must meet these standards before merging.

**AI agents** should use this document when generating or reviewing code.

**If you used AI to generate code:** You are responsible for reviewing the output. Treat AI code as a first draft that requires human oversight.

---

## TL;DR - The Golden Rules

1. **No hardcoded secrets** - use environment variables
2. **No hardcoded paths/URLs** - use constants or config files
3. **No SQL string concatenation** - use parameterized queries
4. **Type hints everywhere** - use TypedDict/Pydantic for structured data
5. **Use enums for status fields** - avoid stringly-typed code
6. **Tests for new code** - with descriptive names
7. **No files over 600 lines** - split into smaller modules
8. **No print()** - use logging with `%s` formatting
9. **Handle edge cases** - nulls, empty inputs, boundaries
10. **Don't swallow errors** - catch specific exceptions, add context
11. **LLM outputs must use structured schemas** - no raw text parsing (see Section 18)

---

## Pre-Submission Checklist

Before submitting or approving code, verify:

- [ ] All imports are at the top of the file (after docstring)
- [ ] No typos in function names, variable names, or class names
- [ ] No duplicate code or functions across the codebase
- [ ] All defined constants are actually used
- [ ] No catch-and-rethrow exceptions without adding value
- [ ] Code follows DRY principle - no copy-pasted logic
- [ ] Docstrings present for public functions/classes
- [ ] Type hints added for function signatures (use TypedDict/dataclass for structured data)
- [ ] No dead code or commented-out blocks
- [ ] Tests added/updated for new functionality
- [ ] No mutable default arguments (lists, dicts, sets)
- [ ] Using f-strings for string formatting
- [ ] Using pathlib for file paths (not os.path)
- [ ] All resources use context managers (with statements)
- [ ] Using logging, not print statements
- [ ] No assert statements for runtime validation
- [ ] No hardcoded secrets (use environment variables)
- [ ] No SQL string concatenation (use parameterized queries)
- [ ] Input validation at system boundaries (API endpoints, webhooks, file uploads)
- [ ] No insecure crypto (no MD5/SHA1 for passwords, no custom encryption)
- [ ] Async code uses async libraries (aiohttp, not requests)
- [ ] Enums used for fixed sets of values (not strings)
- [ ] HTTP status codes use named constants (e.g., `status.HTTP_400_BAD_REQUEST`)
- [ ] Test names describe the scenario being tested
- [ ] Edge cases handled (empty inputs, nulls, boundary conditions)
- [ ] No N+1 queries (batch/JOIN instead of query-per-item loops)
- [ ] Auth checks in place for protected operations
- [ ] Sensitive data not logged or exposed in error messages
- [ ] No hardcoded file paths, URLs, or external resource locations (use constants/env vars)
- [ ] PR has a clear, descriptive title (not just ticket number)
- [ ] PR description explains what/why/how
- [ ] LLM outputs use structured output (Pydantic models) - no raw text parsing (Section 18)

---

## Common Issues & Anti-Patterns

### 1. Imports at Top, DRY, Constants, Naming, Style

These are basic Python hygiene - follow PEP 8:

- **Imports:** Always at module level, never inside functions
- **DRY:** If you copy-paste code, extract it into a reusable function
- **Constants:** Define them, then actually use them. No magic numbers.
- **Naming:** `process_order()` not `do_stuff()`. Be descriptive.
- **Style:** snake_case for functions/variables, use `is None` not `== None`
- **Spelling:** Check for typos in function names, variables, and class names

#### Catching Typos in Identifiers

❌ **Typos in names cause confusion and bugs:**
```python
def print_EPRA_summarrry():  # typo: "summarrry"
    pass

def calcualte_total():  # typo: "calcualte"
    pass

user_assesment = {}  # typo: "assesment"
```

✓ **Review names carefully:**
```python
def print_epra_summary():  # correct
    pass

def calculate_total():  # correct
    pass

user_assessment = {}  # correct
```

**Tips:**
- Enable spell-checking in your IDE (most have plugins for code spell-check)
- Read function/variable names out loud during PR review
- Watch for common typos: `recieve`, `occured`, `seperate`, `definately`, `enviroment`

#### Don't Duplicate Code - Reuse First

Before writing new code, **search the codebase** for existing implementations.

❌ **Bad - duplicating logic that already exists:**
```python
# In file_a.py
def get_bearer_token():
    return f"Bearer {app_config.secret.token}"

# In file_b.py (copy-pasted!)
def get_auth_header():
    return f"Bearer {app_config.secret.token}"
```

✓ **Good - reuse the existing function:**
```python
# In file_b.py
from src.util.auth import get_bearer_token
headers = {"Authorization": get_bearer_token()}
```

#### Use the Same Constants Repo-Wide

Don't redefine the same constant in multiple files. See **Section 13** for more examples.

❌ **Bad - same constant defined in multiple places:**
```python
# In service_a.py
DEFAULT_TIMEOUT = 30

# In service_b.py
TIMEOUT_SECONDS = 30  # same value, different name!
```

✓ **Good - single source of truth:**
```python
# In src/constants.py
DEFAULT_TIMEOUT_SECONDS = 30

# In service_a.py and service_b.py
from src.constants import DEFAULT_TIMEOUT_SECONDS
```

---

### 2. Catch-and-Rethrow Without Adding Value

❌ **Bad:**
```python
try:
    return json.load(f)
except Exception as e:
    raise e  # Pointless!
```

✓ **Good - Add Context:**
```python
try:
    return json.load(f)
except FileNotFoundError:
    logger.error(f"Config file not found: {path}")
    raise ConfigError(f"Missing config file: {path}")
```

✓ **Or Don't Catch:** Let exceptions bubble up naturally if you're not adding value.

---

### 3. Overly Broad Exception Handling

❌ **Bad:**
```python
try:
    user = db.get_user(user_id)
    result = calculate_score(user)
    send_email(user.email, result)
except Exception:  # Catches everything including bugs!
    return None
```

✓ **Good:** Catch specific exceptions, handle each appropriately:
```python
try:
    user = db.get_user(user_id)
except UserNotFoundError:
    logger.warning(f"User {user_id} not found")
    return None

try:
    send_email(user.email, result)
except EmailError as e:
    logger.error(f"Email failed: {e}")  # Continue anyway
```

---

### 4. Missing Type Hints

❌ **Vague:**
```python
def calculate_total(items: list[dict], tax_rate: float) -> float:
```

✓ **Better - Use TypedDict for structured data:**
```python
from typing import TypedDict

class Item(TypedDict):
    price: float
    name: str

def calculate_total(items: list[Item], tax_rate: float) -> float:
    """Calculate total price including tax."""
    return sum(item['price'] for item in items) * (1 + tax_rate)
```

---

### 5. Mutable Default Arguments

❌ **Classic gotcha:**
```python
def append_item(item, items=[]):  # List persists between calls!
    items.append(item)
    return items
```

✓ **Good:**
```python
def append_item(item, items: list | None = None):
    if items is None:
        items = []
    items.append(item)
    return items
```

---

### 6. Use f-strings and pathlib

```python
# ❌ Old: "Hello, %s" % name / "Hello, {}".format(name)
# ✓ New:
f"Hello, {name}"

# ❌ Old: os.path.join(base, "config", "settings.json")
# ✓ New:
from pathlib import Path
path = Path(base) / "config" / "settings.json"
```

---

### 7. Context Managers for Resources

```python
# ❌ Bad: f = open("file.txt"); data = f.read(); f.close()
# ✓ Good:
with open("file.txt") as f:
    data = f.read()
```

Use `with` for files, connections, locks - anything needing cleanup.

---

### 8. Logging Best Practices

❌ **Bad:**
```python
print(f"Error: {e}")  # No timestamps, levels, or log files
logger.debug(f"Processing {expensive_call()}")  # Always evaluates expensive_call()!
```

✓ **Good:**
```python
logger.debug("Processing user %s", user_id)  # Lazy evaluation
logger.exception("Failed to process")  # Auto-includes traceback
```

**When `%s` matters:** Use `%s` formatting for DEBUG logs or when interpolating expensive operations. For simple INFO/ERROR logs with cheap variables, f-strings are fine.

---

### 9. Don't Use `assert` for Runtime Validation

❌ **Bad:** `assert user is not None` - stripped with `python -O`!

✓ **Good:**
```python
if user is None:
    raise ValueError("User required")
```

---

### 10. Security Concerns

#### SQL Injection
❌ **Bad:**
```python
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
query = "SELECT * FROM users WHERE name = '" + name + "'"
```

✓ **Parameterized queries:**
```python
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

#### Hardcoded Credentials / Sensitive Data
❌ **Secrets in code:**
```python
API_KEY = "sk-abc123"
DB_PASSWORD = "supersecret"
conn_string = "postgres://user:password123@host/db"
```

✓ **Environment variables:**
```python
API_KEY = os.environ["API_KEY"]
DB_PASSWORD = os.environ.get("DB_PASSWORD")  # Or use a secrets manager
```

#### Missing Input Validation at System Boundaries
❌ **Trusting external input:**
```python
def process_webhook(request):
    user_id = request.json["user_id"]  # No validation!
    amount = request.json["amount"]    # Could be negative, huge, or non-numeric
    transfer_money(user_id, amount)
```

✓ **Validate at boundaries:**
```python
from pydantic import BaseModel, Field, validator

class TransferRequest(BaseModel):
    user_id: int = Field(gt=0)
    amount: float = Field(gt=0, le=10000)

def process_webhook(request):
    data = TransferRequest(**request.json)  # Validates & fails fast
    transfer_money(data.user_id, data.amount)
```

#### Insecure Cryptographic Practices
❌ **Weak/broken crypto:**
```python
import hashlib
hashed = hashlib.md5(password.encode()).hexdigest()  # MD5 is broken!
hashed = hashlib.sha1(password.encode()).hexdigest()  # SHA1 is weak for passwords
```

✓ **Use proper password hashing:**
```python
from passlib.hash import bcrypt
hashed = bcrypt.hash(password)
is_valid = bcrypt.verify(password, hashed)
```

❌ **Rolling your own crypto:**
```python
def encrypt(data, key):
    return bytes([a ^ b for a, b in zip(data, key)])  # XOR "encryption" 🤦
```

✓ **Use established libraries:**
```python
from cryptography.fernet import Fernet
f = Fernet(key)
encrypted = f.encrypt(data)
```

#### Unsafe Deserialization
❌ **Pickle with untrusted data:**
```python
data = pickle.loads(untrusted_input)  # Remote code execution risk!
```

✓ **Safe formats:**
```python
data = json.loads(untrusted_input)  # Safe
```

---

### 11. Async/Await Gotchas

❌ **Blocking the event loop:**
```python
async def fetch_data():
    time.sleep(5)  # BLOCKS everything!
    requests.get(url)  # Also blocks!
```

✓ **Non-blocking:**
```python
async def fetch_data():
    await asyncio.sleep(5)
    async with aiohttp.ClientSession() as session:
        await session.get(url)
```

❌ **Forgetting to await:** `fetch_data()` returns a coroutine, doesn't run!

✓ **Always await:** `await fetch_data()`

---

### 12. Dataclasses vs Pydantic vs TypedDict

| Use Case | Choice |
|----------|--------|
| External JSON/dict data | TypedDict |
| Internal domain objects | dataclass |
| API request/response | Pydantic |
| Config file parsing | Pydantic |
| Need validation | Pydantic |
| Need methods on objects | dataclass |

---

### 13. Use Enums and Constants for Repeated Values

❌ **Stringly typed:**
```python
if status == "pendng":  # Typo - silent bug!
```

✓ **Enum:**
```python
from enum import Enum

class Status(Enum):
    PENDING = "pending"
    ACTIVE = "active"

if status == Status.PENDNG:  # NameError - caught immediately!
```

#### When to Use Enums vs Constants vs Inline Strings

| Scenario | Use | Example |
|----------|-----|--------|
| Entity status fields | **Enum** | `PENDING`, `PROCESSING`, `COMPLETED` |
| Fixed set of options | **Enum** | `UserRole.ADMIN`, `UserRole.VIEWER` |
| API response codes | **Enum** | `ResponseCode.SUCCESS`, `ResponseCode.ERROR` |
| Dict keys used in 2+ places | **Constant** | `FIELD_EMAIL = "emailId"` |
| Config/feature flag names | **Constant** | `FLAG_HISTORIC_SEARCH = "epra.historic.search.enabled"` |
| Magic numbers with meaning | **Constant** | `DEFAULT_CONFIDENCE_SCORE = 95` |
| One-off local string | **Inline OK** | `logger.info("Starting process")` |

#### Entity Status Fields - Always Use Enum

❌ **Bad - typos waiting to happen:**
```python
update_status(epra_guid, status='RESPONSE_GENERATED_WITH_GIVEN_RESOURCE')
update_status(epra_guid, status='PROCESING')  # typo!
```

✓ **Good - type-safe with autocomplete:**
```python
class AgentProcessingStatus(Enum):
    PENDING = "PENDING"
    PROCESSING = "PROCESSING"
    RESPONSE_GENERATED = "RESPONSE_GENERATED_WITH_GIVEN_RESOURCE"
    COMPLETED = "COMPLETED"
    FAILED = "FAILED"

update_status(epra_guid, status=AgentProcessingStatus.RESPONSE_GENERATED)
```

#### Dictionary Keys - Extract When Reused

❌ **Bad - same key in multiple places:**
```python
# In file A
payload = {"emailId": user_email, "assessment_id": id}

# In file B  
email = response.get("emailid")  # typo: "emailid" vs "emailId"
```

✓ **Good - single source of truth:**
```python
# constants.py
FIELD_EMAIL_ID = "emailId"
FIELD_ASSESSMENT_ID = "assessment_id"

# In file A
payload = {FIELD_EMAIL_ID: user_email, FIELD_ASSESSMENT_ID: id}

# In file B
email = response.get(FIELD_EMAIL_ID)  # Can't typo, IDE autocompletes
```

#### Don't Overdo It

❌ **Over-engineering:**
```python
LOG_MESSAGE_STARTING = "Starting process"  # Overkill
logger.info(LOG_MESSAGE_STARTING)
```

✓ **Just use the string:**
```python
logger.info("Starting process")  # One-off, local, no risk of typo mattering
```

**Rule of thumb:** If a string represents a *contract* (API field, status, config key) or is used in 2+ places, extract it. If it's just a log message or one-off label, leave it inline.

#### HTTP Status Codes - Use Named Constants

❌ **Bad - magic numbers:**
```python
raise HTTPException(status_code=500, detail="Internal server error")
raise HTTPException(status_code=400, detail="Bad request")
return JSONResponse(status_code=404, content={"error": "Not found"})
```

✓ **Good - use FastAPI/Starlette status constants:**
```python
from fastapi import status

raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Internal server error")
raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Bad request")
return JSONResponse(status_code=status.HTTP_404_NOT_FOUND, content={"error": "Not found"})
```

**Why:** Magic numbers like `500`, `400`, `404` are easy to typo and unclear. Named constants are self-documenting and IDE-autocomplete friendly.

---

### 14. No Hardcoded Paths, URLs, or External Resource Locations

File paths, API URLs, and external resource locations should **never** be hardcoded inline. They change between environments and are easy to typo.

❌ **Bad - hardcoded inline:**
```python
props = read_properties_file("/etc/secrets/epra_secrets/application.properties")
response = requests.get("https://api.prod.walmart.com/v1/users")
```

✓ **Good - externalize using ONE of these approaches:**

```python
# Option 1: Constants (simplest - good when path rarely changes)
# constants.py
EPRA_CONFIG_PATH = "/etc/secrets/epra_secrets/application.properties"

# Option 2: Config file (good for multiple environments)
EPRA_CONFIG_PATH = config.get("epra.config.path", "/etc/secrets/epra_secrets/application.properties")

# Option 3: Environment variable (good for container deployments)
EPRA_CONFIG_PATH = os.environ.get("EPRA_CONFIG_PATH", "/etc/secrets/epra_secrets/application.properties")
```

**Then use the constant everywhere:**

```python
# constants.py
EPRA_CONFIG_PATH = "/etc/secrets/epra_secrets/application.properties"
ONETRUST_API_BASE = "https://onetrust.walmart.com/api/v1"

# service.py
from constants import EPRA_CONFIG_PATH, ONETRUST_API_BASE

props = common_util.read_properties_file(EPRA_CONFIG_PATH)
response = requests.get(f"{ONETRUST_API_BASE}/assessments")
```

**The key benefit:** The path is defined in **ONE place**, not scattered inline. Typos are caught once, and it's easy to find and change.

#### Why This Matters

| Problem | Example |
|---------|---------|
| **Typos** | `epra_secretts` vs `epra_secrets` - silent failure |
| **Environment differences** | Prod path ≠ Dev path ≠ Test path |
| **Maintenance** | Changing a URL requires finding all usages |

#### What Should Be Externalized

| Type | Externalize To |
|------|---------------|
| File paths | Constants or config file |
| API URLs | Constants, config file, or env vars |
| Database connection strings | Secrets manager or env vars |
| S3 buckets, Kafka brokers, Redis URLs | Config file or env vars |
| Feature flag names | Constants |

**Rule of thumb:** If it's an external resource location that could differ between environments (dev/stage/prod), externalize it. Define it in ONE place so the IDE can find all usages.

---

### 15. Testing Best Practices

| Rule | Bad | Good |
|------|-----|------|
| Descriptive names | `test1()`, `test_user()` | `test_login_with_invalid_password_returns_401()` |
| Test behavior, not implementation | `assert service._hash_password("x") == "abc"` | `assert user.verify_password("secret") is True` |
| One concept per test | Giant test checking 15 things | Focused tests, one assertion each |

#### Method Names Must Match Actual Behavior

Names should describe what the function **actually does** - not just the data type it handles.

| ❌ Bad | ✓ Good | Why |
|--------|--------|-----|
| `handle_string(value)` | `normalize_whitespace(value)` | "Handle" how? |
| `process_list(items)` | `filter_active_items(items)` | "Process" to do what? |
| `validate_email()` that also sends email | `validate_and_send_confirmation()` | Hidden side effect |
| `test_check_fails_string` | `test_check_fails_when_value_is_wrong` | Fails because string, or when given string? |

**Rule of thumb:** A developer should understand what a function does just by reading its name.

---

### 16. No Single-Line If Statements

❌ **Bad:** `if x: return y`

✓ **Good:**
```python
if x:
    return y
```

**Why:** Readability, debugging (breakpoints), git blame, consistency.

---

### 17. No Inline Classes

Define classes at module level, not inside functions or other classes.

| ❌ Bad | ✓ Good | Why |
|--------|--------|-----|
| `def foo(): class Bar: ...` | `class Bar: ...` at module level | Hard to test, reuse, mock, and discover |

---

### 18. LLM Structured Output Standard

**Rule:** All LLM responses must use structured output (Pydantic models). No raw text parsing.

#### Why This Matters

- **Type safety** - Catch schema errors at parse time, not runtime
- **No regex hacking** - Stop extracting JSON from markdown code blocks
- **Auto-validation** - Pydantic validates types, constraints, and required fields
- **Retries work** - Frameworks can auto-retry on schema validation failures

#### ❌ Don't Do This

```python
# Splitting/parsing raw text
for line in response.content.split('\n'):
    if line.startswith('Score:'):
        score = float(line.split(':')[1])

# Stripping markdown to get JSON
cleaned = content.replace('```json', '').replace('```', '')
data = json.loads(cleaned)

# Regex to find JSON in response
json_match = re.search(r'\{[\s\S]*\}', content)
data = json.loads(json_match.group())

# Prompts asking for formatted text
prompt = "Format as: Score: [value]\nReason: [text]"
```

#### ✅ Do This Instead

```python
from pydantic import BaseModel, Field

# 1. Define your schema
class MyResponse(BaseModel):
    score: float = Field(ge=0, le=1)
    reason: str

# 2. Use structured output (pick one based on your framework)

# LangChain
structured_llm = llm.with_structured_output(MyResponse)
result = structured_llm.invoke(prompt)  # Returns MyResponse object

# OpenAI
response = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[{"role": "user", "content": prompt}],
    response_format=MyResponse
)
result = response.choices[0].message.parsed

# Pydantic AI
agent = Agent('openai:gpt-4o', result_type=MyResponse)
result = await agent.run(prompt)
```

#### Framework Reference

| Framework | Method | Docs |
|-----------|--------|------|
| **LangChain** | `llm.with_structured_output(Model)` | [Docs](https://python.langchain.com/docs/how_to/structured_output/) |
| **OpenAI** | `response_format=Model` | [Docs](https://platform.openai.com/docs/guides/structured-outputs) |
| **Pydantic AI** | `Agent(result_type=Model)` | [Docs](https://ai.pydantic.dev/) |
| **Gemini** | `response_schema=Model` | [Docs](https://ai.google.dev/gemini-api/docs/structured-output) |
| **Google ADK** | `output_schema=Model` | [Docs](https://google.github.io/adk-docs/agents/llm-agents/) |

#### Quick Reference

| 🚫 Reject | ✅ Approve |
|-----------|------------|
| `response.content.split()` | `with_structured_output(Model)` |
| `re.search(r'\{.*\}', text)` | `response_format=Model` |
| `.replace('```json', '')` | `result_type=Model` |
| `json.loads(raw_text)` | `.message.parsed` |
| `if line.startswith('Score:')` | `Field(ge=0, le=1)` |
| `except: return default` | Let validation errors raise |

#### Find Violations

```bash
rg "response\.content\.split" --type py
rg "startswith.*Score" --type py
rg "replace.*json" --type py
rg "re\.search.*\\{" --type py
```

---

## AI-Generated Code: Special Considerations

Watch for:
1. **Over-engineering:** Unnecessary abstractions when a simple function would do
2. **Verbose error handling:** Try-except blocks that don't add value
3. **Outdated patterns:** Deprecated libraries or old syntax
4. **Missing edge cases:** Happy path only, no error handling
5. **Inconsistent with codebase:** Doesn't follow your existing patterns
6. **Unnecessary comments:** Comments for obvious code
7. **Not using existing utilities:** Rewrites what already exists

**Your responsibility:** Verify it works, ensure it follows standards, remove AI "slop".

---

## Review Process

### For Code Authors
1. Run linters/formatters before submitting
2. Review your own diff first
3. Check this document for common mistakes
4. Write a clear PR title and description (see below)

#### PR Title & Description Guidelines

**PR Title** should be concise and descriptive:
- ❌ Bad: `fix`, `update`, `changes`, `GBSCENG-12345`, `WIP`
- ❌ Bad: `Gbsceng 12345 assessment data` (what does this even mean?)
- ✅ Good: `fix: handle null response in user lookup`
- ✅ Good: `feat: add bulk upsert for assessment records`
- ✅ Good: `refactor: split epra_service into smaller modules`

**PR Description** should answer:
- **What** changed? (brief summary)
- **Why** did it change? (link to ticket, explain the problem)
- **How** to test? (if not obvious)
- **Any risks?** (breaking changes, migrations, etc.)

A reviewer should understand the PR's purpose without reading the code.

### For Reviewers

Focus on these key areas:

| Area | Look For |
|------|----------|
| **Correctness** | Logic errors, edge cases (empty/null/boundary), race conditions |
| **Security** | Injection, auth issues, data exposure (see Section 10) |
| **Performance** | N+1 queries, memory leaks, O(n²) algorithms |
| **Maintainability** | Code clarity, DRY violations, SOLID principles |
| **Testing** | Coverage gaps, missing edge case tests, brittle tests |
| **LLM Integration** | Raw text parsing, missing structured output, silent fallbacks (see Section 18) |

**Don't focus on:** Nitpicky style issues (linters catch these), personal preferences

---

### Quick Grep Commands for Finding Violations

Run these from your project root to find common violations:

```bash
# LLM Structured Output Violations (Section 18)
rg "response\.content\.split" --type py             # Raw text splitting
rg "startswith\(['\"]Score" --type py                # Parsing "Score:" from text
rg "replace\(['\"]\`\`\`json" --type py              # Stripping markdown code blocks
rg "re\.search.*\\{.*\\}" --type py                  # Regex JSON extraction

# HTTP Status Code Violations (Section 13)
rg "status_code=\d{3}" --type py                    # Magic number status codes

# File Size Check
wc -l src/**/*.py | sort -n | tail -20              # Find largest files (>600 lines)

# print() statements
rg "^\s*print\(" --type py                          # print() instead of logging

# os.path instead of pathlib
rg "os\.path\.join" --type py                        # Should use pathlib

# Broad exception handling
rg "except Exception:" --type py                    # Overly broad catch
rg "except Exception as" --type py                  # Check if context added
```

---

## Questions?

If something is unclear or you think a standard should change, raise it in your PR review comments. This is a living document.

**Remember:** These standards exist to make our codebase maintainable and bug-free. Each one prevents real problems we've encountered.
