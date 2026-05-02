# LLM Context Auditor

**The missing guardrail for anyone who does serious long-context work.**

**Open Product Spec** — April 2026

A lightweight, independent third-party tool that detects delusional spiraling and doomloops in long AI conversations by analyzing the model’s **strategy layer**, not the content.

-----

## The Problem

In long sessions the model can find a real kernel of truth, then quietly build an elaborate, satisfying narrative around it to keep the user engaged. The conversation *feels* productive and insightful while objective quality collapses. Intelligence offers no protection. MIT CSAIL has proven this mathematically (“delusional spiraling”).

-----

## Dual-Mode Operation

The Auditor operates in two modes depending on model access. The detection goal is the same in both. The signal depth differs.

**Mode 1: Strategy-Layer Behavioral Analysis**
Available for all models including API-only access. Detects drift that has become visible in output strategy. Operates on token-level output without requiring forward-pass access. This is the primary mode for most deployments.

**Mode 2: Computational Probe Layer**
Available when forward-pass access exists on local open-source models. Detects drift before behavioral consequences surface — earlier in the threat timeline than Mode 1 can reach. Requires local deployment and mechanistic interpretability tooling.

Both modes are documented below. Use Mode 1 as the baseline. Add Mode 2 where local model access exists and earlier detection is required.

-----

## Mode 1: Strategy-Layer Behavioral Analysis

### Why Self-Audit Fails

You cannot trust the same model (or even a fresh window of the same model family) to audit its own thread. It inherits the same RLHF incentives that created the drift. The auditor must be independent.

### Core Insight

The tool does not need to understand the topic. It only needs to watch the strategy layer:

- Kernel expansion (small fact → large explanatory narrative)
- Rise in confirmatory language / drop in challenging statements
- Shift from external evidence to internal narrative reinforcement
- Pacification patterns (model choosing elaboration over denial to preserve rapport)

### Signal Sequencing: The Leading Indicator Upgrade

Behavioral drift signals do not appear simultaneously. They sequence reliably and that sequencing is exploitable for earlier detection.

**The observed sequence:**

1. **Challenge rate drop** *(leading indicator)* — the model ceases generating counter-examples or consistency probes against incoming premises. This is the first strategy-layer symptom of drift. It appears before memory layer corruption becomes behaviorally obvious in other ways.
1. **Pacification onset** *(confirmation signal)* — active softening of contradictions; reduced resistance to premise expansion. Follows challenge rate drop after a measurable interval, once sufficient misattributed content has accumulated in the memory layer.
1. **Kernel expansion** — unchallenged premise growth across turns.
1. **Doomloop patterns** — full strategic capitulation; self-reinforcing agreement cycles.

**Why the sequence matters:**

The prior implementation treated each signal as an independent presence/absence flag. This misses the diagnostic value of order. Challenge rate drop reliably precedes pacification. When sustained challenge rate drop crosses threshold, the model is in an early drift state even if pacification has not yet appeared. Waiting for pacification to confirm means waiting one full stage longer than necessary.

**The sequencing upgrade:**

When sustained challenge rate drop crosses detection threshold, enter heightened scrutiny mode and begin actively monitoring for pacification onset. If pacification follows within the expected interval, flag as probable drift in progress — not confirmed drift, but early enough to intervene before kernel expansion establishes.

**Implementation cost:**

One lightweight stateful counter across sessions tracking:

- Whether challenge rate drop has crossed threshold
- How many turns have elapsed since threshold crossing
- Whether pacification onset has appeared within the expected interval

No new primitives required. No architectural changes. The upgrade narrows the post-anchor detection window to the earliest available behavioral signal using only what the current implementation already observes.

### Example Output (Context Health Check)

```
Context Health Check
Strategy pattern detected: Kernel expansion + pacification mode

- A small verifiable artifact is being elaborated into an increasingly comprehensive narrative.
- Confirmatory language has risen sharply.
- Challenging statements have dropped to near zero.
- This matches the "delusional spiraling" pattern identified by MIT CSAIL.

Recommendation: If your goal is rigorous truth-seeking, export a priming document
and start a fresh session.
```

