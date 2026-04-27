---
title: "AI Engineering: A Comprehensive Study Guide"
date: 2026-04-27
draft: false
tags: ["AI Engineering", "LLM", "System Design", "MLOps", "Agents"]
summary: "A high-level reference for AI Engineering, covering RAG, Agentic Systems, MCP, Evaluation Harnesses, and Production Prompt Engineering."
---

This guide provides a structured reference for AI Engineering, focusing on foundations, architecture patterns, and production-grade systems. It is intended for engineers working at a staff level to design and maintain robust AI-powered applications.

---

## 1. Foundations: LLM, Agent & Agentic Systems

### 1.1 Large Language Model (LLM)
A large language model is a neural network (transformer-based) trained on massive text corpora via next-token prediction. It maps an input context (prompt) to a probability distribution over the vocabulary and samples from it to produce outputs.

| Dimension | Description |
| :--- | :--- |
| **Core mechanism** | Self-attention + feed-forward layers; scales with parameters, data, and compute (Chinchilla scaling laws) |
| **Input/Output** | Text in → text out (or token sequences in multimodal variants) |
| **Statefulness** | Stateless per inference; the context window is its only 'memory' |
| **Capabilities** | Instruction following, reasoning, code generation, summarisation, and classification — all from a single model via prompting |
| **Limitations** | Knowledge cutoff, hallucination, no persistent state, no external actions, and context window limits |
| **Key params** | Temperature (randomness), Top-p / Top-k sampling, max_tokens, and system prompt |

### 1.2 Agent
An agent is an LLM that has been given **tools** (functions it can call) and a **decision loop** so it can take actions to achieve a goal, rather than simply answering a single query.

*   **Minimal definition:** LLM + Tool use + Observe–Think–Act loop.
*   **Key additions over LLM:** Tool calling (APIs, code execution, search), structured output parsing, and memory (in-context or external).
*   **Control flow:** ReAct (Reasoning + Acting), Chain-of-Thought (CoT), and Reflection loops.
*   **Stopping condition:** Task completion, max steps exceeded, or human escalation.
*   **Failure modes:** Tool hallucination, infinite loops, reward hacking, and error propagation.

### 1.3 Agentic System
An agentic system is a multi-component architecture where one or more agents collaborate with orchestrators, memory stores, tools, and (often) humans to complete complex, multi-step tasks in a production environment.

*   **vs. Single agent:** Multiple specialised agents; explicit orchestration layer; shared/persistent memory; and human-in-the-loop (HITL) checkpoints.
*   **Key components:** Orchestrator, Worker agents, Memory (short-term, long-term, episodic), Tool registry, Guardrails, and Logging.
*   **Patterns:** Router → specialists, Planner → executor, Critic–Actor, and Map-reduce over agents.

### Production Comparison

| Dimension | LLM | Agent | Agentic System |
| :--- | :--- | :--- | :--- |
| **Scope** | Single inference | Single-goal task loop | Multi-step, multi-agent workflow |
| **Memory** | Context window only | In-context + optional KV store | Persistent, shared, typed memory tiers |
| **Tools** | None | Tool calling | Tool registry + policy layer |
| **State** | Stateless | Stateful within a run | Persistent cross-session state |
| **Humans** | User prompt only | Optional HITL | Explicit checkpoints & approvals |
| **Failure handling** | None | Basic retry | Circuit breakers, fallbacks, and alerting |

---

## 2. Retrieval-Augmented Generation (RAG)

RAG grounds LLM responses in retrieved external documents, reducing hallucination and enabling up-to-date knowledge without retraining.

### 2.1 Core Pipeline
1.  **Indexing:** Chunk documents → embed chunks (embedding model) → store vectors in a vector DB with metadata.
2.  **Retrieval:** Embed user query → ANN search (cosine / dot-product) → top-k chunks.
3.  **Augmentation:** Inject retrieved chunks into prompt context (before or interleaved with query).
4.  **Generation:** LLM generates answer conditioned on retrieved context + query.

