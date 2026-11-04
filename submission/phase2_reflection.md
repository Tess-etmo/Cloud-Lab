# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->402.7<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.121<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->458043.8<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->15301.1<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4458<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
 "1": 3620.241,
"2": 1729.335,
"4": 1869.561,
"8": 2139.574
The 2 was the fastest.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive version spends most of its time stuck running slow Python loops, processing just one single number at a time and wasting valuable CPU power on constant interpreter overhead. In contrast, the vectorized version hands entire matrices directly over to optimized BLAS libraries. These use special hardware instructions to process multiple numbers simultaneously in parallel, drastically reducing instruction overhead and speeding things up.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
Performance stopped helping and started hurting at 4 threads (latency increased from 1,729 ms to 1,869 ms).
The main reason is that a standard Colab runtime typically provides 2 physical vCPUs. When you set threads beyond the available physical hardware cores, threads start competing for the same execution units. The overhead of thread coordination, context-switching, and cache contention ends up costing more time than the parallel execution saves.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training steps rely almost entirely on large-scale matrix multiplications and tensor operations that run simultaneously across thousands of identical tokens and heads. This massive data parallelism perfectly suits the GPU's architecture, which features thousands of small parallel cores optimized for high-throughput arithmetic.
Conversely, sequential control-flow tasks would not see this speed-up.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Across the three experiments, computational performance was constrained by three distinct hardware and architectural boundaries:
Instruction & Execution Overhead (Software Bottleneck): In Experiment 1, scalar execution via Python loops spent CPU cycles on interpreter overhead and scalar dispatch rather than parallel math. Vectorization eliminated this, unlocking a massive speedup (e.g., from slow scalar loops down to optimized BLAS SIMD execution).

Core Saturation & Coordination Overhead (Hardware Boundary): In Experiment 2, scaling past the available physical hardware backfired. Moving from 2 to 4 threads increased execution latency from 1729.335 ms to 1869.561 ms, as thread context switching and core contention on the 2 vCPU runtime cost more than the extra threads could save.

Massive Parallel Throughput vs. Sequential Logic (Architecture Fit): In Experiment 3, the transformer step exploited thousands of GPU cores for matrix multiplication. Workloads bound by serial logic or memory transport (data moving across PCIe buses) fail to achieve this acceleration.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
Running a benchmark on multi-node distributed training (across multiple machines) would sharpen the results.
If scaling to multiple nodes introduced massive communication overhead that negated compute gains, it would prove that inter-node network latency, rather than local core limits, becomes the ultimate bottleneck.
<!--/answer-->
