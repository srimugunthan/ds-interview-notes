Based on your background — ~11 years in DS/ML, hands-on with LangGraph, LLM inference optimization (speculative decoding, vLLM, KV cache), and production ML systems in financial services — here's a targeted curriculum that skips the basics and focuses on what actually matters for your level.

---

## PyTorch Mastery Curriculum for Senior ML Practitioners

### Orientation: What to Skip

You likely already have solid intuitions about tensors, basic autograd, and `nn.Module`. Don't spend time on those. Your gaps are almost certainly in:

- PyTorch internals (memory model, dispatcher)
- Custom autograd functions
- Production compilation (`torch.compile`)
- Distributed execution (DDP / FSDP)

---

### Module 1 — Memory Architecture (1 week)

The goal here is to stop treating PyTorch as a black box for memory.

- `torch.Storage` and how tensor views share underlying memory — understand `strides`, `offset`, `contiguous()`
- Why `view()` fails on non-contiguous tensors; when `reshape()` silently copies
- In-place ops (`_` suffix) and version counter errors inside computational graphs
- Pinned memory (`pin_memory=True`) and async CPU→GPU transfers
- Why `time.time()` lies — `torch.cuda.synchronize()` and correct benchmarking

Practical exercise: Profile a real data pipeline from one of your existing projects (the click propensity system with Triton is a good candidate). Identify CPU→GPU transfer bottlenecks and apply pinned memory.

---

### Module 2 — Autograd Internals (1–2 weeks)

This is the single highest-leverage module for someone doing adversarial ML, custom loss functions, and noise-robust learning (directly relevant to your AGCE / bi-tempered loss work).

- `grad_fn` chain construction during forward pass; how `requires_grad` propagates
- `retain_graph=True` — when you actually need it vs. when it's a memory leak
- `torch.no_grad()` vs `tensor.detach()` — the precise semantic difference
- Writing `torch.autograd.Function` with custom `forward()` and `backward()` — this is where Jacobian-vector products become concrete
- `register_full_backward_hook` for gradient inspection — invaluable for debugging vanishing gradients in deep GNN architectures (relevant to your financial crime KG work)

Practical exercise: Reimplement your AGCE loss as a proper `torch.autograd.Function` with an explicit backward pass. Compare gradient norms against the standard CE loss.

---

### Module 3 — `nn.Module` Mastery (1 week)

You probably use `nn.Module` daily, but a few internals are worth cementing:

- `__call__` vs `forward` — why bypassing `__call__` silently breaks hooks
- `nn.Parameter` vs plain tensors; `nn.ModuleList` / `nn.ModuleDict` for correct parameter registration
- `register_buffer()` for non-trainable state — relevant if you're building BatchNorm variants or custom normalization layers
- `state_dict` surgery: partial loading, key remapping, strict vs non-strict loading — useful for LLM fine-tuning workflows (directly relevant to your QLoRA work on RazorFunc)
- `model.train()` vs `model.eval()` internals: what actually changes in Dropout and BatchNorm

---

### Module 4 — Data Pipeline Engineering (3–4 days)

Given your Triton + FastAPI stack on the click propensity system, you want the pipeline to never be the bottleneck.

- `num_workers` and copy-on-write memory; tuning for your specific workload
- Custom `collate_fn` for variable-length sequences and multi-modal inputs
- `IterableDataset` for streaming use cases (relevant to any streaming transaction data pipelines in financial services)
- `worker_init_fn` to handle dataset sharding correctly across workers

---

### Module 5 — Production Optimization (2 weeks)

This is where senior practitioners separate from mid-level. Highly relevant to your InstantReply (speculative decoding / vLLM) project.

**AMP:**
- `torch.cuda.amp.autocast` + `GradScaler` — gradient underflow mechanics, when to exclude specific layers from autocast
- Numerical stability traps in float16 (directly applicable to financial risk models where precision matters)

**torch.compile:**
- TorchDynamo (graph acquisition via Python bytecode interception), AOTAutograd (ahead-of-time autograd), Inductor (kernel fusion + codegen)
- `fullgraph=True` vs `dynamic=True` tradeoffs
- Graph breaks — how to diagnose and reduce them using `torch._dynamo.explain()`

**TorchScript / ONNX:**
- Tracing vs scripting and why dynamic control flow breaks tracing
- ONNX export pipeline; opset selection; custom op registration for non-standard layers

---

### Module 6 — Distributed Training (1–2 weeks)

Relevant both for large model training and for your Principal-level positioning — FSDP expertise is a genuine differentiator.

