# Anthropic Best Practices for Agentic Coding

## Sources

This document synthesizes best practices from official Anthropic resources:
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)

---

## Core Principles

### 1. The Agentic Feedback Loop

```
Gather Context → Take Action → Verify Work → Repeat
```

This fundamental pattern drives all effective agentic coding:
1. **Gather context**: Read files, understand patterns, search codebase
2. **Take action**: Write code, execute commands, modify files
3. **Verify work**: Run tests, check output, validate changes
4. **Repeat**: Iterate until success criteria met

### 2. Explore → Plan → Code → Commit Workflow

**Critical pattern**: Prevent premature optimization and wasted effort.

1. **Explore**: Have Claude read relevant files without writing code
2. **Plan**: Request detailed implementation plan using thinking modes
3. **Code**: Implement solution with verification steps
4. **Commit**: Generate commit messages automatically

> "Steps #1-#2 are crucial—without them, Claude tends to jump straight to coding a solution."

---

## Context Engineering

### CLAUDE.md Files

Special markdown files automatically ingested for project context:

**Placement strategy**:
- Repository root: Shared team conventions
- Parent directories: Monorepo configuration
- Child directories: Module-specific rules
- Home folder (`~/.claude/CLAUDE.md`): Personal cross-project rules

**Content guidelines**:
```markdown
# Project: MyApp

## Build Commands
- `npm run build` - Production build
- `npm test` - Run tests

## Code Style
- Use TypeScript strict mode
- Prefer functional components

## Testing
- All new code requires tests
- Use Jest + React Testing Library

## Architecture
- See docs/architecture.md for overview
```

**Best practices**:
- Keep under 100-200 lines (300 max)
- Universal applicability—avoid task-specific instructions
- Pointers over copies—reference files, don't duplicate
- Use `#` key to have Claude update CLAUDE.md during conversations

### Progressive Disclosure

Instead of cramming everything into CLAUDE.md:
1. Create focused documentation files
2. Reference them from CLAUDE.md
3. Let Claude determine relevance per task

---

## Thinking Modes

Trigger extended thinking with specific phrases:

| Phrase | Thinking Budget |
|--------|-----------------|
| "think" | Base level |
| "think hard" | Moderate |
| "think harder" | High |
| "ultrathink" | Maximum |

Use for complex architectural decisions and multi-step planning.

---

## Test-Driven Development (TDD)

TDD becomes exponentially more powerful with agentic coding:

```
1. Write tests (explicitly state TDD intent)
2. Confirm tests fail without implementation
3. Commit tests
4. Implement code to pass tests
5. Use subagents to verify non-overfitting
6. Commit passing code
```

**Key insight**: Tests provide clear, unambiguous success criteria that agents can work toward.

---

## Subagent Architecture

### Benefits

1. **Context preservation**: Separate context windows prevent pollution
2. **Parallelization**: Multiple subagents work simultaneously
3. **Specialization**: Fine-tuned for specific domains
4. **Reusability**: Share across projects and teams

### Configuration

Subagents defined as Markdown with YAML frontmatter:

```yaml
---
name: code-reviewer
description: Expert code review. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
permissionMode: default
---

You are a senior code reviewer ensuring high standards.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Provide prioritized feedback

Review checklist:
- Code clarity and readability
- Proper error handling
- No exposed secrets
- Good test coverage
```

### Built-in Subagents

| Subagent | Model | Purpose | Tools |
|----------|-------|---------|-------|
| General-purpose | Sonnet | Complex multi-step tasks | All |
| Plan | Sonnet | Research and analysis | Read-only |
| Explore | Haiku | Fast codebase searching | Read-only |

### Multi-Agent Patterns

**Parallel verification**:
- One Claude writes code
- Another Claude reviews/tests
- Separate context windows yield better results

**Git worktrees**:
```bash
git worktree add ../project-feature-a feature-a
```
Run independent Claude sessions on different branches.