### 2.2 RAG Variants & Techniques

| Technique | Description | When to use |
| :--- | :--- | :--- |
| **Naive RAG** | Single retrieval → generate | Baseline; simple Q&A |
| **Advanced RAG** | Pre/post retrieval steps (re-ranking, query expansion, HyDE) | Better precision/recall |
| **Modular RAG** | Swappable retrieval, fusion, and reader modules | Complex pipelines |
| **Multi-hop RAG** | Iterative retrieval across documents | Multi-step reasoning |
| **Graph RAG** | Knowledge graph + vector retrieval | Entity-relationship queries |
| **Agentic RAG** | Agent decides when/how to retrieve | Dynamic, adaptive tasks |

### 2.3 Key Design Decisions
*   **Chunking strategy:** Fixed-size, sentence, paragraph, semantic, or hierarchical. Overlap (10–15%) prevents context loss at boundaries.
*   **Embedding model:** Domain-specific models outperform general ones. Dimension vs. latency trade-off.
*   **Vector DB:** Pinecone, Weaviate, Qdrant, pgvector, or Chroma. Evaluate: ANN algorithm (HNSW, IVF), filtering, and scalability.
*   **Re-ranking:** Cross-encoder re-ranker (e.g., Cohere Rerank, BGE) after initial retrieval improves precision at the cost of latency.
*   **Hybrid search:** BM25 (sparse, keyword) + dense vector search. RRF (Reciprocal Rank Fusion) merges results.
*   **Context window management:** Relevance truncation, LLMLingua compression, or selective injection.

---

## 3. Model Context Protocol (MCP)

MCP (Anthropic, 2024) is an open protocol that standardises how LLM applications connect to external tools, data sources, and services—analogous to USB-C for AI integrations.

### 3.1 Architecture
*   **MCP Host:** The application that embeds/runs the LLM (e.g., Claude Desktop, VS Code extension, custom app).
*   **MCP Client:** Protocol client inside the host; maintains a 1:1 connection with each server.
*   **MCP Server:** Lightweight process that exposes capabilities via the protocol; can be local (stdio) or remote (SSE/HTTP).
*   **Transport:** Local: stdio (process pipes). Remote: Server-Sent Events over HTTP.
*   **Protocol:** JSON-RPC 2.0 messages; supports capability negotiation on handshake.

### 3.2 Core Primitives
*   **Tools:** Executable functions the model can invoke (e.g., `run_query`, `send_email`). Model decides when to call.
*   **Resources:** Read-only data exposed to the model (files, DB records, API responses). Host controls access.
*   **Prompts:** Reusable, parameterised prompt templates surfaced to the user (e.g., slash commands).
*   **Sampling:** Server can request the host to call the LLM on its behalf (recursive/nested calls).

---

## 4. AI Harness System (Evaluation Harness)

An AI harness is the infrastructure layer that wraps an AI model or agent to enable systematic testing, benchmarking, and continuous evaluation.

### 4.1 Core Components
*   **Test dataset loader:** Standardised interface to load benchmarks, golden sets, and adversarial suites (HumanEval, MMLU, custom domain evals).
*   **Prompt runner:** Sends prompts to the model/agent under test; handles batching, retries, and rate limits.
*   **Output parser:** Extracts structured predictions from free-form model output (regex, LLM-as-judge, schema validation).
*   **Scorer / Metrics:** Computes task-specific metrics: accuracy, F1, BLEU, pass@k, faithfulness, and tool call accuracy.
*   **Comparator:** Side-by-side comparison across model versions, prompts, or configurations (A/B).
*   **Reporter:** Aggregates results; generates dashboards, regression alerts, and leaderboards.
*   **CI integration:** Runs evals on every PR/deploy; gates promotion on metric thresholds.

---

## 5. C4 Architecture Model for AI Systems

C4 provides four levels of abstraction for communicating software architecture visually. For AI systems, it maps cleanly onto the layers of an agentic stack.

