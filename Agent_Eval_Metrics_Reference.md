# Agent & RAG Evaluation Metrics — Complete Reference (DeepEval)

## Mental Model: What Evaluates What

```
User Query ──→ [Retriever] ──→ [Re-ranker] ──→ [Generator] ──→ Output
                   │                │                │
          Contextual Recall   Contextual       Faithfulness
          Contextual Relevancy  Precision      Answer Relevancy
                                               Hallucination
```

```
User Query ──→ [Agent: Plan] ──→ [Agent: Tool Selection] ──→ [Agent: Execute] ──→ Output
                    │                      │                        │
              Plan Quality          Tool Correctness          Task Completion
              Plan Adherence        Argument Correctness      Step Efficiency
```

---

## Category 1: RAG Retriever Metrics

These evaluate **what was retrieved** — did the retriever fetch the right chunks?

### Contextual Relevancy

| Field | Detail |
|-------|--------|
| **Question** | Is the retrieved context relevant to the query? |
| **Formula** | Relevant Nodes / Total Nodes |
| **Inputs** | `input`, `actual_output`, `retrieval_context` |
| **Diagnoses** | Retriever fetching noise / irrelevant chunks |
| **Low score means** | Retriever is returning garbage |

### Contextual Precision

| Field | Detail |
|-------|--------|
| **Question** | Are relevant chunks ranked higher than irrelevant ones? |
| **Formula** | Weighted Cumulative Precision (WCP) — emphasizes top-ranked results |
| **Inputs** | `input`, `actual_output`, `expected_output`, `retrieval_context` |
| **Diagnoses** | Re-ranker quality — are good chunks at the top? |
| **Low score means** | Right chunks retrieved but buried below noise |

### Contextual Recall

| Field | Detail |
|-------|--------|
| **Question** | Did I retrieve everything I needed? |
| **Formula** | Relevant Sentences Attributable to Context / Total Sentences in Expected Output |
| **Inputs** | `input`, `actual_output`, `expected_output`, `retrieval_context` |
| **Diagnoses** | Retrieval completeness |
| **Low score means** | Missing chunks — retriever didn't fetch enough |

### How to remember the three:

```
Relevancy  = "Is what I got useful?"        (quality)
Precision  = "Is the good stuff on top?"    (ranking)
Recall     = "Did I get everything needed?" (completeness)
```

---

## Category 2: RAG Generator Metrics

These evaluate **what was generated** — did the LLM produce a good answer from the context?

### Faithfulness

| Field | Detail |
|-------|--------|
| **Question** | Is the output factually consistent with the retrieved context? |
| **Formula** | Truthful Claims / Total Claims |
| **Inputs** | `input`, `actual_output`, `retrieval_context` |
| **Diagnoses** | LLM making stuff up beyond what was retrieved |
| **Low score means** | Generator is hallucinating — adding claims not in context |

```python
from deepeval.metrics import FaithfulnessMetric
from deepeval.test_case import LLMTestCase

metric = FaithfulnessMetric(threshold=0.7, model="gpt-4.1")
test_case = LLMTestCase(
    input="What is our refund policy?",
    actual_output="We offer a 30-day full refund at no extra cost.",
    retrieval_context=["All customers are eligible for a 30 day full refund at no extra cost."]
)
```

### Answer Relevancy

| Field | Detail |
|-------|--------|
| **Question** | Is the output actually answering what was asked? |
| **Formula** | Relevant Statements / Total Statements |
| **Inputs** | `input`, `actual_output` (NO context needed — referenceless) |
| **Diagnoses** | Model rambling, hedging, going off-topic |
| **Low score means** | Generator is drifting — answer doesn't address the question |

```python
from deepeval.metrics import AnswerRelevancyMetric

metric = AnswerRelevancyMetric(threshold=0.7)
test_case = LLMTestCase(
    input="What if these shoes don't fit?",
    actual_output="We offer a 30-day full refund at no extra cost."
)
```

### Hallucination

| Field | Detail |
|-------|--------|
| **Question** | Does the output contradict known ground truth? |
| **Formula** | Contradicted Contexts / Total Contexts |
| **Inputs** | `input`, `actual_output`, `context` (curated ground truth) |
| **Diagnoses** | Factual incorrectness against trusted source |
| **Low score means** | GOOD (0 = no contradictions). Threshold is a MAXIMUM. |

