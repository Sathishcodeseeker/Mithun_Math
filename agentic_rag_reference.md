# Agentic RAG — Complete Reference

## Fundamentals Taxonomy

| Layer | Core Concepts |
|-------|--------------|
| Query | Rewriting, decomposition, HyDE, routing |
| Retrieval | Vector, sparse, hybrid, graph, adaptive, multi-hop |
| Ranking | Re-ranking, filtering, relevance grading |
| Context | Chunking, compression, assembly, token budgeting |
| Generation | Grounded synthesis, citation, faithfulness |
| Agent Loop | Planning, tool-use, ReAct, state, memory |
| Evaluation | Self-critique, CRAG, hallucination detection |
| Multi-Agent | Specialization, delegation, critic agents |
| Infra | Embeddings, indexes, caching, ingestion pipelines |
| Ops | Tracing, cost, latency, security, permissions |
| Human | Disambiguation, approval gates, feedback loops |

---

## 1. Retrieval Primitives

- **Vector search** — embeddings, similarity metrics, ANN indexes
- **Sparse retrieval** — BM25, keyword matching
- **Hybrid retrieval** — combining dense + sparse with reciprocal rank fusion
- **Chunking strategies** — fixed-size, semantic, recursive, document-aware

## 2. Generation Primitives

- **LLM inference** — prompt to completion
- **Grounded generation** — conditioning output on retrieved context
- **Citation / attribution** — tracing claims back to source chunks

## 3. Agent Fundamentals (what makes it "Agentic")

- **Planning / decomposition** — breaking a query into sub-tasks
- **Tool use** — the LLM decides when and which retriever/tool to invoke
- **Reasoning loop (ReAct)** — observe, think, act, observe, repeated until done
- **State / memory** — maintaining context across multiple retrieval steps
- **Self-reflection / critique** — evaluating whether retrieved context is sufficient or the answer is correct, then re-routing

## 4. Orchestration & Control Flow

- **Query routing** — classifying intent to pick the right retriever or data source
- **Multi-step retrieval** — iterative refinement (retrieve, judge, re-query)
- **Conditional branching** — "if context insufficient, try different source"
- **Fallback / escalation** — graceful degradation when retrieval fails

## 5. Knowledge Infrastructure

- **Vector stores** — Pinecone, Weaviate, pgvector, FAISS
- **Document ingestion pipelines** — parsing, cleaning, embedding, indexing
- **Metadata filtering** — pre-filtering by date, source, permission
- **Re-ranking** — cross-encoder reranking after initial retrieval

## 6. Evaluation & Guardrails

- **Relevance scoring** — is the retrieved chunk actually useful?
- **Hallucination detection** — is the answer grounded in the context?
- **Answer faithfulness** — does the response contradict the sources?
- **Guardrails** — scope control, PII filtering, safety checks

---

## Query Layer Deep Dive

### Query Rewriting

LLM reformulates the query into something more retrievable.

```
User: "Why is my app slow?"
Rewritten: "common causes of application latency in Spring Boot services including database connection pooling, N+1 queries, and garbage collection pauses"
```

Techniques:
- LLM-based rewriting
- Multi-query generation (3-5 variants, retrieve for all, merge results)
- Step-back prompting ("what's the broader concept behind this question?")

### Query Decomposition

Breaks complex questions into atomic sub-questions, retrieves for each independently, then synthesizes.

```
Original: "Compare auth in user-service vs payment-service"

Decomposed:
  Q1: "What authentication mechanism does user-service use?"
  Q2: "What authentication mechanism does payment-service use?"
  Q3: (no retrieval needed — LLM reasons over Q1 + Q2 results)
```

### HyDE (Hypothetical Document Embeddings)

1. Ask the LLM to generate a hypothetical answer (hallucinates one)
2. Embed that fake answer
3. Use that embedding to search the index

Bridges the query-document semantic gap. The fake answer doesn't need to be correct — it just needs to sound like the real answer so the embedding is in the right neighborhood.

Tradeoff: Extra LLM call per query (latency + cost). Only worth it when query-document mismatch is a real problem.

### Query Routing

Agent classifies the query intent and sends it to the right retriever.

```
"What's our API rate limit?"       → Vector DB (documentation)
"How many users signed up today?"  → SQL / analytics DB
"Latest CVE for Log4j?"           → Web search
"Where is AuthFilter defined?"     → Code search (grep/AST)
```

Implementation options:
- LLM-based classifier (system prompt with descriptions of each source)
- Function calling / tool selection (each retriever is a tool, LLM picks)
- Fine-tuned small model for classification
- Rule-based / keyword heuristics

---

## Advanced Retrieval Patterns

- **Graph RAG** — knowledge graphs + entity relationships as retrieval substrate
- **Adaptive retrieval** — deciding whether to retrieve at all
- **Self-RAG** — the model generates retrieval tokens, critique tokens, decides mid-generation to fetch more
- **Corrective RAG (CRAG)** — retrieve, grade relevance, if low web search fallback, refine
- **Speculative RAG** — multiple drafts from different chunks, select best
- **RAFT** — fine-tuning the LLM to better ignore distractor documents