- Why `nn.DataParallel` is a GIL-bound single-process bottleneck
- DDP mechanics: gradient all-reduce, bucket communication, `find_unused_parameters`
- FSDP: parameter sharding strategies (FULL\_SHARD, SHARD\_GRAD\_OP, NO\_SHARD); mixed precision with FSDP; checkpoint activation to manage memory
- `torch.distributed` primitives: when to use collective ops directly

---

### Capstone Projects (pick one)

Given your portfolio direction, these are more meaningful than generic exercises:

1. **Custom Attention with Manual Autograd:** Implement a sparse or linear attention variant (relevant to FinVision / financial table extraction) using `torch.autograd.Function` with an explicit backward pass. Profile against `F.scaled_dot_product_attention`.
2. **Custom Optimizer:** Implement a noise-robust optimizer (building on your AGCE loss expertise) that modifies gradient updates based on sample confidence. Wire it into PyTorch's scheduler API.
3. **Distributed Fine-tuning with FSDP:** Take your QLoRA fine-tuning setup from RazorFunc and move it from single-GPU to FSDP-wrapped multi-GPU. Document memory savings per shard strategy.

---

### Books (from the referenced list, prioritized for your profile)

1. **Deep Learning with PyTorch (2nd Ed.)** — Antiga, Stevens, Viehmann, Huang — for Phases 1–2 (memory + autograd internals)
2. **Mastering PyTorch** — Jha — for Phases 3 and 5 (production engineering, AMP, TorchScript)
3. **Edward Yang's "PyTorch Internals" blog + PyTorch Dev Podcast** — for the C++ dispatcher, `TensorImpl`, and ATen layer; this is the material that differentiates Principal-level candidates in systems-heavy interviews

---

### Rough Timeline

| Module | Duration |
|---|---|
| Memory Architecture | 1 week |
| Autograd Internals | 1.5 weeks |
| nn.Module Mastery | 1 week |
| Data Pipeline Engineering | 4 days |
| Production Optimization | 2 weeks |
| Distributed Training | 1.5 weeks |
| Capstone | 2–3 weeks |

---
Given your target — Principal Data Scientist / Principal AI Engineer — the interview gauntlet typically has 4–5 distinct rounds. The curriculum needs to map directly to what gets tested, not just what's intellectually interesting.

---

## PyTorch Interview-Focused Curriculum

### What Actually Gets Tested at Principal Level

Before the modules, it's worth being precise about the interview surface area:

- **ML Systems Design rounds** — "Design a distributed training pipeline for X" — this is where FSDP/DDP knowledge gets used, but as *design vocabulary*, not code
- **Coding rounds** — Custom layers, autograd functions, debugging gradient flows
- **Deep-dive / bar-raiser rounds** — "Walk me through how autograd works" — internals knowledge tested verbally
- **Take-home or live debugging** — Given broken training code, diagnose the issue

The curriculum below is structured around *interview question types*, not just topics.

---

### Tier 1 — Must-Know Cold (Highest Interview Frequency)

These come up in nearly every senior/principal PyTorch interview.

**1. Autograd Mechanics (verbal + coding)**

What to be able to do:
- Explain `grad_fn` chain construction from scratch, without notes
- Explain `no_grad()` vs `detach()` with a concrete use case for each
- Write a `torch.autograd.Function` subclass with correct `forward()` and `backward()` live on a whiteboard
- Diagnose a gradient not flowing — enumerate the 5–6 reasons this happens (`detach()` in wrong place, in-place op on a leaf, `requires_grad=False`, etc.)

Likely interview question: *"You notice a layer's weights aren't updating during training. Walk me through how you'd debug this."*

Prep method: Write the autograd Function exercise from scratch 3–4 times until it's muscle memory. Practice the verbal explanation out loud.

---

**2. Memory Management**

What to be able to do:
- Explain `torch.Storage`, strides, and views without hand-waving
- Articulate exactly when a copy happens vs. a view (the `view()` vs `reshape()` distinction)
- Explain pinned memory and why it matters for throughput
- Know the common OOM causes and their fixes: gradient accumulation, `del` + `torch.cuda.empty_cache()`, activation checkpointing

Likely interview question: *"Your training job is hitting OOM at batch size 32 but the model fits at batch size 8. What are your options and what are the tradeoffs of each?"*

Prep method: Build a cheat sheet of OOM scenarios → fixes. Reproduce each one in a notebook so you've seen the actual error.

---

**3. `nn.Module` Internals**

What to be able to do:
- Explain `__call__` vs `forward` and why it matters for hooks
- Explain `register_buffer()` vs `nn.Parameter` — give a concrete example of when you'd use each
- Load a partial state dict (mismatched keys scenario) — write the code live
- Explain `train()` vs `eval()` — what *specifically* changes in BatchNorm and Dropout

