# LLM Evaluation & Observability for a RAG Chatbot

> Built on the [Hyperskill starter project](https://github.com/hyperskill-content/LLM-evals) as part of the **AI Engineer Intensive Bootcamp (2026)**.
> The base chatbot was provided; observability, feedback collection and the evaluation setup are my implementation.

A smartphone recommendation chatbot (LangChain + Qdrant + tool calling), instrumented end to end so that every answer is **traceable, measurable and reviewable**: from the retrieval step to the final response, including cost, latency, user feedback and LLM-as-a-judge scores.

**Why this matters:** LLM output is non-deterministic. Without traces and evaluation you cannot tell *why* an answer was wrong, *where* in the chain it went wrong, or whether a prompt change made things better or worse.

---

## What I implemented

- **Hierarchical tracing with Langfuse** – `@observe` decorators on the core functions (`embed-documents`, `generate-context`, `ai-response`) combined with the LangChain `CallbackHandler`. Every chain step (`context`, `final-response`, `goodbye-message`) appears as a named span with tokens, cost and latency.
- **Session & user tracking** – each conversation gets a session ID and user ID, so traces can be grouped and analysed per conversation and per user.
- **User feedback loop** – at the end of a session the user rates the answer (categorical score `usefulness`) and leaves a comment. Both are attached to the trace via `score_current_trace`.
- **Human annotation** – annotation queues in the Langfuse UI for expert review of individual traces.
- **LLM-as-a-judge** – automated evaluators (*Relevance*, *Answer Relevance*) running on the LLM generations in Langfuse.
- **Local infrastructure** – `docker-compose.yml` for Qdrant; Langfuse self-hosted via its official compose setup.

## Architecture

```
User (console)
   │
   ▼
context chain ──► LLM with tools ──► SmartphoneInfo tool ──► Qdrant (semantic search)
   │
   ▼
final-response chain ──► LLM ──► answer
   │
   ▼
feedback (usefulness + comment)

All steps ──► Langfuse: traces · tokens · cost · latency · scores · annotations · LLM-as-a-judge
```

## Results

| Trace with nested spans, token usage and cost | LLM-as-a-judge scores per generation |
|---|---|
| ![Trace](learnings/task2.png) | ![Evaluation](learnings/task5.png) |

| User feedback score | Human annotation |
|---|---|
| ![Feedback](learnings/task3.png) | ![Annotation](learnings/task4.png) |

**Key takeaways** (details in [learnings/learnings.md](learnings/learnings.md)):
- Decorator-based tracing gives a quick overview; passing the Langfuse config into each chain invocation adds the depth that matters (tool calls, tokens, per-step cost).
- Evaluation works best in layers: **manual annotation** with domain experts early on, **LLM-as-a-judge** once traffic scales, and **direct user feedback** throughout.
- Traces can be exported and analysed by script – a basis for standardised checks in a CI/CD pipeline.

## Tech stack

Python · LangChain · OpenAI API · Qdrant · Langfuse (SDK v3, self-hosted) · Docker

## Run it

```bash
# 1. Start Qdrant
docker compose up -d

# 2. Start Langfuse (self-hosted) – see https://langfuse.com/self-hosting
git clone https://github.com/langfuse/langfuse.git && cd langfuse && docker compose up -d && cd ..

# 3. Configure and run
cp .env.sample .env      # add OpenAI and Langfuse keys
pip install -r requirements.txt
python main.py
```

---

<details>
<summary>Original task description (Hyperskill)</summary>

The project is structured into five tasks, see the [tasks](./tasks) folder:

1. **Introduction and Setup** – Langfuse, Qdrant and API keys
2. **Tracing and Monitoring** – collect logs and metrics via Langfuse
3. **Collecting User Feedback**
4. **Annotation** – annotate traces in the Langfuse UI
5. **Model-based evaluation** – LLM-as-a-judge

</details>