---

## Embedding Layer

- **Model selection** — general vs. domain-fine-tuned embeddings
- **Dimensionality tradeoffs** — accuracy vs. latency vs. storage
- **Late interaction models** — ColBERT (token-level matching instead of single-vector)
- **Matryoshka embeddings** — truncatable dimensions for flexible precision

---

## Chunking as a First-Class Problem

- **Parent-child retrieval** — retrieve small chunk, pass surrounding parent for context
- **Agentic chunking** — LLM decides boundaries based on semantics
- **Proposition-level indexing** — decompose docs into atomic factual claims, embed each

---

## Context Management

- **Context compression/summarization** — fitting more into the window
- **Lost-in-the-middle mitigation** — positioning relevant chunks where the LLM attends best
- **Token budget allocation** — how much window for retrieval vs. reasoning

---

## LangGraph Mental Model

### Core Pattern

1. **Nodes** — each has its own agent.md (system prompt + tools)
2. **Deterministic edges** — A always goes to B
3. **Conditional edges** — read state, pick next node

```
Node (LLM decides) → state updated → Edge (reads state, routes deterministically)
```

### How LLM Chooses Correctly at Conditional Edges

1. Router node's agent.md constrains the LLM with clear category descriptions + few-shot examples
2. LLM responds, state gets updated
3. Conditional edge reads state deterministically

Use **structured output** (Pydantic models with Literal types) to eliminate format errors:

```python
class RouteDecision(BaseModel):
    route: Literal["sql_agent", "vector_search", "web_search"]

llm_with_structure = llm.with_structured_output(RouteDecision)
```

### Model Tiers Per Component

| Component | Model needed |
|-----------|-------------|
| Router | Small/cheap LLM or fine-tuned classifier |
| Rewriter | Small LLM |
| Decomposer | Medium LLM (needs reasoning) |
| HyDE generation | Small LLM |
| Final synthesis | Flagship LLM |
| Self-evaluation/critique | Medium-to-flagship LLM |

---

## Prompt Injection Detection

### Ready-Made Solutions

| Tool | Type |
|------|------|
| **Azure AI Content Safety — Prompt Shields** | Managed API |
| **LLM Guard** (Protect AI) | Open source |
| **NeMo Guardrails** (NVIDIA) | Open source |
| **Lakera Guard** | Commercial API |
| **Protect AI Guardian** | Commercial |
| **Arthur AI Shield** | Commercial |

### Azure Content Safety — Query Layer Sanitization

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import ShieldPromptOptions, TextContent

client = ContentSafetyClient(endpoint, credential)

response = client.shield_prompt(
    ShieldPromptOptions(
        user_prompt=TextContent(text=user_query),
        documents=[TextContent(text=chunk) for chunk in retrieved_docs]
    )
)

if response.user_prompt_analysis.attack_detected:
    state["blocked"] = True
```

Detects: direct injection, morse, base64, encoded attacks, jailbreaks, indirect injection in retrieved documents.

### In LangGraph Flow

```
[User Query] → [Content Safety Node] → conditional edge
                                          |
                            clean ←───────┼───────→ attack detected
                              |                          |
                        [Router Node]              [Block Node]
```

### Azure AI Foundry — Built-in Protection

If deployed in Foundry, content safety is automatic. Configure in:
```
Project → Deployments → Model → Content Filters → Prompt attacks (toggle)
```

---

## Evaluation

### Azure AI Evaluation SDK (Azure equivalent of DeepEval)

```bash
pip install azure-ai-evaluation
```

```python
from azure.ai.evaluation import (
    GroundednessEvaluator,
    RelevanceEvaluator,
    CoherenceEvaluator,
    FluencyEvaluator,
    RetrievalEvaluator,
)

groundedness = GroundednessEvaluator(model_config)

result = groundedness(
    query="What is our refund policy?",
    context="Refunds are processed within 7 days...",
    response="Refunds take about a week to process."
)
```

### Metric Mapping

| DeepEval | Azure AI Evaluation |
|----------|-------------------|
| faithfulness | GroundednessEvaluator |
| answer_relevancy | RelevanceEvaluator |
| contextual_precision | RetrievalEvaluator |
| hallucination | GroundednessEvaluator |
| toxicity | ViolenceEvaluator, HateUnfairnessEvaluator |
| summarization | CoherenceEvaluator, FluencyEvaluator |

### Multi-turn & MCP Evaluation — Current State

| What | Maturity | Best tool today |
|------|----------|----------------|
| Single-turn RAG eval | Mature | Azure AI Evaluation, DeepEval, RAGAS |
| Safety / injection | Mature | Azure Content Safety |
| Multi-turn conversation | Early | LangSmith (tracing) + custom metrics |
| Tool selection correctness | Immature | Custom evaluators + golden traces |
| MCP-specific evaluation | Doesn't exist | Roll your own with assertion tests |
| End-to-end agent evaluation | Emerging | LangSmith, Braintrust, custom |

### Custom Tool-Use Evaluator

```python
def tool_use_evaluator(*, query, response, tool_calls, expected_tools):
    actual = [t["name"] for t in tool_calls]
    precision = len(set(actual) & set(expected_tools)) / len(actual) if actual else 0
    recall = len(set(actual) & set(expected_tools)) / len(expected_tools)
    return {"tool_precision": precision, "tool_recall": recall}
