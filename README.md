<details>
<summary>Baseline output (click to expand)</summary>

```text
Generic LangGraph answer (no documentation sources):

Approach 1: astream_events(version="v2")
- Automatically captures events from nested subgraphs and tools.
- Listen for on_tool_start (tool input) and on_tool_end (tool result).
- on_chat_model_stream gives live LLM tokens.

Approach 2: stream_mode="custom" with get_stream_writer()
- Inside the subagent node, call writer({...}) to push custom progress
  or intermediate tool results.
- On the parent graph, use stream_mode=["updates", "custom"].

Summary:
- Capture all tool inputs/results automatically -> astream_events (on_tool_start, on_tool_end)
- Custom progress from inside a subagent -> stream_mode="custom"
- Token-by-token output -> stream_mode="messages" or astream_events

No source links were provided.
```

</details>

<details>
<summary>RAG output (click to expand)</summary>

```text
To stream intermediate tool results from a subagent, enable `subgraphs=True`
when calling `.stream()` on the agent.

- With `subgraphs=True`, stream modes such as "updates" or "messages" yield
  (namespace, data) tuples.
- Subagent namespaces start with "tools:" (e.g. ("tools:<id>",)).
- Check the namespace to tell subagent events from coordinator events.

Example:

for namespace, data in agent.stream(
    {"messages": [{"role": "user", "content": "Research quantum computing"}]},
    stream_mode="updates",
    subgraphs=True,
):
    is_subagent = any(s.startswith("tools:") for s in namespace)
    print("[Subagent]" if is_subagent else "[Coordinator]", data)

For frontends, use the `useStream` hook (stream.subagents) to render subagent
work and tool calls alongside coordinator messages.

Sources:
- https://docs.cloud.langchain.com/oss/python/deepagents/streaming
- https://docs.cloud.langchain.com/oss/python/deepagents/frontend/subagent-streaming
```

</details>