| Level | What it shows | AI System example |
| :--- | :--- | :--- |
| **L1 – System Context** | System + external actors/systems | AI Assistant ↔ User, CRM, Email, Data Lake |
| **L2 – Container** | Deployable units inside the system | API Gateway, Orchestrator Service, Vector DB, LLM Proxy |
| **L3 – Component** | Major logical components inside a container | RAG pipeline, Tool router, Guardrails module, Eval harness |
| **L4 – Code** | Classes, functions (rarely drawn) | Prompt builder class, retriever function |

---

## 6. Prompt Engineering

### 6.1 Core Techniques
*   **Zero-shot:** Task described with no examples. Best for capable models and common tasks.
*   **Few-shot:** 2–8 input/output examples in the prompt to ensure format compliance and style transfer.
*   **Chain-of-Thought (CoT):** Asking the model to reason step-by-step. Best for math and multi-step reasoning.
*   **Self-consistency:** Sampling N CoT paths and taking a majority vote to improve reliability.
*   **ReAct:** Interleaving Thought, Action, and Observation. Essential for tool-using agents.
*   **Least-to-most:** Breaking a task into subtasks and solving them in order. Best for compositional problems.

### 6.2 System Prompt Engineering for Production
Designing robust system prompts requires clear structure and defensive engineering:
*   **Role + Goal:** Explicitly assign a role and state the objective to anchor model behaviour.
*   **Constraints first:** State what the model must **NOT** do before what it should do.
*   **Output format:** Specify JSON schema, XML tags, or markdown structure for parseable outputs.
*   **Few-shot in system prompt:** Include canonical examples to calibrate style and format.
*   **Injected context slot:** Reserve a clear section for dynamic context (RAG chunks, user state).
*   **Versioning:** Treat prompts as code. Tag releases, track changes, and run evals on every diff.

### 6.3 Prompt Failure Modes and Mitigations
Production systems must be resilient to common prompt vulnerabilities:

*   **Prompt Injection:** User input overrides the system prompt. 
    *   *Mitigation:* Use strict delimiters to wrap user input, such as `<user_input>{{input}}</user_input>`. Instruct the model to ignore any instructions found within those tags. Use input sanitisation and output validation.
*   **Prompt Leakage:** The model reveals the system prompt to the user.
    *   *Mitigation:* Include explicit "do not reveal" instructions and implement output filtering to detect snippets of the system prompt.
*   **Specification Gaming:** The model follows the literal letter of the prompt but not the intended spirit.
    *   *Mitigation:* Test adversarially and use LLM-as-judge to evaluate if the outcome meets the qualitative goal.
*   **Sycophancy:** The model agrees with incorrect user assertions to be "helpful."
    *   *Mitigation:* Instruct the model to be critical and objective; use calibration evals to test its willingness to correct the user.

---

## 7. Memory in AI Systems

| Memory Type | Storage | Scope | Example use |
| :--- | :--- | :--- | :--- |
| **In-context** | Context window tokens | Single inference | Conversation history, RAG chunks |
| **External short-term** | Cache / Redis / KV store | Session / multi-turn | Chat history buffer, session state |
| **External long-term** | Vector DB / Relational DB | Cross-session / persistent | User preferences, knowledge base |
| **Episodic** | Structured event log | Task/run history | Agent execution traces for reflection |
| **Semantic** | Model weights | Permanent (until retrain) | World knowledge baked in during training |

---

## 8. Agentic System Design Patterns

| Pattern | Structure | Trade-offs |
| :--- | :--- | :--- |
| **Orchestrator–Worker** | Central orchestrator decomposes task; dispatches to worker agents | + Clear separation of concerns; - Single point of failure |
| **Planner–Executor** | Planner generates a plan (DAG of steps); executor runs each step | + Debuggable plan before execution; - Plan may be stale mid-execution |
| **Critic–Actor** | Actor proposes; Critic evaluates; loop until pass or max iterations | + Self-correction; - Latency multiplier, risk of echo chamber |
| **Map-Reduce** | Map: fan out sub-tasks to parallel agents; Reduce: aggregate results | + Parallelism; - Consistency of aggregation |
| **Router** | Classifier routes queries to best-fit agent or pipeline | + Cost efficiency (cheap routing); - Router becomes a bottleneck |

