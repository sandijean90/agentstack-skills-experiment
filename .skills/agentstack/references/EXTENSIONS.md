# Extensions Reference

Extensions provide optional capabilities. Only use them if needed — basic wrapping is sufficient for most agents.

---

## LLM Extension

Use when the platform needs to control model/provider selection.

```python
from typing import Annotated
from agentstack_sdk.a2a.extensions import (
    LLMServiceExtensionServer,
    LLMServiceExtensionSpec,
)

@server.agent()
async def agent_with_llm(
    message: Message,
    context: RunContext,
    llm: Annotated[
        LLMServiceExtensionServer,
        LLMServiceExtensionSpec.single_demand()
    ],
):
    model_config = llm.data.llm_fulfillments["default"]
    api_key = model_config.api_key
    model = model_config.model

    response = your_agent(get_message_text(message), model=model, api_key=api_key)
    yield response
```

---

## Trajectory Extension

Use when you want to log agent reasoning steps to the platform.

```python
from typing import Annotated
from agentstack_sdk.a2a.extensions import (
    TrajectoryExtensionServer,
    TrajectoryExtensionSpec,
)

@server.agent()
async def agent_with_trajectory(
    message: Message,
    context: RunContext,
    trajectory: Annotated[TrajectoryExtensionServer, TrajectoryExtensionSpec()],
):
    yield trajectory.trajectory_metadata(
        title="Planning",
        content="Agent is breaking down the task..."
    )

    response = your_agent(get_message_text(message))

    yield trajectory.trajectory_metadata(
        title="Execution",
        content="Agent is running tools..."
    )

    yield response
```

---

## When to Use Extensions

| Extension   | Use when...                                      |
|-------------|--------------------------------------------------|
| LLM         | Platform needs to control model selection        |
| Trajectory  | You want to log reasoning steps                  |
| Form        | You need structured form input from users        |
| Citation    | You're integrating source citations into output  |