Likely interview question: *"You're loading a pretrained model but your architecture has one extra layer. How do you handle the state dict mismatch?"*

---

### Tier 2 — Depth Expected at Principal Level

These distinguish Principal from Senior. You won't always be asked to code these, but you need to discuss them fluently.

**4. torch.compile and the 2.x Compilation Stack**

What to be able to do:
- Explain TorchDynamo → AOTAutograd → Inductor as a pipeline, one sentence each
- Explain what a *graph break* is, what causes one, and how you'd diagnose it
- Know when `torch.compile` hurts performance (small models, dynamic shapes, frequent recompilation)

Likely interview question: *"You added `torch.compile` to your training loop and it got slower. What would you investigate?"*

Prep method: Read the official torch.compile tutorial once, then read the graph break documentation. You don't need to memorize compiler internals — you need the *diagnostic vocabulary*.

---

**5. AMP and Numerical Precision**

What to be able to do:
- Explain loss scaling and why it exists (float16 underflow)
- Know which layer types should be excluded from autocast and why (loss functions, certain normalizations)
- Explain the `GradScaler` update loop — what happens if a gradient is inf/nan

Likely interview question: *"Your loss goes to NaN after switching to mixed precision. Walk me through your debugging process."*

---

**6. Distributed Training — Design Vocabulary**

At Principal level you're expected to reason about this architecturally, not necessarily implement it from scratch in an interview.

What to be able to do:
- Explain DDP's all-reduce gradient synchronization and bucket communication
- Contrast DDP vs FSDP — when would you choose one over the other (model size threshold, memory constraints)
- Explain `find_unused_parameters=True` — what it does and what the performance cost is
- Know activation checkpointing — the memory/compute tradeoff in one sentence

Likely interview question: *"You need to fine-tune a 13B parameter model on 4 A100s. Walk me through your setup."*

Prep method: Practice answering this specific question out loud. The answer should cover: FSDP vs DDP decision, mixed precision, gradient accumulation, activation checkpointing, and monitoring. That's the full Principal-level answer.

---

### Tier 3 — Differentiators (Use Strategically)

Bring these up proactively when the conversation allows. They signal depth beyond what most candidates show.

**7. Custom Autograd Functions for Non-Standard Layers**

Connecting to your existing work: your AGCE loss and bi-tempered loss implementations are perfect anchors here. Being able to say *"I implemented this loss function as a `torch.autograd.Function` because the standard autodiff path had numerical instability at the backward pass"* is a strong signal.

**8. Hooks for Debugging**

`register_full_backward_hook` for gradient monitoring is something most candidates haven't used. Mentioning it in a debugging scenario question immediately elevates your answer.

**9. TorchScript / ONNX Export**

Relevant if the role has a deployment focus. Know the tracing vs. scripting distinction and one concrete failure mode of tracing (dynamic control flow).

---

### Interview Question Bank to Drill

Organize your prep around these specifically:

**Debugging questions (most common):**
- Weights not updating — walk through the diagnosis
- Loss is NaN — walk through the diagnosis
- GPU utilization is low — walk through the diagnosis
- Training is slower with `torch.compile` — walk through the diagnosis
- OOM at training time — enumerate your options

**Design questions:**
- Fine-tune a large model on multiple GPUs — full setup walkthrough
- Build a low-latency inference pipeline — covers TorchScript/ONNX, batching, AMP
- Design a custom training loop from scratch without a trainer framework

**Conceptual / verbal:**
- Explain autograd in 2 minutes to a non-expert
- What's the difference between `detach()` and `no_grad()`
- When would you write a custom `autograd.Function`
- What exactly changes between `train()` and `eval()`

---

### Prep Schedule (6 weeks, part-time)

| Week | Focus | Output |
|---|---|---|
| 1 | Autograd internals — read + code | `autograd.Function` written from scratch; verbal explanation rehearsed |
| 2 | Memory model + OOM scenarios | OOM cheat sheet; `view` vs `reshape` examples in notebook |
| 3 | `nn.Module` internals + state dict surgery | Partial load code written; buffer vs parameter examples |
| 4 | AMP + `torch.compile` | NaN debugging flow; graph break diagnosis |
| 5 | Distributed design vocabulary | Large model fine-tuning answer rehearsed out loud |
| 6 | Mock interviews | Answer all 15 question bank questions on camera or with a peer |

---


---

The highest-ROI modules for your specific trajectory are **Autograd Internals** (ties directly to your adversarial ML and custom loss work) and **torch.compile + FSDP** (Principal-level differentiators in interviews and system design discussions).