---

## Long-Running Agent Harnesses

### Challenge

Long-running agents face a fundamental problem: discrete sessions with no memory of prior work.

### Two-Part Solution

**1. Initializer Agent**:
- Creates `init.sh` for consistent environment startup
- Establishes `claude-progress.txt` for work history
- Makes initial git commit for baseline

**2. Coding Agent**:
- Works on single features incrementally
- Commits with descriptive messages
- Updates progress documentation before session end

### Session Management Pattern

```
1. Check working directory (pwd)
2. Read git logs and progress files
3. Review feature list; select highest-priority
4. Validate current state through testing
5. Implement feature
6. Update progress documentation
7. Commit changes
```

### Key Artifacts

**Feature List (JSON format)**:
- Comprehensive requirements with pass/fail status
- Structured format resists inappropriate modifications

**Progress Documentation**:
- "It is unacceptable to remove or edit tests"
- Prevents regressions across sessions

---

## Hooks and Automation

### Hook Types

| Event | Trigger Point | Use Case |
|-------|---------------|----------|
| PreToolUse | Before tool execution | Block dangerous commands |
| PostToolUse | After tool completion | Auto-format, run linters |
| UserPromptSubmit | User submits prompt | Validate input |
| Stop | Agent finishes responding | Cleanup |

### Configuration

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",
        "hooks": [{
          "type": "command",
          "command": "npm run lint:fix"
        }]
      }
    ]
  }
}
```

### Exit Codes

- **Exit 0**: Continue normally
- **Exit 2**: Block tool call, show error to Claude

---

## Permission Management

### Principle of Least Privilege

> "Permission sprawl is the fastest path to unsafe autonomy."

Configure tool access:
1. Session prompts ("Always allow")
2. `/permissions` command
3. `.claude/settings.json` (version controlled)
4. `--allowedTools` CLI flag

### Subagent Tool Scoping

```yaml
# Read-heavy agents (PM, Architect)
tools: Read, Grep, Glob, Bash

# Implementation agents
tools: Read, Edit, Write, Bash, Glob, Grep
```

---

## Verification Methods

### 1. Rules-Based Feedback

- Type checking (TypeScript > JavaScript)
- Linting with auto-fix
- Compilation errors

### 2. Visual Feedback

- Screenshots for UI verification
- Browser automation (Puppeteer MCP)
- Design mockup comparison

### 3. LLM as Judge

- Another model evaluates output
- Useful for fuzzy requirements
- Higher latency tradeoff

---

## Multi-Agent Economics

### Token Usage

| Mode | Token Multiplier |
|------|-----------------|
| Standard chat | 1x |
| Single agent | ~4x |
| Multi-agent | ~15x |

### Justification Criteria

Multi-agent systems require tasks where value exceeds token cost:
- Research-heavy workflows
- Complex analysis
- Parallel independent tasks

**Poor candidates**: Heavily interdependent coding tasks with limited parallelization.

---

## Performance Optimization

### Speed Considerations

1. **Fast tool responses**: Milliseconds, not seconds
2. **Incremental builds**: Avoid full compilation cycles
3. **Clear feedback**: Explicit error messages

### Context Management

1. Use `/clear` between tasks
2. Implement working scratchpads
3. Compress global state—store plan and key decisions
4. Let subagents handle detailed exploration

---

## Summary: Key Takeaways

1. **Always plan before coding**—explore → plan → code → commit
2. **Use CLAUDE.md strategically**—less is more, under 200 lines
3. **Deploy subagents for complex tasks**—preserve context, enable parallelism
4. **Implement validation loops**—TDD, linting, visual verification
5. **Manage context aggressively**—clear often, use progressive disclosure
6. **Configure permissions carefully**—principle of least privilege
7. **Use hooks for automation**—quality gates without manual intervention
8. **Consider economics**—multi-agent for high-value tasks only