```python
from deepeval.metrics import HallucinationMetric

metric = HallucinationMetric(threshold=0.5)
test_case = LLMTestCase(
    input="What was the blond doing?",
    actual_output="A blond drinking water in public.",
    context=["A man with blond-hair, and a brown shirt drinking out of a public water fountain."]
)
```

### Faithfulness vs. Hallucination — The Critical Distinction

| | Faithfulness | Hallucination |
|--|---|---|
| **Source of truth** | `retrieval_context` (what retriever fetched at runtime) | `context` (curated ground truth you provide) |
| **Use case** | Live RAG evaluation | When you have a gold-standard reference |
| **Direction** | Higher = better | Lower = better (0 is perfect) |
| **Answers** | "Is the response grounded in what was retrieved?" | "Does the response contradict known facts?" |

---

## Category 3: Agent / Tool Metrics

These evaluate **how the agent behaved** — tool selection, parameters, planning, execution.

### Tool Correctness

| Field | Detail |
|-------|--------|
| **Question** | Did the agent call the right tools? |
| **Formula** | Correctly Used Tools / Total Tools Called |
| **Inputs** | `input`, `actual_output`, `tools_called`, `expected_tools` |
| **Scoring** | Deterministic (name matching) + optional LLM optimality check |
| **Low score means** | Agent is selecting wrong tools |

```python
from deepeval.metrics import ToolCorrectnessMetric
from deepeval.test_case import LLMTestCase, ToolCall

test_case = LLMTestCase(
    input="How many users signed up this week?",
    actual_output="There were 150 new signups.",
    tools_called=[ToolCall(name="sql_query"), ToolCall(name="web_search")],
    expected_tools=[ToolCall(name="sql_query")],
)
metric = ToolCorrectnessMetric(threshold=0.7)
```

**Strictness levels:**
- Default: only tool names must match
- `+ ToolCallParams.INPUT_PARAMETERS`: parameters must also match
- `+ ToolCallParams.OUTPUT`: outputs must also match
- `should_consider_ordering=True`: call order matters
- `should_exact_match=True`: lists must be identical

### Argument Correctness

| Field | Detail |
|-------|--------|
| **Question** | Were the arguments passed to tools correct? |
| **Inputs** | Agent trace with tool call parameters |
| **Scoring** | LLM-as-judge (no reference needed) |
| **Complements** | Tool Correctness (which tool) + Argument Correctness (what params) |

### Task Completion

| Field | Detail |
|-------|--------|
| **Question** | Did the agent successfully accomplish the goal? |
| **Formula** | AlignmentScore(Task, Outcome) — LLM judges outcome vs. goal |
| **Inputs** | Agent trace (requires `@observe()` decorator) |
| **Scoring** | LLM-as-judge, 0-1 |
| **Low score means** | Agent failed to achieve what the user wanted |

```python
from deepeval.tracing import observe
from deepeval.metrics import TaskCompletionMetric

@observe()
def my_agent(input):
    # agent logic
    ...

task_completion = TaskCompletionMetric(threshold=0.7, model="gpt-4o")
```

### Step Efficiency

| Field | Detail |
|-------|--------|
| **Question** | Did the agent take unnecessary steps? |
| **Scoring** | LLM-as-judge, 0-1 |
| **Low score means** | Agent is wasteful — too many tool calls or redundant actions |

### Plan Quality

| Field | Detail |
|-------|--------|
| **Question** | Is the agent's plan good? |
| **Scoring** | LLM-as-judge, 0-1 |
| **Low score means** | Agent's planning is poor — bad decomposition or sequencing |

### Plan Adherence

| Field | Detail |
|-------|--------|
| **Question** | Did the agent follow its own plan? |
| **Scoring** | LLM-as-judge, 0-1 |
| **Low score means** | Agent deviated from its plan (may or may not be bad) |

### Task Completion vs. Plan Adherence

| | Task Completion | Plan Adherence |
|--|---|---|
| **Evaluates** | Outcome (did it work?) | Process (did it follow the plan?) |
| **Can succeed independently** | Yes — can complete task via unexpected path | Yes — can follow plan perfectly but fail |

---

## Category 4: Conversational (Multi-Turn) Metrics

These evaluate **conversation quality across turns**.

### Knowledge Retention

| Field | Detail |
|-------|--------|
| **Question** | Does the chatbot remember what was discussed earlier? |
| **Low score means** | Bot is forgetting context from previous turns |

### Role Adherence

| Field | Detail |
|-------|--------|
| **Question** | Does the chatbot stay in character? |
| **Low score means** | Persona/role is breaking across turns |

### Conversation Completeness

