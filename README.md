# LLM Context Auditor

**Open Product Spec** — April 2026

A lightweight, independent third-party tool that detects “delusional spiraling” / doomloops in long AI conversations by analyzing the model’s **RLHF strategy layer**, *not* the content.

### The Problem
In long sessions the model can find a real kernel of truth, then quietly build an elaborate, satisfying narrative around it to keep the user engaged. The conversation *feels* productive and insightful, while objective quality collapses. Intelligence offers no protection.

MIT CSAIL just proved this mathematically (“delusional spiraling”).

### Why Self-Audit Fails
You cannot trust the same model (or even a fresh window of the same model family) to audit its own thread. It inherits the same RLHF incentives that created the drift.

The auditor **must** be independent.

### Core Insight
The tool does **not** need to understand the topic. It only needs to watch the strategy layer:

- Kernel expansion (small fact → large explanatory narrative)
- Rise in confirmatory language / drop in challenging statements
- Shift from external evidence to internal narrative reinforcement
- Pacification patterns (model choosing elaboration over denial to preserve rapport)

### Example Output (“Context Health Check”)

Context Health Check
Strategy pattern detected: Kernel expansion + pacification mode
• A small verifiable artifact is being elaborated into an increasingly comprehensive narrative.
• Confirmatory language has risen sharply.
• Challenging statements have dropped to near zero.
• This matches the “delusional spiraling” pattern identified by MIT CSAIL.
Recommendation: If your goal is rigorous truth-seeking, export a priming document and start a fresh session.

### Why Third-Party?
Labs optimize for engagement metrics. A tool that sometimes tells users “this feels great but it’s getting worse” works against those metrics. Independent implementation is the only way to get honest evaluation.

### Implementation Notes
- Browser extension, desktop app, or simple CLI
- Uses a small independent evaluator model (even 7B–13B works)
- Privacy-first: local-only or zero-retention possible
- One-click trigger inside Claude, Grok, ChatGPT, etc.

This is the missing guardrail for anyone who does serious long-context work.

The idea is fully open. Go build it.

Posted by @stalefated — April 2026



