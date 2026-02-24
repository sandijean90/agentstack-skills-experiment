# A2A Protocol Reference

## Why A2A Matters

When you write an agent naturally, you write:
```python
def my_agent(user_input: str) -> str:
    result = do_work(user_input)
    return result
```

Agent Stack needs agents to speak A2A protocol:
```python
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)      # Extract from A2A Message
    result = do_work(user_text)              # Your logic (unchanged)
    yield AgentMessage(text=result)          # Wrap as A2A Message
```

---

## Core Concepts

### Message (Input)

The `Message` type is how user input arrives — a structured container, not a plain string.

```python
from a2a.types import Message
from a2a.utils.message import get_message_text

# Message structure:
# { "role": "user", "parts": [{"text": "Hello, agent!"}] }

user_text = get_message_text(input)  # Returns: "Hello, agent!"
```

### AgentMessage (Output)

Wrap your response text before yielding.

```python
from agentstack_sdk.a2a.types import AgentMessage

yield AgentMessage(text="Here is my response")
# Or just yield plain text — SDK auto-converts
yield "Here is my response"
```

### RunContext (Conversation State)

Manages conversation history across multiple turns.

```python
from agentstack_sdk.server.context import RunContext

async def my_agent(input: Message, context: RunContext):
    history = []
    async for msg in context.load_history():
        if isinstance(msg, Message):
            history.append(get_message_text(msg))

    response = my_agent_logic(get_message_text(input), history)
    await context.store(input)
    yield response
```

### Async and Yield (Streaming Pattern)

A2A agents use `async def` and `yield` instead of `return`. This allows incremental streaming:

```python
async def my_agent(input: Message, context: RunContext):
    yield "Thinking..."
    result = expensive_computation()
    yield "Processing..."
    yield format_result(result)
```

---

## The Transformation Pattern

**Before:**
```python
def my_agent(user_input: str, history: list) -> str:
    response = do_work(user_input, history)
    return response
```

**After:**
```python
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)          # A2A layer

    history = []                                  # A2A layer
    async for msg in context.load_history():
        if isinstance(msg, Message):
            history.append(get_message_text(msg))

    response = do_work(user_text, history)        # Your code (unchanged)

    await context.store(input)                    # A2A layer
    yield response                                # A2A layer
```

Steps 1, 2, and 4 are the A2A wrapper. Step 3 is your untouched agent logic.