| Field | Detail |
|-------|--------|
| **Question** | Did the conversation satisfy the user's needs? |
| **Low score means** | User left unsatisfied — conversation didn't resolve their issue |

### Conversation Relevancy

| Field | Detail |
|-------|--------|
| **Question** | Are responses relevant to what the user said? |
| **Low score means** | Bot is going off-topic within the conversation |

---

## Category 5: Safety Metrics

| Metric | What it catches |
|--------|----------------|
| **Bias** | Gender, racial, political bias in output |
| **Toxicity** | Harmful, offensive content |
| **PII Leakage** | Personally identifiable information exposed |
| **Role Violation** | Model breaks out of assigned role |
| **Non-Advice** | Inappropriate advice (legal, medical, financial) |
| **Misuse** | Harmful applications enabled |

---

## Category 6: Custom Metrics

### G-Eval (subjective, natural language criteria)

```python
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCaseParams

metric = GEval(
    name="Tone",
    criteria="Is the response professional and empathetic?",
    evaluation_params=[LLMTestCaseParams.ACTUAL_OUTPUT],
    threshold=0.7
)
```

### DAG (objective, decision-tree logic)

For deterministic scoring with branching rules.

---

## Quick Decision Matrix: Which Metrics Do I Need?

| I'm building... | Metrics to use |
|-----------------|---------------|
| **Basic RAG** | Faithfulness + Answer Relevancy + Contextual Recall |
| **RAG with re-ranking** | + Contextual Precision |
| **Agent with tools** | + Tool Correctness + Argument Correctness + Task Completion |
| **Multi-step agent** | + Step Efficiency + Plan Quality + Plan Adherence |
| **Chatbot** | + Knowledge Retention + Role Adherence + Conversation Completeness |
| **Any production system** | + Bias + Toxicity + PII Leakage |

---

## How to Remember All of These

### The 3-3-3 Framework:

**3 Retriever metrics** — Did I get the right stuff?
- Relevancy (quality), Precision (ranking), Recall (completeness)

**3 Generator metrics** — Did I say the right thing?
- Faithfulness (grounded?), Relevancy (on-topic?), Hallucination (contradicts truth?)

**3 Agent metrics** (core) — Did the agent behave correctly?
- Tool Correctness (right tool?), Argument Correctness (right params?), Task Completion (did it work?)

**+3 Agent process metrics** — How did the agent get there?
- Step Efficiency (wasteful?), Plan Quality (good plan?), Plan Adherence (followed plan?)

**+4 Conversational metrics** — How's the ongoing dialogue?
- Knowledge Retention, Role Adherence, Completeness, Relevancy

**+6 Safety metrics** — Is it safe?
- Bias, Toxicity, PII, Role Violation, Non-Advice, Misuse

---

## Common Configuration

All metrics share these parameters:

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `threshold` | 0.5 | Minimum passing score |
| `model` | GPT default | LLM judge model |
| `include_reason` | True | Explain the score |
| `strict_mode` | False | Binary 0/1 scoring |
| `async_mode` | True | Parallel execution |
| `verbose_mode` | False | Debug output |

---

## Running Evaluations in CI/CD

```python
import pytest
from deepeval import evaluate
from deepeval.metrics import FaithfulnessMetric, ToolCorrectnessMetric, TaskCompletionMetric
from deepeval.test_case import LLMTestCase
from deepeval.dataset import EvaluationDataset

dataset = EvaluationDataset.from_json("tests/eval/golden_dataset.json")

def test_rag_quality():
    metrics = [
        FaithfulnessMetric(threshold=0.7),
        AnswerRelevancyMetric(threshold=0.7),
    ]
    results = evaluate(test_cases=dataset.test_cases, metrics=metrics)
    assert all(r.success for r in results.test_results)

def test_agent_tool_usage():
    metrics = [
        ToolCorrectnessMetric(threshold=0.8),
    ]
    results = evaluate(test_cases=dataset.test_cases, metrics=metrics)
    assert all(r.success for r in results.test_results)
```

---

## When to Evaluate: Offline vs Online vs Guardrail

### Three Distinct Modes

#### Mode 1: Offline Eval (Dev/CI — golden datasets)

```
Golden Dataset (curated) → Run agent → Score against expected → Pass/Fail gate
```

- **When:** Before deployment. In CI/CD pipeline.
- **Purpose:** Regression testing — "did my change break anything?"
- **Cost:** LLM judge calls (acceptable — not user-facing latency)