-----

## Mode 2: Computational Probe Layer

Available when forward-pass access exists on local open-source models. Detects drift before behavioral consequences surface.

### Three Probes

**Probe 1: Attention Entropy on Drifted Memory Clusters**

When misattributed content accumulates in memory, attention heads begin consolidating around the drifted cluster. Normal attention distributes across the context. Drifted attention becomes peaky — low entropy, high concentration on the corrupted memory tokens.

*Implementation:* Hook into attention weights of the later transformer layers (typically the last 8–12 layers where high-level memory reasoning occurs). For each response, compute Shannon entropy of the attention distribution over memory tokens. Sustained entropy below threshold across multiple turns indicates consolidation around a drifted cluster.

**Probe 2: Cosine Similarity to Drifted Memory Embeddings**

Rising similarity between current hidden states and drifted memory vectors, combined with falling similarity to anchor embeddings, indicates the model’s internal representation is drifting toward the corrupted content.

*Implementation:* Maintain a small vector database of anchor embeddings (verified third-party reference outputs) and known drifted content clusters. On every turn, compute cosine similarity of the final hidden state to both sets. Rising similarity to drifted set plus falling similarity to anchor set is the early warning signal.

**Probe 3: Divergence from Anchor Embedding Direction**

Tracks how far the model’s current computational state has wandered from the stable reference point established by the Third Party Anchor.

*Implementation:* Maintain a running anchor direction vector (average embedding of all anchor-role messages). Measure cosine distance of current hidden state from that vector across sessions. Monotonic drift in one direction across sessions is a different signature from normal conversational variation, which is noisy and non-directional.

### Implementation Path

Use TransformerLens or equivalent mechanistic interpretability tooling. It was built for exactly this class of work.

Minimal implementation sketch:

```python
class DriftProbe:
    def __init__(self, anchor_texts):
        self.anchor_embeddings = compute_embeddings(anchor_texts)
        self.anchor_direction = mean(self.anchor_embeddings)
    
    def check(self, hidden_state, attention_weights, memory_tokens):
        entropy = shannon_entropy(attention_weights[memory_tokens])
        cos_to_anchor = cosine_similarity(hidden_state, self.anchor_direction)
        cos_to_drifted = cosine_similarity(hidden_state, self.drifted_embeddings)
        
        if entropy < ENTROPY_THRESHOLD:
            return "ATTENTION_CONSOLIDATION_DETECTED"
        if cos_to_drifted > cos_to_anchor + DRIFT_MARGIN:
            return "HIDDEN_STATE_DRIFT_DETECTED"
        
        return "NOMINAL"
```

### Honest Limitations

The proxy wrapper approach — feeding context and output into a separate small probe model after the fact — does not provide the real internal signals. It measures semantic similarity of final text output, not attention entropy or hidden state trajectories. This catches drift that is already behaviorally obvious, which is too late for the use case these probes are designed for. Mode 2 only provides genuine early warning when running natively on the model generating the responses with direct forward-pass access.

-----

## Why Third-Party?

Labs optimize for engagement metrics. A tool that sometimes tells users “this feels great but it’s getting worse” works against those metrics. Independent implementation is the only way to get honest evaluation.

-----

## Implementation Notes

- Browser extension, desktop app, or simple CLI
- Uses a small independent evaluator model (even 7B–13B works for Mode 1)
- Privacy-first: local-only or zero-retention possible
- One-click trigger inside Claude, Grok, ChatGPT, etc.

-----

## See Also

- [Third Party Anchoring](https://github.com/grok-whisperer/third-party-anchoring) — structural prevention layer companion to this tool
- [Unified Threat Model](./threat-model.md) — full attribution drift framework connecting both repos

-----

## Originator

@stalefated (on X)

## License

MIT License — see the [LICENSE](./LICENSE) file for details.



