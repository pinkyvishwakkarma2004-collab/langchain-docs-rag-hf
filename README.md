# LangChain Docs Q&A Agent (RAG + HuggingFace Embeddings)

A Retrieval-Augmented Generation (RAG) agent that answers questions about LangChain / Deep Agents documentation, with source citations.

Based on the official LangChain Deep Agents RAG tutorial. My changes: replaced OpenAI embeddings with free HuggingFace embeddings, switched the chat model to Gemini, and tuned the agent for free-tier API limits.

## How it works
1. **Index:** fetch 14 LangChain doc pages, split into 926 chunks, embed with `sentence-transformers/all-MiniLM-L6-v2`, store in an in-memory vector store.
2. **Search tool:** retrieves the top matching chunks and saves them as files in the agent filesystem.
3. **Subagent (`chunk-analyst`):** reads each chunk file and returns a short summary.
4. **Main agent:** combines the summaries into a final answer with source links.

## Tech stack
Python, LangChain, Deep Agents, HuggingFace sentence-transformers, Google Gemini (`gemini-3.5-flash-lite`)

## Setup
1. Open the notebook in Google Colab.
2. Get a free Google API key from https://aistudio.google.com/apikey
3. Set it in the session (never commit your key):
   `os.environ["GOOGLE_API_KEY"] = "<your key>"`
4. Install dependencies: `pip install -r requirements.txt`
5. Run the cells in order: install, key, index, tool, instructions, agent, question.

See the notebook for the full run outputs.

## Example
**Question:** How do I stream intermediate tool results from a subagent?

<details>
<summary>Baseline output (no retrieval) - click to expand</summary>

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
- Capture all tool inputs/results automatically -> astream_events
- Custom progress from inside a subagent -> stream_mode="custom"
- Token-by-token output -> stream_mode="messages" or astream_events

No source links were provided.
```

</details>

<details>
<summary>RAG agent output (with sources) - click to expand</summary>

```text
To stream intermediate tool results from a subagent, enable `subgraphs=True`
when calling `.stream()` on the agent.

- With subgraphs=True, stream modes such as "updates" or "messages" yield
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

## Challenges and fixes
- **OpenAI quota error:** moved to free local HuggingFace embeddings.
- **Google embedding rate limit (429):** same fix, local embeddings.
- **Deprecated model (404):** updated to `gemini-3.5-flash-lite`.
- **Free-tier limits on agent calls:** reduced retrieved chunks to 2 and parallel subagent tasks to 1.

## Limitations
- In-memory vector store (rebuilt every session).
- Only 14 documentation pages indexed.
- Free-tier API limits make runs slow.