```

---

## Observability — K8s / AKS Deployment

### LangSmith Setup (simplest)

```yaml
# deployment.yaml
env:
  - name: LANGCHAIN_TRACING_V2
    value: "true"
  - name: LANGCHAIN_API_KEY
    valueFrom:
      secretKeyRef:
        name: langsmith-secret
        key: api-key
  - name: LANGCHAIN_PROJECT
    value: "my-agent-prod"
```

If using LangGraph — tracing is automatic. No decorators needed.

For custom functions:
```python
from langsmith import traceable

@traceable(name="router_node")
def route_query(state):
    ...
```

### Azure-Native Stack (no external vendor)

```python
from opentelemetry import trace
from azure.monitor.opentelemetry import configure_azure_monitor

configure_azure_monitor(connection_string="...")

tracer = trace.get_tracer("agent")

with tracer.start_as_current_span("route_query") as span:
    span.set_attribute("query", user_query)
    span.set_attribute("route", chosen_route)
```

### Comparison

| | LangSmith | Azure-native |
|--|-----------|-------------|
| Setup effort | 2 env vars | OTel + Container Insights |
| Agent-specific traces | Excellent | Generic spans |
| Data stays in Azure | No (cloud) / Yes (self-hosted Enterprise) | Yes |
| Cost at 1M traces/month | ~$1,000-3,000 | ~$50-200 |

---

## LangSmith PII Redaction

### Level 1: Kill switch

```bash
LANGSMITH_HIDE_INPUTS=true
LANGSMITH_HIDE_OUTPUTS=true
```

### Level 2: Regex-based anonymizer

```python
from langsmith.anonymizer import create_anonymizer
from langsmith import Client

anonymizer = create_anonymizer([
    {"pattern": r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", "replace": "<email>"},
    {"pattern": r"\b\d{3}-\d{2}-\d{4}\b", "replace": "<SSN>"},
    {"pattern": r"\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b", "replace": "<card>"},
])

client = Client(anonymizer=anonymizer)
```

### Level 3: Microsoft Presidio (NLP-based)

```python
from presidio_analyzer import AnalyzerEngine
from presidio_anonymizer import AnonymizerEngine
from langsmith import Client

analyzer = AnalyzerEngine()
anonymizer_engine = AnonymizerEngine()

def presidio_anonymize(data):
    for message in data.get("messages", []):
        content = message.get("content", "")
        results = analyzer.analyze(
            text=content,
            entities=["PERSON", "PHONE_NUMBER", "EMAIL_ADDRESS", "US_SSN"],
            language="en"
        )
        anonymized = anonymizer_engine.anonymize(text=content, analyzer_results=results)
        message["content"] = anonymized.text
    return data

client = Client(
    hide_inputs=presidio_anonymize,
    hide_outputs=presidio_anonymize
)
```

### Level 4: Per-function control

```python
from langsmith import traceable

def redact_inputs(inputs: dict) -> dict:
    return {k: "<redacted>" if k in ("ssn", "email", "password") else v
            for k, v in inputs.items()}

@traceable(process_inputs=redact_inputs)
def my_node(state):
    ...
```

### Precedence Order

```
LANGSMITH_HIDE_INPUTS=true    → skips everything (nuclear option)
         ↓ (if false)
anonymizer (create_anonymizer) → takes precedence over hide_inputs
         ↓ (if not set)
hide_inputs / hide_outputs     → client-level transform
         ↓ (if not set)
process_inputs on @traceable   → function-level transform
```

---

## LangSmith Pricing

| Tier | Cost | Traces |
|------|------|--------|
| Developer (free) | $0 | 5,000/month |
| Plus | $39/seat/month | 10k included, then $2.50/1k (14-day) or $5.00/1k (400-day) |
| Enterprise | Custom | Custom volume, negotiated |

Enterprise hosting options:
- Cloud (US/EU) — fully managed by LangChain
- Hybrid — SaaS control plane + self-hosted data plane
- Self-hosted — fully in your VPC

---

## Learning Timeline

| Phase | What | Duration |
|-------|------|----------|
| Foundations | Embeddings, vector DBs, basic RAG | 2-3 weeks |
| Intermediate | Chunking, reranking, hybrid search, eval | 2-3 weeks |
| Agent Core | ReAct loop, tool use, planning, LangGraph | 3-4 weeks |
| Advanced Retrieval | Query transformation, CRAG, Self-RAG, Graph RAG | 3-4 weeks |
| Production | Observability, security, caching, cost, multi-agent | 3-4 weeks |

Total: ~3-4 months at 1.5-2 hours daily with hands-on building.

Fastest path: Build one project end-to-end first (basic RAG → add agentic loop → add evaluation), then circle back to theory.