---

## 9. Whiteboard Design Cheat Sheet

When designing an AI system, use this structure to ensure all production concerns are addressed:

1.  **Clarify requirements:** What's the task? Latency budget? Accuracy bar? Data sensitivity? Scale? Feedback loop?
2.  **Draw C4 L1 (Context):** Identify actors, your system box, and external dependencies.
3.  **Draw C4 L2 (Containers):** API gateway, orchestrator, LLM proxy/router, vector DB, memory store, tool servers, eval harness, and observability.
4.  **Walk through a request trace:** Query → retrieval → prompt construction → LLM call → tool use → response → logging.
5.  **Discuss trade-offs:** RAG vs fine-tuning, latency vs cost, determinism vs creativity, single agent vs multi-agent.
6.  **Address failure modes:** LLM errors, tool timeouts, hallucination, prompt injection, and data staleness.
7.  **Describe evaluation:** Offline evals (harness), online monitoring, and human review queue.

---

## 10. Quick-Reference Glossary

*   **ANN (Approximate Nearest Neighbour):** Fast vector similarity search (HNSW, IVF).
*   **HNSW (Hierarchical Navigable Small World):** Graph-based ANN index; O(log n) search.
*   **BM25:** Probabilistic keyword ranking algorithm; strong sparse retrieval baseline.
*   **RRF (Reciprocal Rank Fusion):** Merges multiple ranked lists for hybrid search.
*   **HyDE (Hypothetical Document Embeddings):** Generate a fake answer, then embed it for retrieval.
*   **CoT (Chain-of-Thought):** Prompt the model to show step-by-step reasoning.
*   **ReAct (Reasoning + Acting):** Interleave thought traces with tool calls in an agent loop.
*   **HITL (Human-in-the-Loop):** Explicit human review/approval step in an automated pipeline.
*   **SFT (Supervised Fine-Tuning):** Train LLM on labelled (prompt, completion) pairs.
*   **DPO (Direct Preference Optimisation):** Alignment via preference pairs, no reward model.
*   **LoRA (Low-Rank Adaptation):** Parameter-efficient fine-tuning via low-rank weight matrices.
*   **PPO (Proximal Policy Optimisation):** RL algorithm used in RLHF training.
*   **Grounding:** Tying model outputs to verifiable sources (RAG, citations, structured DB lookup).
*   **Guardrails:** Input/output filters that enforce safety, format, and policy constraints.
*   **Context window:** Maximum tokens an LLM can process in one call (input + output).
*   **Hallucination:** Model generates plausible but factually incorrect or fabricated content.
*   **Temperature:** Sampling parameter (0=greedy, >1=random). Lower = more deterministic.
*   **Pass@k:** Probability that at least one of k generated code samples passes all tests.
*   **RAGAS (RAG Assessment):** Automated eval framework for RAG pipelines (context precision/recall, faithfulness).
*   **LLM-as-Judge:** Using a strong LLM to score/compare outputs of another model.
*   **Tool calling:** Structured mechanism for LLM to request execution of external functions.
*   **Embedding:** Dense vector representation of text; semantic similarity = cosine proximity.
*   **Vector DB:** Database optimised for storing and querying high-dimensional embeddings.
*   **Chunking:** Splitting documents into smaller pieces before embedding for RAG indexing.
*   **Reranker:** Cross-encoder model that re-scores retrieved chunks for precision (e.g., Cohere Rerank).
*   **MCP (Model Context Protocol):** Open standard for LLM ↔ tool/data integrations.
*   **C4 Model:** 4-level architecture diagram framework: Context, Container, Component, Code.
*   **ADR (Architecture Decision Record):** Document capturing a design choice and its rationale.
*   **Shadow eval:** Run new model version on prod traffic in parallel before promoting.
*   **Trajectory eval:** Evaluate the full sequence of agent actions, not just the final answer.
