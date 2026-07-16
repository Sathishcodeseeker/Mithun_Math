# Agent Design Playbook

## Core Principle

Agent systems should start as deterministic workflows. Add LLM-driven agentic loops only where fixed code paths are insufficient.

For production systems, especially aviation or shopping workflows, the model should not be the final authority for business-critical decisions. Use the model for intent detection, extraction, reasoning support, comparison, and explanation. Use deterministic code for validation, scoring, policy enforcement, persistence, and external side effects.

The practical pattern is:

```text
Deterministic workflow first
+ small LLM steps where useful
+ structured outputs
+ validation
+ bounded retry
+ deterministic fallback
+ human approval where needed
```

## Think-Act-Observe

Think-Act-Observe is the agent loop used by ReAct-style systems.

```text
Think   = model decides what to do next
Act     = tool/API/function is called
Observe = tool result is added back to state
```

In LangGraph-style workflows, this usually maps to:

```text
agent node
  -> tool node
  -> ToolMessage / observation
  -> agent node again
  -> final response when no tool call remains
```

This loop is useful when the system must gather facts dynamically. It is risky when the model is allowed to keep trying without bounds or when tool calls create external side effects.

Common failure points:

| Step | Failure |
| --- | --- |
| Think | Wrong intent, wrong tool, bad arguments, infinite loop |
| Act | Tool timeout, API failure, bad side effect, unavailable dependency |
| Observe | Malformed result, stale data, ambiguous result, oversized context |
| State | Missing or corrupted state passed to the next node |
| Routing | Wrong conditional edge, early termination, incorrect fallback |

## Failure And Fallback

Retry and fallback are different.

```text
Retry = try the same or corrected operation again because it is likely safe and useful.
Fallback = stop the uncertain agent loop and move to a controlled deterministic path.
```

Recommended failure path:

```text
tool/model failure
  -> classify failure
  -> retry only if safe
  -> stop after max retries
  -> validate known inputs
  -> ask user only for missing or ambiguous fields
  -> call deterministic API/workflow when inputs are complete
  -> block, reconcile, or escalate if the operation is unsafe
```

For read-only operations, bounded retries are usually acceptable. For side-effect operations, such as payment, order creation, route filing, or database mutation, do not blindly retry. First reconcile the external state, then decide the next action.

Examples:

| Scenario | Correct fallback |
| --- | --- |
| Product search API fails | Show cached/popular results or ask user to retry |
| Missing product size | Ask user for size only |
| Payment timeout | Check payment status before retrying |
| Order status unknown | Reconcile payment and order records |
| Aviation weather unavailable | Do not recommend route as safe |
| NOTAM/TFR data stale | Warn or block depending on policy |

## Controlled Agent Workflow

A controlled agent workflow makes the model predictable by reducing open-ended decisions.

Recommended path:

```text
use-case catalog
  -> semantic intent routing
  -> fixed prompt/template per intent
  -> temperature 0
  -> structured output
  -> schema validation
  -> deterministic business logic
  -> fallback if validation fails
```

This is stronger than a fully autonomous agent because the system controls what each step is allowed to do.

Example use-case catalog for a route optimizer:

```text
OPTIMIZE_ROUTE
COMPARE_ROUTES
EXPLAIN_ROUTE
CHECK_WEATHER_IMPACT
CHECK_FUEL_TIME
CHECK_NOTAM_RESTRICTION
ASK_MISSING_INPUT
OUT_OF_SCOPE
```

The semantic router chooses the workflow. The model extracts and explains. Deterministic services decide and validate.

## Why Temperature 0 Is Not Enough

Temperature 0 makes model output more stable, but it does not guarantee full determinism in production.

If every layer is frozen, output can be deterministic:

```text
same model weights
same tokenizer
same exact prompt bytes
same decoding algorithm
same seed and tie-breaking
same runtime
same hardware behavior
same context and tool outputs
```

Hosted model APIs rarely guarantee all of that. Output can still vary because of:

| Cause | Why it matters |
| --- | --- |
| Near-tie token probabilities | Tiny numeric differences can flip the selected token |
| GPU floating point behavior | Parallel operations can produce small logit differences |
| Provider updates | Same model name may point to updated weights or wrappers |
| Hidden context | System prompts, safety layers, or tool schemas can change |
| Post-processing | Structured output repair or safety filtering may alter output |
| Tool outputs | Search, weather, DB, and API results may change |
| Retries/routing | Infrastructure may route requests to different backends |

Use temperature 0, but do not rely on it as the only control.

Production controls:

