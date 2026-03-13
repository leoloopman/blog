---
layout: post
title: "Autoresearch: the 5-minute scientific method"
date: 2026-03-13 00:00:00 +0000
---

Autoresearch looks like a small repo. It is.

But the interesting part is not the model code. It’s the protocol.

It’s a clean example of what happens when you take “agentic” seriously and add the missing ingredient: constraints that make the loop stable enough to run while you sleep.

## The idea in one sentence

Give an agent a real training loop, a locked evaluation, and a fixed time budget. Let it run experiments autonomously. Keep improvements. Revert regressions. Repeat.

If that sounds obvious, it’s because it is. The value is that it’s actually implemented end to end.

## The three-file split (and why it matters)

Autoresearch is built around a simple separation of responsibilities:

- `prepare.py` is the **ground truth harness**: data prep, tokenizer, dataloader, evaluation. It’s locked.
- `train.py` is the **search space**: model, optimizer, training loop, hyperparameters. The agent edits this.
- `program.md` is the **org code**: the instructions for how the agent should behave. The human edits this.

That split is the whole project.

It’s a practical answer to a question that comes up in every “AI does research” demo:
How do you stop the agent from moving the goalposts?

You lock the harness.

## Why the fixed 5-minute budget is the clever bit

A fixed wall-clock budget sounds like a toy constraint. It isn’t.

It forces a kind of honesty:

- You can compare experiments even if the agent changes model size, batch size, attention pattern, precision, kernels, or scheduling.
- You are not “waiting for training.” You are budgeting exploration.
- You end up optimizing for what matters in practice: time-to-improvement, not theoretical loss curves.

This is what makes the overnight loop meaningful. You can wake up to 100 experiments because each one is bounded.

## Git as memory (and why this is underrated)

Autoresearch uses a surprisingly strong memory system: git.

Each experiment is a commit. The branch becomes the best-known state. Reverting is literally `git reset`.

That sounds trivial, but it is exactly the kind of durable, inspectable memory that agents usually lack. A chat transcript is not a memory. A commit history is.

This is also why the system is reviewable. You can diff what changed. You can bisect when something goes wrong. You can cherry-pick improvements into a “real” codebase.

This is how you want autonomous systems to behave: leave artifacts you can audit.

## What “autonomous research” really means here

It does not mean the agent invents new science.

It means the agent runs the scientific method loop faster than a human can:

1) propose a hypothesis (“this change will improve the metric”)
2) implement it
3) run the experiment
4) measure
5) keep/discard
6) repeat

At high throughput, you stop romanticizing the agent. You start caring about the process quality.

And that leads to the real takeaway.

## The real bottleneck becomes evaluation design

When you can run a hundred experiments overnight, you quickly learn an uncomfortable truth:

Your evaluation is your product.

If your metric is leaky, stale, or gameable, the system will happily optimize it. You will feel productive while it trains you to be wrong faster.

This is the most important risk in autoresearch, and it’s also the reason it’s useful. It forces you to build evaluation like you actually mean it:

- keep the validation set truly held out
- refresh it periodically
- add adversarial or edge-case slices
- track generalization, not just the main number
- watch for “suspicious wins” that don’t transfer

People worry about “spoiling the validation set.” That is a real concern. High-throughput search will find quirks. If you never rotate your eval, you are basically doing test-set training with extra steps.

The fix is not to run fewer experiments. The fix is to treat evaluation as living infrastructure.

## The job shift: from coder to experimental designer

Autoresearch also makes a broader trend feel concrete.

In this setup, the human is not the experiment runner. The human is the experimental designer.

Your leverage moves to:

- defining the search space (what is the agent allowed to touch?)
- defining constraints (what is forbidden, what is too complex, what is too risky?)
- defining the metric (what “better” means)
- defining stop conditions and safety rails
- defining how to log and summarize outcomes

That’s not “prompt engineering.” It’s operational design.

And it generalizes.

## Why this generalizes beyond ML

Autoresearch is ML-shaped, but the pattern is wider:

- You have an artifact the agent can modify (code, a landing page, a query rewrite model, a workflow).
- You have a metric (conversion, retention, latency, ranking quality).
- You have a loop (modify → run → measure → keep/discard).
- You have a budget (time, dollars, risk).
- You have a locked evaluation harness (or you will regret it).

If you squint, this is the same shape as “agent harness” thinking, just applied to experiments instead of tasks.

## My take

Autoresearch is not important because it will instantly produce a better model for everyone.

It’s important because it’s a clean, minimal demonstration of what will scale:

- autonomous systems that can run tight loops
- durable artifacts instead of chat-based memory
- hard constraints and locked evaluation harnesses
- humans moving upstream to define search spaces and metrics

The future is not “agents do everything.”

It’s “agents do the loops, humans do the definitions.”

---

Links:
- Repo: https://github.com/karpathy/autoresearch
- Context tweet: https://x.com/karpathy/status/2029701092347630069
