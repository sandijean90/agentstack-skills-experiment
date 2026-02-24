---
name: agentstack
description: Deploy agents to Agent Stack. Use when wrapping LangGraph, CrewAI, or custom Python agents with A2A protocol, creating agentstack-sdk wrappers, writing Server() and @server.agent() code, setting up pyproject.toml entry points, or building Dockerfiles for Agent Stack deployment.
metadata:
  version: "1.0"
compatibility: Requires Python 3.11+, agentstack-sdk, a2a-sdk
---

# Agent Stack Deployment Skill

## Key Principle

**Your agent code doesn't change. Only the deployment wrapper changes.**

You'll add two layers:
1. **A2A Protocol** — the message format agents use to communicate
2. **Agent Stack SDK** — the runtime platform wrapper

> **Do NOT modify:** your graph/crew/logic functions, existing class definitions, framework configuration.

---

## Reference Files

Read these on demand — do not load all of them upfront:

- If the user's agent uses LangGraph, CrewAI, or a custom class, read [Framework Patterns](references/FRAMEWORK_PATTERNS.md) before writing any code.
- If you need detail on Message, RunContext, or async/yield patterns, read [A2A Protocol](references/A2A_PROTOCOL.md).
- If the user is testing locally, configuring ports, or building a Dockerfile, read [Deployment](references/DEPLOYMENT.md).
- If the user asks about model selection or logging reasoning steps, read [Extensions](references/EXTENSIONS.md).
- If something isn't working, read [Pitfalls](references/PITFALLS.md) before making changes.

---

## Minimal Adaptation Checklist

### Phase 1 — Setup
1. Install `agentstack-sdk` and `a2a-sdk`
2. Import: `Server`, `RunContext`, `Message`, `AgentMessage`, `get_message_text`
3. Create `server = Server()`

### Phase 2 — Wrap
4. Decorate your handler function with `@server.agent(name="...", description="...")`
5. Extract input with `get_message_text(input)`
6. Load history with `context.load_history()`
7. Call your existing agent logic (UNCHANGED)
8. Store input with `await context.store(input)`
9. Yield response (not return)
10. Add `serve()` function calling `server.run()`
11. Add entry point to `pyproject.toml`

### Phase 3 — Deploy
12. Test locally
13. Create Dockerfile
14. Deploy to Agent Stack

---

## Complete Minimal Example

```python
# agent.py
import os
from agentstack_sdk.server import Server
from agentstack_sdk.server.context import RunContext
from a2a.types import Message
from agentstack_sdk.a2a.types import AgentMessage
from a2a.utils.message import get_message_text

# Your existing agent logic — untouched
def my_agent_logic(user_input: str, history: list) -> str:
    return f"Processed: {user_input}"

# Agent Stack wrapper
server = Server()

@server.agent(
    name="My Agent",
    description="What your agent does"
)
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)

    history = []
    async for msg in context.load_history():
        if isinstance(msg, Message):
            history.append(get_message_text(msg))

    response = my_agent_logic(user_text, history)

    await context.store(input)
    yield AgentMessage(text=response)

def serve():
    try:
        server.run(
            host=os.getenv("HOST", "127.0.0.1"),
            port=int(os.getenv("PORT", 10000)),
            self_registration=True,
            configure_telemetry=True,
        )
    except KeyboardInterrupt:
        pass

if __name__ == "__main__":
    serve()
```

```toml
# pyproject.toml
[project.scripts]
server = "agent:serve"
```