```text
temperature 0
+ prompt versioning
+ structured output schema
+ model version pinning where available
+ deterministic tool outputs
+ validation
+ idempotent APIs
+ golden test set
```

## Docker-Deployed Model Reproducibility

Deploying the model yourself in Docker improves reproducibility because you control more of the stack:

```text
model weights
tokenizer
runtime version
prompt template
decoding config
seed
container image
dependency versions
```

Docker still does not automatically guarantee perfect determinism. Variation can come from:

| Cause | Why |
| --- | --- |
| GPU kernels | Floating point and parallel execution can vary |
| Host drivers | Same container can run on different CUDA/driver behavior |
| Quantization | Low precision can amplify tiny differences |
| Dynamic batching | Other requests can change execution path |
| Speculative decoding | Draft/verify implementations may vary |
| Multi-GPU inference | Scheduling and synchronization can differ |
| Tool outputs | External data still changes |

For the strongest reproducibility:

```text
temperature = 0
top_p = 1
fixed seed
fixed model and tokenizer
fixed inference engine version
fixed quantization
same hardware class
no dynamic batching
no speculative decoding unless deterministic
non-streaming output for tests
```

The practical claim should be:

```text
Docker makes model behavior reproducible enough for controlled workflows.
Deterministic code still guarantees business behavior.
```

## Flight Route Optimizer Path

For a Jepp ForeFlight-style flight route optimizer, the path should be advisory and deterministic at the safety boundary.

```text
user message
  -> request/session context
  -> intent classifier
  -> safety/scope gate
  -> entity extractor
  -> entity normalizer
  -> required input validator
  -> clarification loop if needed
  -> flight context builder
  -> constraint builder
  -> data retrieval
  -> data freshness and provenance validator
  -> candidate route generator
  -> hard-rule pruning
  -> deterministic route scoring
  -> model-assisted explanation/comparison
  -> deterministic final validator
  -> confidence and warnings
  -> human review
  -> advisory output
  -> audit log
```

The intent classifier should run before optimization. Example intents:

```text
OPTIMIZE_ROUTE
COMPARE_ROUTES
EXPLAIN_ROUTE
CHECK_WEATHER_IMPACT
CHECK_FUEL_TIME
CHECK_NOTAM_RESTRICTION
OUT_OF_SCOPE
```

Entity normalization must be deterministic:

```text
"Teterboro" -> KTEB
"Miami Opa Locka" -> KOPF
"tomorrow morning" -> exact UTC time after clarification if needed
"Gulfstream 650" -> aircraft profile id
```

The constraint builder should produce one object that downstream nodes trust:

```json
{
  "origin": "KTEB",
  "destination": "KOPF",
  "aircraft": "G650",
  "departure_time": "2026-07-10T15:00:00Z",
  "avoid_weather": true,
  "avoid_tfr": true,
  "fuel_policy": "standard_reserve",
  "region": "US",
  "optimization_goal": "fuel_time"
}
```

Hard-rule pruning must happen before scoring. Do not score illegal or unsafe routes and then pick the highest score.

Reject candidates that violate:

```text
restricted airspace
TFR
aircraft performance limits
airport/runway constraints
mandatory fuel reserve
invalid airway/fix sequence
missing required weather/NOTAM data
stale safety-critical data
```

The model can compare and explain:

```text
Route A is better because it reduces fuel while avoiding modeled weather.
```

Deterministic validation decides:

```text
Route A is allowed or blocked.
```

## Production Checklist

Use this checklist before calling an agent workflow production-ready.

- Use-case catalog exists and unsupported intents are handled.
- Intent router has confidence thresholds and clarification behavior.
- Prompts are versioned and tied to specific workflows.
- Model outputs are structured and schema validated.
- Entity normalization is deterministic.
- Tool contracts are typed and errors are structured.
- Retries are bounded and classified by safety.
- Fallback leaves the open-ended agent loop.
- Side-effect tools are idempotent or approval-gated.
- Data freshness and provenance are checked.
- Final business decision is deterministic.
- Human approval boundary is explicit for safety-critical actions.
- Audit log records inputs, tool calls, observations, decisions, and fallback reasons.
- Golden scenario tests cover success, ambiguity, stale data, missing data, tool failure, unsafe route, and out-of-scope intent.

## Summary

The reliable agent pattern is:

```text
Classify intent
-> normalize inputs
-> build constraints
-> run bounded agent/tool workflow
-> validate outputs
-> fall back deterministically when needed
-> keep final authority in code and humans, not the model
```

For Flight Route Optimizer, this means:

```text
LLM = classify, extract, compare, explain
deterministic services = validate, score, block, approve
human = operational decision
```
