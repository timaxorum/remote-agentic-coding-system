# HumanLayer and Human-in-the-Loop Patterns

## Sources

- [HumanLayer GitHub](https://github.com/humanlayer/humanlayer)
- [Human-in-the-Loop Best Practices - Permit.io](https://www.permit.io/blog/human-in-the-loop-for-ai-agents-best-practices-frameworks-use-cases-and-demo)
- [CAMEL-AI HITL Integration](https://www.camel-ai.org/blogs/human-in-the-loop-ai-camel-integration)
- [HumanLayer Blog - Writing CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## Overview

Human-in-the-Loop (HITL) is not a temporary workaround—it's a long-term pattern for building AI agents we can trust. It ensures:
- LLMs stay within safe operational boundaries
- Sensitive actions don't slip through automation
- Teams remain in control as autonomy grows

---

## HumanLayer Platform

### What is HumanLayer?

HumanLayer enables AI agents to communicate with humans in tool-based and async workflows. It guarantees human oversight of high-stakes function calls with approval workflows across Slack, email, and more.

**Key features**:
- Framework-agnostic integration
- Async human decisions
- Multi-channel communication (Slack, Email, Discord)
- Orchestration-agnostic implementation

### CodeLayer (Open Source)

CodeLayer is an open-source IDE orchestrating AI coding agents:
- Built on Claude Code
- Keyboard-first workflows
- Advanced context engineering
- Multi-Claude capability (parallel sessions)
- ~50% token reduction through smart orchestration

---

## When to Use HITL

### Decision Framework

Use HITL whenever **cost of wrong autonomous action > cost of quick review**.

| Risk Level | Action | HITL Required |
|------------|--------|---------------|
| Low | Read files, search code | No |
| Medium | Create files, run tests | Optional |
| High | Delete files, modify prod | Yes |
| Critical | Deploy, credentials, billing | Always |

### Common Scenarios

1. **Access control changes**: User permissions, role assignments
2. **Financial operations**: Payments, refunds, pricing changes
3. **Customer data**: PII access, data export, deletion
4. **Production changes**: Deployments, configuration updates
5. **External integrations**: API key creation, webhook setup

---

## SDK Integration

### Core Decorators

```python
from humanlayer import require_approval, human_as_tool

# Require human approval before execution
@require_approval(contact_channel="slack:#approvals")
def delete_user(user_id: str) -> bool:
    """Delete a user account permanently."""
    return db.delete_user(user_id)

# Human as a tool - agent can ask human for input
@human_as_tool(contact_channel="slack:#support")
def get_human_guidance(question: str) -> str:
    """Ask a human for guidance on a complex decision."""
    pass
```

### Configuration

```python
from humanlayer import HumanLayer

hl = HumanLayer(
    api_key="your-api-key",
    contact_channels={
        "default": "slack:#dev-approvals",
        "critical": "slack:#critical-approvals",
        "async": "email:team@company.com"
    }
)
```

---

## Routing Strategies

### Confidence Bands

```
Confidence > 0.95  → Auto-approve
0.70 < Confidence < 0.95  → Human review
Confidence < 0.70  → Auto-reject
```

### Two-Step Process

For high-value decisions:
1. **Proposer**: Reviews and proposes action
2. **Confirmer**: Independent verification

### Async Review Channels

For non-blocking flows:
- Slack channels for team review
- Email for stakeholder approval
- Dashboard for batch processing

---

## Implementation Patterns

### Pattern 1: Shadow Mode

Start conservatively:

```python
async def shadow_mode_action(action, context):
    """Let AI predict, compare to human decision."""

    # AI makes prediction
    ai_decision = await agent.predict(action, context)

    # Log for comparison (don't execute)
    logger.info(f"AI would have chosen: {ai_decision}")

    # Human makes actual decision
    human_decision = await request_human_decision(action, context)

    # Track agreement rate
    metrics.record_agreement(ai_decision, human_decision)

    return human_decision
```

### Pattern 2: Graduated Autonomy

```python
class GraduatedAutonomy:
    def __init__(self, agent_id: str):
        self.agent_id = agent_id
        self.trust_level = 0  # Start at zero trust

    async def execute(self, action: Action) -> Result:
        if self.trust_level < action.risk_threshold:
            # Requires human approval
            approval = await self.request_approval(action)
            if not approval:
                return Result.rejected()

        result = await self.agent.execute(action)

        # Track outcomes to adjust trust
        await self.record_outcome(action, result)

        return result

    async def record_outcome(self, action, result):
        if result.success:
            self.trust_level += 0.1
        else:
            self.trust_level -= 0.5  # Failures penalized heavily
```

### Pattern 3: Explain-Before-Execute

```python
@require_approval
def execute_with_explanation(action: Action) -> Result:
    """
    Approval request includes:
    - Clear signal: "vendor mismatch" or "amount outlier"
    - Impact summary
    - Rollback plan
    - Confidence score
    """
    return execute(action)
```

---

## Framework Comparison

### LangGraph

**Best for**: Structured workflows with full control

```python
from langgraph import Graph, interrupt

def approval_node(state):
    """Pause graph mid-execution for human input."""
    if state.requires_approval:
        return interrupt("Awaiting approval")
    return state

graph = Graph()
graph.add_node("validate", validate_node)
graph.add_node("approve", approval_node)
graph.add_node("execute", execute_node)
```

**Use when**:
- Custom routing logic needed
- Deterministic, debuggable behavior required
- Managing multiple agents/tool types

### CrewAI

**Best for**: Collaborative, role-based agent teams

```python
from crewai import Agent, Task, Crew
from crewai_tools import HumanTool

human_tool = HumanTool()

developer = Agent(
    role="Senior Developer",
    goal="Implement features correctly",
    tools=[human_tool]  # Can consult humans
)
```

**Use when**:
- Multiple agents with different goals
- Humans as decision-makers or experts
- Complex task decomposition

### HumanLayer SDK

**Best for**: Async human decisions across channels

```python
from humanlayer import require_approval

@require_approval(
    contact_channel="slack:#approvals",
    timeout=3600  # 1 hour timeout
)
async def sensitive_operation(params):
    return await execute(params)
```

**Use when**:
- Asynchronous human decisions
- Multi-channel communication
- Orchestration-agnostic implementation needed

---

## HITL Best Practices

### 1. Start Conservative

Begin with more human oversight, reduce as trust builds:

```
Week 1-2: All actions require approval
Week 3-4: Low-risk actions auto-approved
Week 5+: Graduated autonomy based on track record
```

### 2. Clear UI/UX

Design approval interfaces to explain, not obscure:

```
┌─────────────────────────────────────────────┐
│ 🔔 Approval Required                        │
├─────────────────────────────────────────────┤
│ Action: Delete user account                 │
│ User: john@example.com                      │
│                                             │
│ Signals:                                    │
│ ⚠️ Account has active subscription          │
│ ⚠️ User logged in within last 24 hours      │
│ ✓ No pending orders                         │
│                                             │
│ Confidence: 72%                             │
│ Suggested: REVIEW                           │
│                                             │
│ [Approve] [Reject] [Request More Info]      │
└─────────────────────────────────────────────┘
```

### 3. Audit Trail

Log all decisions for compliance and learning:

```python
@dataclass
class ApprovalRecord:
    action_id: str
    action_type: str
    requester: str  # Agent ID
    approver: str   # Human ID
    decision: str   # approved/rejected
    reason: str
    timestamp: datetime
    context: dict
```

### 4. Timeout Handling

Define behavior when approval times out:

```python
@require_approval(
    timeout=3600,
    on_timeout="reject",  # or "escalate", "auto_approve"
    escalate_to="slack:#critical-approvals"
)
```

---

## Integration with Remote Agentic Coding System

### Current State

The system has basic approval via platform interaction but lacks:
- Structured approval workflows
- Confidence-based routing
- Async approval channels
- Graduated autonomy

### Recommended Enhancements

1. **Add approval gates** for destructive operations:
   - `/clone` with force flag
   - `/reset` session clearing
   - Production deployments

2. **Implement confidence scoring**:
   ```typescript
   interface ActionRequest {
     action: string;
     confidence: number;
     context: Record<string, unknown>;
     requiresApproval: boolean;
   }
   ```

3. **Add approval channels**:
   - Slack: Real-time team approvals
   - GitHub: PR-based code approvals
   - Telegram: Direct user approval

4. **Create audit logging**:
   ```sql
   CREATE TABLE remote_agent_audit_log (
     id UUID PRIMARY KEY,
     session_id UUID REFERENCES sessions(id),
     action_type VARCHAR(50),
     action_data JSONB,
     approval_status VARCHAR(20),
     approved_by VARCHAR(100),
     timestamp TIMESTAMPTZ DEFAULT NOW()
   );
   ```

---

## Key Takeaways

1. **HITL is permanent**: Not a training wheel—a fundamental safety pattern
2. **Graduated autonomy**: Start conservative, increase trust with track record
3. **Multiple channels**: Slack, email, dashboard for different urgency levels
4. **Clear signals**: Explain why approval needed, not just that it's needed
5. **Audit everything**: Compliance, learning, and accountability
6. **Handle timeouts**: Define behavior when humans don't respond
