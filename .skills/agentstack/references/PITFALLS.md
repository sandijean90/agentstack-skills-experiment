# Common Pitfalls

## Using plain strings instead of A2A types

```python
# ❌ Wrong
async def my_agent(user_input: str):
    return "response"

# ✅ Right
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    yield "response"
```

---

## Passing Message object directly to your logic

```python
# ❌ Wrong — input is a Message object, not a string
response = my_logic(input)

# ✅ Right
user_text = get_message_text(input)
response = my_logic(user_text)
```

---

## Using return instead of yield

```python
# ❌ Wrong — blocks streaming
async def my_agent(input: Message, context: RunContext):
    return "response"

# ✅ Right
async def my_agent(input: Message, context: RunContext):
    yield "response"
```

---

## Hardcoding configuration

```python
# ❌ Wrong
OPENAI_API_KEY = "sk-..."
MODEL = "gpt-4o"

# ✅ Right
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
MODEL = os.getenv("CHAT_MODEL", "gpt-4o")
```

---

## Ignoring conversation history

```python
# ❌ Wrong — agent has no memory between turns
async def my_agent(input: Message, context: RunContext):
    response = my_logic(get_message_text(input))
    yield response

# ✅ Right
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    history = [get_message_text(m) async for m in context.load_history()
               if isinstance(m, Message)]
    response = my_logic(user_text, history)
    await context.store(input)
    yield response
```

---

## Rewriting your agent logic

```python
# ❌ Wrong — defeats the purpose
@server.agent()
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    # ...200 lines of rewritten logic using SDK primitives...

# ✅ Right — call your existing function
@server.agent()
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    response = my_existing_agent_function(user_text)
    yield response
```
