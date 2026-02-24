# Deployment Reference

## Dependencies

Add to `requirements.txt` or `pyproject.toml`:
```
agentstack-sdk>=0.6.3
a2a-sdk>=0.3.21
```

With `uv`:
```bash
uv add agentstack-sdk a2a-sdk
```

---

## Entry Point

```toml
# pyproject.toml
[project.scripts]
server = "agent:serve"  # module:function
```

---

## Port Configuration

Agents and the Agent Stack platform run on **different ports**:

- **Agent Stack Platform**: port `8333`
- **Your agent**: port `10000` (default) — change if in use

```bash
# Check if port is taken
lsof -i :10000

# Run on a different port
PORT=10001 python agent.py
```

Common environment variables:
- `HOST` — bind address (default: `127.0.0.1`)
- `PORT` — agent port (default: `10000`)
- `PLATFORM_URL` — platform endpoint (default: `http://127.0.0.1:8333`)
- `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` — API keys
- `CHAT_MODEL` — model name (e.g. `gpt-4o`, `claude-sonnet-4-5`)
- `TEMPERATURE`, `MAX_TOKENS`

Always load config from environment, never hardcode:
```python
MODEL = os.getenv("CHAT_MODEL", "gpt-4o")
API_KEY = os.getenv("OPENAI_API_KEY")
```

---

## Testing Locally

```bash
# Start your agent
python agent.py

# Send a test message
curl -X POST http://127.0.0.1:10000/a2a/messages/send \
  -H "Content-Type: application/json" \
  -d '{
    "context_id": "test-context",
    "task_id": "test-task",
    "message": {
      "role": "user",
      "parts": [{"text": "Hello, agent!"}]
    }
  }'

# Verify agent card
curl http://127.0.0.1:10000/.well-known/a2a/agent-card
```

---

## Dockerfile

```dockerfile
FROM python:3.13-slim
ARG RELEASE_VERSION="main"

COPY . /app/agent
WORKDIR /app/agent

RUN --mount=type=cache,target=/tmp/.cache/uv \
    --mount=type=bind,from=ghcr.io/astral-sh/uv:0.9.5,source=/uv,target=/bin/uv \
    UV_COMPILE_BYTECODE=1 HOME=/tmp uv sync --no-dev

ENV PRODUCTION_MODE=True RELEASE_VERSION=${RELEASE_VERSION}

CMD ["/app/agent/.venv/bin/server"]
```

---

## Deployment Checklist

- [ ] Agent runs locally without errors
- [ ] Agent responds to test messages
- [ ] Config loaded from environment variables (not hardcoded)
- [ ] `serve()` function defined
- [ ] Entry point added to `pyproject.toml`
- [ ] Dockerfile created
- [ ] Dependencies listed in `requirements.txt` or `pyproject.toml`
- [ ] Agent card endpoint returns valid JSON