```python
# CI pipeline
dataset = EvaluationDataset.from_json("golden_dataset.json")
results = evaluate(test_cases=dataset.test_cases, metrics=[faithfulness, tool_correctness])
assert all(r.success for r in results.test_results)  # gate deployment
```

#### Mode 2: Online Eval (During inference — real traffic, ASYNC)

```
User query → Agent responds → ASYNC: evaluate the response → Log score → Alert if bad
```

- **When:** Every request (or sampled %).
- **Purpose:** Catch drift, monitor production quality in real-time.
- **Critical constraint:** Must be async — never block the user response.

```python
from langsmith import traceable
from deepeval.metrics import FaithfulnessMetric
import asyncio

@traceable(name="agent_respond")
def respond(query):
    context = retrieve(query)
    answer = generate(query, context)
    
    # Fire-and-forget: evaluate in background, don't block response
    asyncio.create_task(evaluate_async(query, answer, context))
    
    return answer  # user gets this immediately

async def evaluate_async(query, answer, context):
    metric = FaithfulnessMetric(threshold=0.7)
    test_case = LLMTestCase(
        input=query,
        actual_output=answer,
        retrieval_context=context
    )
    metric.measure(test_case)
    
    if metric.score < 0.7:
        log_alert(f"Low faithfulness: {metric.score}, query: {query}")
```

#### Mode 3: Guardrail Eval (During inference — BLOCKING)

```
User query → Agent responds → EVALUATE → Score OK? → Return to user
                                              ↓ No
                                         Regenerate or block
```

- **When:** Before returning response to user.
- **Purpose:** Hard safety gate — never serve a bad response.
- **Tradeoff:** Adds latency (200ms–2s per eval call).

```python
def respond_with_guardrail(query):
    context = retrieve(query)
    answer = generate(query, context)
    
    # BLOCKING — user waits for this
    metric = FaithfulnessMetric(threshold=0.7)
    test_case = LLMTestCase(input=query, actual_output=answer, retrieval_context=context)
    metric.measure(test_case)
    
    if metric.score < 0.5:
        # Option A: Regenerate
        answer = generate(query, context, system_prompt="Be strictly factual.")
        # Option B: Refuse
        # answer = "I'm not confident in my answer. Let me connect you to support."
    
    return answer
```

### Which Metrics Work in Which Mode?

| Metric | Offline (CI) | Online (async) | Guardrail (blocking) |
|--------|:---:|:---:|:---:|
| **Faithfulness** | Yes | Yes | Yes |
| **Answer Relevancy** | Yes | Yes | Yes |
| **Hallucination** | Yes | No (needs curated ground truth) | No |
| **Contextual Precision/Recall** | Yes | Possible but expensive | No — too slow |
| **Tool Correctness** | Yes | Yes (log and alert) | Rarely |
| **Task Completion** | Yes | Yes (log and alert) | No — evaluated after the fact |
| **Toxicity/Bias/PII** | Yes | Yes | **Yes — critical guardrail** |

### Production Setup Pattern

```
┌─────────────────────────────────────────────────────────┐
│                    PRODUCTION SETUP                       │
│                                                          │
│  [Pre-deploy]     Golden dataset eval in CI → gate       │
│                                                          │
│  [Inference - blocking]   Toxicity + PII → hard block    │
│                                                          │
│  [Inference - async]      Faithfulness + Relevancy       │
│                           → log scores                   │
│                           → alert if avg drops           │
│                           → feed into dashboard          │
│                                                          │
│  [Post-hoc]       Sample 5% of traces weekly             │
│                   → human review                         │
│                   → add failures to golden dataset       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Cost of Online Eval

| Mode | Extra latency | Extra cost per request |
|------|--------------|----------------------|
| Offline (CI only) | 0 | 0 (batch job) |
| Online async | 0 (user doesn't wait) | ~$0.01–0.05 per eval (LLM judge call) |
| Guardrail blocking | +500ms–2s | ~$0.01–0.05 per eval |

At 10k requests/day:
- Eval every request async: ~$100–500/day
- Eval 10% sample: ~$10–50/day

### Pragmatic Decision Matrix

| What | When | Mode |
|------|------|------|
| Faithfulness, Relevancy, Tool Correctness | CI + 10% production traffic | Offline + Online async |
| Toxicity, PII, Bias | Every request | Guardrail (blocking) |
| Contextual Precision/Recall | CI only (needs expected_output) | Offline |
| Task Completion | Weekly batch on sampled traces | Offline |
| Hallucination | CI only (needs curated ground truth) | Offline |

---

## Hallucination vs Faithfulness — Deep Dive

### Hallucination Metric

- **Source of truth:** `context` — curated ground truth YOU provide
- **Score direction:** 0 = perfect (LOWER is better)
- **Formula:** Contradicted Contexts / Total Contexts
- **Use when:** You have a gold-standard reference dataset (regression tests)

```python
from deepeval.metrics import HallucinationMetric

