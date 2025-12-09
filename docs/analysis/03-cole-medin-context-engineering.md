# Cole Medin's Context Engineering Framework

## Sources

- [Context Engineering Intro Repository](https://github.com/coleam00/context-engineering-intro)
- [Advanced Claude Code Techniques - GitNation](https://gitnation.com/contents/advanced-claude-code-techniques-agentic-engineering-with-context-driven-development)
- Cole Medin YouTube Channel (@ColeMedin)
- Dynamous Agentic Coding Course

---

## Core Philosophy

> "Context engineering is the new vibe coding - it's the way to actually make AI coding assistants work."

Context Engineering represents a fundamental shift from traditional prompt engineering. Rather than relying on clever wording, it provides:
- **Documentation**: Project structure, conventions, APIs
- **Examples**: Code patterns to follow
- **Rules**: Constraints and requirements
- **Patterns**: Reference implementations
- **Validation**: Automated verification mechanisms

**Key insight**: "Most agent failures aren't model failures - they're context failures."

---

## The PRP Framework

### What is a PRP?

**PRP (Product Requirements Prompt)**: A comprehensive implementation blueprint optimized for AI coding assistants.

Similar to PRDs (Product Requirements Documents), but:
- Crafted specifically for AI instruction
- Include validation gates
- Reference concrete examples
- Specify success criteria

### Three-Step Workflow

```
1. Define requirements → Edit INITIAL.md
2. Generate PRP       → /generate-prp INITIAL.md
3. Execute PRP        → /execute-prp PRPs/your-feature.md
```

---

## Directory Structure

```
project-root/
├── .claude/
│   ├── commands/
│   │   ├── generate-prp.md      # PRP generation logic
│   │   └── execute-prp.md       # Implementation execution
│   └── settings.local.json      # Permissions configuration
├── PRPs/                         # Generated implementation blueprints
├── examples/                     # Critical: code patterns to follow
├── CLAUDE.md                     # Global project rules
├── INITIAL.md                    # Feature request template
└── README.md
```

### examples/ Directory (Critical)

The examples folder is essential for success. Include:

| Category | Purpose |
|----------|---------|
| Code structure patterns | Module organization, imports, class/function design |
| Testing patterns | Test structure, mocking, assertions |
| Integration patterns | API clients, database connections, authentication |
| CLI patterns | Argument parsing, output formatting, error handling |

**More patterns = better implementations**

---

## INITIAL.md Template

Four required sections:

### 1. FEATURE

Specific, detailed description with exact requirements:

```markdown
❌ Poor: "Build a web scraper"

✅ Better: "Async scraper using BeautifulSoup for e-commerce sites
   with rate limiting (max 10 req/sec), PostgreSQL storage,
   retry logic with exponential backoff"
```

### 2. EXAMPLES

References to code patterns in the examples/ folder:

```markdown
See examples/ for:
- CLI implementation pattern: examples/cli/main.py
- Database connection: examples/db/connection.py
- Test structure: examples/tests/test_scraper.py
```

### 3. DOCUMENTATION

External resources:

```markdown
- API docs: https://docs.example.com/api
- BeautifulSoup: https://beautiful-soup-4.readthedocs.io/
- PostgreSQL MCP server: Use for database operations
```

### 4. OTHER CONSIDERATIONS

Constraints and edge cases:

```markdown
- Must handle rate limiting (429 responses)
- Respect robots.txt
- Common pitfall: Anti-bot detection
- Performance: Must process 1000 pages/hour
```

---

## PRP Structure

Generated PRPs include:

### Header
```markdown
# PRP: Feature Name
Generated: YYYY-MM-DD
Confidence: 8/10
```

### Sections

1. **Overview**: Feature summary and goals
2. **Context**: Referenced documentation and examples
3. **Implementation Phases**: Step-by-step with validation gates
4. **Success Criteria**: Measurable completion metrics
5. **Error Handling**: Expected failure modes and responses
6. **Testing Requirements**: Unit, integration, E2E test specs

### Validation Gates

Each phase includes mandatory checks:

```markdown
## Phase 1: Database Schema

### Steps
1. Create migration file
2. Define tables and indexes
3. Run migration

### Validation Gate
- [ ] Migration runs without errors
- [ ] Tables created with correct structure
- [ ] Indexes verified
```

---

## Slash Commands

### /generate-prp

Transforms feature requests into comprehensive blueprints:

1. **Research Phase**: Analyze codebase patterns and conventions
2. **Documentation Gathering**: Collect relevant API docs and resources
3. **Blueprint Creation**: Develop step-by-step implementation
4. **Quality Check**: Assign confidence scoring (1-10)

### /execute-prp

Implements features from generated PRPs:

1. Load complete PRP context
2. Create detailed task list
3. Execute implementation components
4. Validate with tests and linting
5. Iterate until requirements met

---

## Best Practices

### 1. Be Explicit

Don't assume AI preferences:

```markdown
❌ "Use best practices for error handling"

✅ "Catch specific exceptions (ConnectionError, TimeoutError).
   Log with context. Retry 3 times with exponential backoff.
   Raise custom AppError on final failure."
```

### 2. Maximize Examples

Show both correct and incorrect approaches:

```python
# ✅ CORRECT - Explicit error handling
try:
    response = await client.fetch(url)
except ConnectionError as e:
    logger.error(f"Connection failed: {e}", exc_info=True)
    raise FetchError(f"Failed to fetch {url}") from e

# ❌ INCORRECT - Generic exception handling
try:
    response = await client.fetch(url)
except Exception:
    print("Error")
```

### 3. Use Validation Gates

Ensure working code on first attempt:

```markdown
### Phase 2: API Client

**Validation Gate** (must pass before proceeding):
- [ ] `npm test src/api/client.test.ts` passes
- [ ] `npm run type-check` succeeds
- [ ] Manual curl test returns expected response
```

### 4. Leverage Documentation

Reference official docs directly:

```markdown
## Documentation Context

### Required Reading
- PostgreSQL MCP: For all database operations
- See: https://docs.example.com/pg-mcp

### API Reference
- Authentication: https://api.service.com/docs/auth
- Rate Limits: https://api.service.com/docs/limits
```

### 5. Customize CLAUDE.md

Add project-specific conventions:

```markdown
# Project Conventions

## Error Handling
- Never swallow exceptions silently
- Always log with structured data
- Use custom error classes from src/errors/

## Testing
- Coverage minimum: 80%
- Use factories for test data
- Mock external services

## Git
- Branch prefix: feature/, bugfix/, hotfix/
- Conventional commits required
```

---

## No Vibe Coding Rule

> "Validate PRPs before executing them and the code after execution!"

**Anti-pattern**: Vibe coding—running generated code without verification.

**Correct pattern**:
1. Review generated PRP for completeness
2. Check referenced examples exist
3. Verify documentation links work
4. Execute with validation gates
5. Run full test suite after completion

---

## Success Metrics

Context Engineering delivers:

| Metric | Impact |
|--------|--------|
| Reduced failures | Address context gaps, not model limitations |
| Consistency | Adherence to project patterns |
| Complex implementations | Multi-step features succeed |
| Self-correction | Validation loops catch issues early |

---

## Comparison: Context Engineering vs Prompt Engineering

| Aspect | Prompt Engineering | Context Engineering |
|--------|-------------------|---------------------|
| Focus | Clever wording | Comprehensive context |
| Scalability | Limited | Project-wide |
| Reliability | Variable | Predictable |
| Maintenance | Per-prompt | Systematic |
| Learning curve | Low | Moderate |
| Effectiveness | Good for simple tasks | Excellent for complex tasks |

---

## Integration with Remote Agentic Coding System

### Current State

The repository has `.claude/commands/` with:
- `core_piv_loop/` - Plan-implement-verify commands
- `github_bug_fix/` - Issue resolution workflow
- `validation/` - Quality assurance commands
- `end-to-end-feature.md` - Feature development

### Recommended Enhancements

1. **Add PRPs/ directory** for implementation blueprints
2. **Create examples/ directory** with project patterns
3. **Implement INITIAL.md template** for feature requests
4. **Add /generate-prp and /execute-prp** commands
5. **Document validation gates** in existing commands

---

## Key Takeaways

1. **Context > Prompts**: Provide comprehensive context, not clever prompts
2. **Examples are critical**: The more patterns, the better the output
3. **Validation gates**: Mandatory checks prevent incomplete implementations
4. **PRP framework**: Structured approach from requirements to execution
5. **No vibe coding**: Always verify before and after execution
