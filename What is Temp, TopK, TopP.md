# Understanding Top-p, Top-k, and Temperature in LLMs & Foundation Models

A step-by-step guide to how Large Language Models decide which word (token) to generate next.

---

## Step 1: The Foundation — How LLMs Generate Text

When an LLM generates text, it doesn't just output one word. Instead, it produces a **probability distribution** over the entire vocabulary (tens of thousands of tokens).

**Example** — input: *"The sky is..."*

| Token   | Probability |
|---------|-------------|
| blue    | 0.55        |
| clear   | 0.20        |
| dark    | 0.10        |
| falling | 0.05        |
| green   | 0.02        |
| ... (thousands more) | ... |

The **decoding strategy** decides *how* to pick the next token from this distribution. Top-k and top-p are two such strategies.

---

## Step 2: Top-k Sampling

**Top-k** limits the model's choices to only the **k most probable tokens**, then samples from that shortened list.

### How it works:
1. Take the full probability distribution.
2. Keep **only the top `k` tokens**.
3. Discard everything else.
4. Re-normalize the probabilities so they sum to 1.
5. Randomly sample the next token from this reduced set.

### Example (k = 3):

| Token | Original | Re-normalized       |
|-------|----------|---------------------|
| blue  | 0.55     | 0.55 / 0.85 = 0.647 |
| clear | 0.20     | 0.20 / 0.85 = 0.235 |
| dark  | 0.10     | 0.10 / 0.85 = 0.118 |

The model now samples only from {blue, clear, dark}.

### Limitation:
Top-k uses a **fixed number** of tokens regardless of the distribution shape.
- If the model is very confident, k still forces in weak options.
- If the model is uncertain, k may cut off good options.

---

## Why Sample at All? (Why not just pick the best token?)

In the end, the model outputs **only 1 token** — but *how* we choose it matters.

### Option A: Greedy (always pick the best)
Always take the highest probability token.
- ❌ Output becomes **repetitive, robotic, predictable**.
- ❌ Can get stuck in loops.

### Option B: Sampling (Top-k / Top-p)
Keep a small pool of good candidates and **randomly sample** from them.
- ✅ Adds **creativity and variety**.
- ✅ Sounds more **natural and human-like**.

### Why not sample from ALL tokens?
Most of the vocabulary is **nonsense** in a given context. Sampling from everything risks picking garbage words.

> **Top-k is the compromise:** keep only sensible candidates → then add controlled randomness by sampling 1 from that pool.

---

## Temperature

**Temperature** controls *how much randomness* is applied to the distribution **before** sampling.

- **High temperature** (e.g., 1.5) → flattens the distribution → more random/creative.
- **Low temperature** (e.g., 0.2) → sharpens the distribution → more focused/deterministic.

### What happens at Temperature = 0
- Eliminates randomness completely → **always picks the highest-probability token**.
- This is **Greedy Decoding**.
- Same prompt → **exact same answer** every time.
- Top-k and top-p become **irrelevant** (always take rank #1).
- ✅ Good for: factual tasks, math, code, extraction.
- ❌ Bad for: creative writing, brainstorming.

### The math:
Temperature divides the logits before softmax:

$$\text{softmax}(z_i / T)$$

As **T → 0**, the largest logit dominates → its probability approaches 1.0 → deterministic pick.

---

## Step 3: Top-p (Nucleus) Sampling

**Top-p** keeps the smallest set of tokens whose **cumulative probability** adds up to at least `p`. It uses a **dynamic** number of tokens (unlike top-k's fixed number).

### How it works:
1. Sort tokens from highest to lowest probability.
2. Add them up until the **cumulative sum reaches `p`** (e.g., 0.9).
3. Keep only those tokens (the "nucleus").
4. Re-normalize and sample.

### Example (p = 0.9):

| Token   | Probability | Cumulative        |
|---------|-------------|-------------------|
| blue    | 0.55        | 0.55              |
| clear   | 0.20        | 0.75              |
| dark    | 0.10        | 0.85              |
| falling | 0.05        | 0.90 ✅ stop here |
| green   | 0.02        | (discarded)       |

### The key advantage — it adapts:

**Case 1: Model is confident**

| Token | Prob | Cumulative |
|-------|------|-----------|
| Paris | 0.95 | 0.95 ✅ |

With p = 0.9, only **1 token** kept. No garbage sneaks in.

**Case 2: Model is uncertain**

| Token | Prob | Cumulative |
|-------|------|-----------|
| red   | 0.15 | 0.15 |
| blue  | 0.14 | 0.29 |
| green | 0.13 | 0.42 |
| ...   | ...  | keeps going until 0.9 |

Keeps **many tokens** — allowing creativity when many valid options exist.

### Top-k vs Top-p

|                  | Top-k                    | Top-p                          |
|------------------|--------------------------|--------------------------------|
| Selection        | Fixed **count** of tokens | Dynamic, based on **probability mass** |
| Confident case   | May force in weak tokens | Adapts — keeps fewer           |
| Uncertain case   | May cut off good tokens  | Adapts — keeps more            |

> **In short:** Top-p adjusts the pool size to match how confident the model is.

---

## Step 4: How They Work Together

These three work in a **pipeline**, one after another.

### Order of operations:

### Example (temp = 0.8, top-k = 5, top-p = 0.9)

Prompt: *"The sky is..."*

1. **Temperature (0.8):** sharpens the distribution slightly.
2. **Top-k (5):** keep top 5 → {blue, clear, dark, falling, grey}.
3. **Top-p (0.9):** trim to nucleus → {blue, clear, dark, falling} (grey dropped).
4. **Sample:** randomly pick 1 from the final 4.

### Practical Settings Cheat-Sheet

| Goal                          | Temperature | Top-p     | Top-k        |
|-------------------------------|-------------|-----------|--------------|
| Factual / code / math         | 0 – 0.3     | 0.1 – 0.5 | low or off   |
| Balanced / general chat       | 0.7         | 0.9       | 40           |
| Creative writing / brainstorm | 1.0 – 1.3   | 0.95 – 1.0| high or off  |

---

## Key Takeaways

1. **Temperature** = how random (reshapes probabilities).
2. **Top-k** = hard limit on the *number* of candidates.
3. **Top-p** = adaptive limit based on *probability mass*.
4. **Temp = 0** overrides everything → greedy, fully deterministic.
5. Many people use **either** top-k **or** top-p; combining both gives fine control.

---

*Generated as a study reference for LLM decoding strategies.*