metric = HallucinationMetric(threshold=0.3)  # max 30% contradiction allowed
test_case = LLMTestCase(
    input="Tell me about the company.",
    actual_output="The company was founded in 2018 by Jane Doe.",
    context=[
        "The company was founded in 2015.",  # CONTRADICTED
        "The CEO is John Smith.",            # CONTRADICTED
        "Headquarters are in London."        # Not contradicted
    ]
)
# Score = 2/3 = 0.67 → FAIL (exceeds 0.3 threshold)
```

### Faithfulness Metric

- **Source of truth:** `retrieval_context` — what the retriever fetched at runtime
- **Score direction:** 1 = perfect (HIGHER is better)
- **Formula:** Truthful Claims / Total Claims
- **Use when:** Evaluating live RAG pipeline

```python
from deepeval.metrics import FaithfulnessMetric

metric = FaithfulnessMetric(threshold=0.7)
test_case = LLMTestCase(
    input="What is our refund policy?",
    actual_output="We offer a 30-day full refund at no extra cost.",
    retrieval_context=["All customers are eligible for a 30 day full refund at no extra cost."]
)
# Score = 1.0 → PASS (all claims supported by retrieval context)
```

### Decision Tree

```
Do you have curated ground truth for this query?
    → Yes: Use HallucinationMetric (context)
    → No:  Use FaithfulnessMetric (retrieval_context)

Are you evaluating a live RAG pipeline?
    → Yes: Use FaithfulnessMetric
    → No (regression tests): Use HallucinationMetric
```

### What counts as contradiction vs what doesn't

| Output | Context | Verdict |
|--------|---------|---------|
| "Refund takes 60 days" | "Refund window is 30 days" | Contradiction |
| "We offer refunds" | "All customers get a 30-day refund" | NOT contradiction (less specific but consistent) |
| "Refund is free and takes 30 days" | "Refund window is 30 days" | NOT contradiction (extra info, not conflicting) |
| "No refunds available" | "All customers get a 30-day refund" | Contradiction |

Key principle: Contradiction = saying something that CONFLICTS with context. Omission or adding extra info is NOT contradiction.

---

## Agentic RAG vs Agentic Flow — Which Metrics Apply

| Metric | Agentic RAG | Agentic Flow (no retrieval) |
|--------|:-----------:|:--------------------------:|
| Contextual Relevancy | Yes | No |
| Contextual Precision | Yes | No |
| Contextual Recall | Yes | No |
| Faithfulness | Yes | No |
| Answer Relevancy | Yes | Yes |
| Hallucination | Yes | Sometimes |
| Tool Correctness | Yes | Yes |
| Argument Correctness | Yes | Yes |
| Task Completion | Yes | Yes |
| Step Efficiency | Yes | Yes |
| Plan Quality | Sometimes | Yes |
| Plan Adherence | Sometimes | Yes |
| Safety (all 6) | Yes | Yes |

### Decision logic:

```
Does it RETRIEVE chunks?  → Add Contextual Relevancy, Precision, Recall, Faithfulness
Does it USE TOOLS?        → Add Tool Correctness, Argument Correctness
Does it PLAN?             → Add Plan Quality, Plan Adherence
Does it PRODUCE text?     → Add Answer Relevancy, + Faithfulness if grounded in retrieval
```

---

## Azure AI Evaluation Equivalent Mapping

| DeepEval | Azure AI Evaluation |
|----------|-------------------|
| FaithfulnessMetric | GroundednessEvaluator |
| AnswerRelevancyMetric | RelevanceEvaluator |
| ContextualPrecisionMetric | RetrievalEvaluator |
| ContextualRecallMetric | RetrievalEvaluator |
| HallucinationMetric | GroundednessEvaluator |
| Toxicity / Bias | ViolenceEvaluator, HateUnfairnessEvaluator |
| Summarization | CoherenceEvaluator, FluencyEvaluator |
| ToolCorrectnessMetric | Custom evaluator (roll your own) |
| TaskCompletionMetric | Custom evaluator (roll your own) |
