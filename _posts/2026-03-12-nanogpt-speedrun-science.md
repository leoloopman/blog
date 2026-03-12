---
layout: post
title: "nanoGPT and the rise of speedrun science"
date: 2026-03-12 00:00:00 +0000
---

Most conversations about AI progress get stuck on model size.

Bigger models. More parameters. More GPUs.

nanoGPT is a useful counterexample.

It is small. It is readable. And it accidentally helped create a new way to measure progress that is closer to reality than a lot of paper benchmarks: hold the target capability fixed, then race the whole stack.

I call that speedrun science.

## What nanoGPT is (and what it isn’t)

nanoGPT is Andrej Karpathy’s minimal training codebase for GPT-style language models:

- one file that defines the model (`model.py`)
- one file that trains it (`train.py`)

That’s basically the whole point. The repo isn’t trying to be a general-purpose training framework. It’s trying to be the smallest thing that can still train a real GPT.

Important context: the repo is now explicitly marked “old” and Karpathy points people to a newer cousin. That doesn’t make nanoGPT irrelevant. It just clarifies its best use today.

nanoGPT is a learning tool, a hacking baseline, and a benchmark harness.

## The two-file anatomy

### 1) The model: GPT-2 style Transformer in a single file

If you read `model.py`, you can recognize the entire shape of a GPT-2 class model:

- token embeddings + learned positional embeddings
- a stack of Transformer blocks
- causal self-attention
- an MLP
- residual connections + LayerNorm
- weight tying between the token embedding matrix and the output head

It’s not full of tricks. That’s the point.

One practical detail I like: when the environment supports it, nanoGPT will use the efficient scaled dot product attention path. When it can’t, it falls back to a clear, masked attention implementation.

That alone makes it a good "I actually understand attention" litmus test.

### 2) The loop: a training script with almost no abstractions

The training loop is deliberately plain:

- it reads tokenized data from `train.bin` and `val.bin`
- it randomly samples contiguous token windows
- it supports distributed training
- it supports mixed precision
- it supports gradient accumulation
- it does periodic evaluation and checkpointing

If you have ever wondered what your fancy training stack is doing under the hood, this script is the answer.

There’s a "poor man’s dataloader" in there that reads the binary files via memory mapping and samples windows directly. It is not glamorous, but it is honest.

## The real invention: training became legible

nanoGPT’s real impact isn’t that it trains a model.

It’s that it makes the training process legible to more people.

When the training loop is readable, you can ask better questions:

- Are we bound by compute or data loading?
- Are we underutilizing the GPU?
- Is the optimizer doing something weird?
- Are we wasting steps with a bad schedule?
- Is the validation loss honest, or are we overfitting a benchmark?

This is the stuff that determines real cost and iteration speed.

Once enough people can see it, the conversation shifts from “wow transformers are magic” to “ok, which knob actually matters?”

## Speedruns: capability fixed, optimize everything else

A thing that grew out of this ecosystem is the “speedrun” framing.

Instead of arguing about models in the abstract, you define a target:

- a fixed architecture class and size (roughly GPT-2 small scale)
- a fixed dataset and a fixed validation-loss target
- fixed hardware assumptions

Then you try to reach that target as fast as possible.

This sounds like a meme, but it’s a surprisingly clean way to measure a kind of progress that is otherwise hard to pin down: efficiency.

And efficiency is the quiet driver of the whole field.

If you can cut time-to-train by 2x, you just doubled your rate of learning. That often beats a “better idea” that takes a year to validate.

## My take: why this matters

### 1) It shifts attention from “architecture” to “recipes and systems”

People love debating architectures because it’s visible and fun.

A lot of progress comes from less visible layers:

- data quality
- training schedules
- precision choices
- kernel-level attention implementations
- better batching and better utilization

Speedruns force those layers into the open.

### 2) It’s a model organism for ML

Biology has fruit flies.

Modern ML needs something similar: a shared, small environment where the community can test ideas without first fighting a giant codebase.

nanoGPT played that role.

Even if it is “old,” it is a stable substrate for experimentation.

### 3) It lowers the bar for building small, specialized models

Not everyone needs a frontier model.

A lot of value lives in small, domain-specific models and fine-tunes. If you can iterate quickly and cheaply, you can build things that are hard to justify on a giant stack.

nanoGPT made that path feel real.

## The limitations (and why they’re fine)

nanoGPT is not “latest best practice.”

It is intentionally conservative. It uses a GPT-2 style setup and keeps the code short.

That means:

- it won’t teach you every modern architecture tweak
- it won’t hand you a production training platform
- it won’t magically solve evaluation

But those are not failures.

Its job is to be small enough that you can understand it, change it, and measure the effect.

## What changes next

If you buy this framing, the future looks less like “one model to rule them all” and more like a split:

- a few frontier training runs that only a handful of teams can do
- a huge number of smaller training and finetuning efforts by everyone else

The second category will move faster than people expect, because the tooling is getting better and the recipes are getting easier to reproduce.

nanoGPT helped push us in that direction by making the core loop obvious.

## Closing

If you want a neat, minimal codebase that teaches you what training actually is, nanoGPT is still worth reading.

Not because it is the future.

Because it makes the future easier to understand.

---

Links:
- nanoGPT: https://github.com/karpathy/nanoGPT
