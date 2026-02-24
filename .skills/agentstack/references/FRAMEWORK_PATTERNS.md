# Framework-Specific Patterns

## LangGraph

Your LangGraph workflow, nodes, edges, and state are completely untouched. Only wrap the entry point.

```python
from langgraph.graph import StateGraph

# Your existing LangGraph definition — untouched
workflow = StateGraph(AgentState)
workflow.add_node("parse", parse_node)
workflow.add_node("process", process_node)
app = workflow.compile()

def run_workflow(user_input: str) -> dict:
    return app.invoke({"input": user_input})

# Agent Stack wrapper
server = Server()

@server.agent(name="LangGraph Agent", description="What your agent does")
async def langgraph_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)

    history = []
    async for msg in context.load_history():
        if isinstance(msg, Message):
            history.append(get_message_text(msg))

    result = run_workflow(user_text)  # Your existing logic — unchanged

    await context.store(input)
    yield result["output"]
```

---

## CrewAI

Your Crew definition (agents, tasks, kickoff) is completely untouched.

```python
from crewai import Crew, Agent, Task

# Your existing CrewAI definition — untouched
researcher = Agent(role="Researcher", goal="...", backstory="...")
analyst = Agent(role="Analyst", goal="...", backstory="...")
crew = Crew(agents=[researcher, analyst], tasks=[...])

def run_research(query: str) -> str:
    return str(crew.kickoff(inputs={"query": query}))

# Agent Stack wrapper
server = Server()

@server.agent(name="CrewAI Research Agent", description="Multi-agent research crew")
async def crew_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    result = run_research(user_text)  # CrewAI is sync — fine in async context

    await context.store(input)
    yield result
```

---

## Custom Python / RAG

Your RAG logic (vector search, retrieval, generation) is completely untouched.

```python
# Your existing RAG logic — untouched
def rag_agent(query: str, history: list) -> str:
    docs = vector_search(query)
    context = "\n".join([d.content for d in docs])
    return llm.generate(f"Context: {context}\n\nHistory: {history}\n\nQuestion: {query}")

# Agent Stack wrapper
server = Server()

@server.agent(name="RAG Q&A Agent", description="Answers questions using RAG")
async def my_rag_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)

    history = []
    async for msg in context.load_history():
        if isinstance(msg, Message):
            history.append(get_message_text(msg))

    response = rag_agent(user_text, history)  # Your existing logic — unchanged

    await context.store(input)
    yield response
```

---

## Streaming / Async Generator

```python
# Your existing streaming agent
async def my_streaming_agent(user_input: str):
    for chunk in generate_response(user_input):
        yield chunk

# Agent Stack wrapper
@server.agent(name="Streaming Agent", description="Streams responses")
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    await context.store(input)

    async for chunk in my_streaming_agent(user_text):
        yield chunk
```

---

## Class-Based Agents

```python
# Your existing agent class — untouched
class MyAgentClass:
    def __init__(self, config):
        self.config = config

    def run(self, input: str) -> str:
        return response

agent_instance = MyAgentClass(config={...})

# Agent Stack wrapper
@server.agent(name="My Class Agent")
async def my_agent(input: Message, context: RunContext):
    user_text = get_message_text(input)
    response = agent_instance.run(user_text)

    await context.store(input)
    yield AgentMessage(text=response)
```
