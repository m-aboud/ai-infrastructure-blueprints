# GPU Cluster Sizing Guide

> Practical sizing methodology for training and inference clusters. Numbers below are 2025-era reference values — always validate against the current vendor reference architecture for your specific GPU.

## 1. Start from the workload, not the GPU

Three workload archetypes drive radically different cluster shapes:

| Archetype | Dominant constraint | Typical config |
|---|---|---|
| **Pre-training (large LLM)** | All-reduce bandwidth | Many nodes, 8× GPU/node, 400–800 Gbps fabric per GPU |
| **Fine-tuning / RLHF** | Memory + interconnect | Tens of GPUs, fast NVLink within node, decent fabric |
| **Inference (online serving)** | Latency + cost | Per-replica sizing, often MIG slices on L40S/H100 |
| **Inference (batch)** | Throughput | Spot-friendly, queue-driven, autoscale 0→N |

## 2. Compute sizing — training

The blunt formula:

```
total_GPU_hours ≈ (6 × P × T) / (η × FLOPS_per_GPU)
```

Where:
- `P` = model parameters
- `T` = training tokens
- `η` = realised throughput vs peak (0.4–0.6 typical at scale; lower for poorly-tuned jobs)
- `FLOPS_per_GPU` = peak FLOPS at the training precision (FP16/BF16/FP8)

**Worked example** — 70B model, 1.5T tokens, H100 FP8 (≈ 4 PFLOPS peak), η = 0.45:

```
6 × 70e9 × 1.5e12 = 6.3e23 FLOPs
6.3e23 / (0.45 × 4e15) = ~3.5e8 GPU-seconds ≈ ~97,000 GPU-hours
≈ 100 H100s × ~970 hours (40 days) — or 1,000 × 4 days
```

> **Reality check** — published runs (e.g. Llama 3.1 405B) report **2.4M H100-hours** for 15.6T tokens. Use the formula to ballpark, then add 25–40% for restarts, evals, debugging, and aborted runs.

## 3. Memory sizing — training

For a model of `P` parameters trained in BF16 with Adam optimiser:

| Item | Bytes per parameter |
|---|---|
| Weights (BF16) | 2 |
| Gradients (BF16) | 2 |
| Optimiser state (Adam FP32: 2× state) | 8 |
| Activations (depends on batch & sequence length) | variable |
| **Subtotal (non-activation)** | **~12 B/param** |

A 70B model needs ≥ 840 GB of GPU memory **just for weights+gradients+optimiser**, before activations. That maps to:
- 11× H100 80GB minimum, **with no headroom**
- More realistically: 16–24 GPUs with ZeRO-3 or tensor/pipeline parallelism

**Activations:** dominated by batch size × sequence length × hidden dim × layers. Use activation checkpointing to trade FLOPs for memory.

## 4. Network fabric sizing

For data-parallel training, all-reduce bandwidth requirement per GPU:

```
required_BW_per_GPU ≈ (2 × (n-1)/n × model_params × bytes_per_param) / step_time
```

A 70B model with ~10 ms step time across 64 GPUs ≈ 280 GB/s ≈ 2.2 Tbps per GPU. In practice you don't get this — you tune step time, gradient accumulation, and chunk size around fabric capability.

**Rule of thumb (2025):**
- < 16 GPUs in one node: NVLink handles it; fabric can be modest (100 G)
- 16–256 GPUs: 400 G InfiniBand or RoCEv2, fat-tree, 1:1 oversubscription
- 256+ GPUs: 800 G fabric or rail-optimised topology; vendor's reference design

## 5. Storage sizing

```
aggregate_read_bandwidth ≈ concurrent_jobs × dataset_throughput_per_job
```

For training, the typical pattern is:

| Tier | Sizing target |
|---|---|
| Local NVMe scratch | 4–8 TB per GPU node, 5–10 GB/s read |
| Parallel FS (training datasets) | 50–200 GB/s aggregate per cluster |
| Object (checkpoints, models) | Big, slow, cheap — sized by retention |

Checkpoint writes are bursty: for a 70B model, a full checkpoint is ~140 GB. Writing it in <30 s requires ≥ 5 GB/s sustained write to the parallel FS.

## 6. Power & cooling per node

Reference power draw (peak under training load):

| GPU node | Peak power |
|---|---|
| 8× A100 80GB SXM | 6–7 kW |
| 8× H100 SXM | 10–11 kW |
| 8× H200 SXM | ~11 kW |
| 8× B200 (Blackwell) | 14–15 kW |

Plan rack density and cooling around these. A 42U rack with 4 such servers (typical) is 40–60 kW — well beyond air-cooling for current-gen and next-gen.

## 7. Sizing summary table

| Cluster size | GPUs | Network | Storage BW | Rack density | Cooling |
|---|---|---|---|---|---|
| Small (single tenant) | 8–16 | NVLink + 100 G | 10 GB/s | 30–50 kW | Rear-door HX |
| Medium (R&D org) | 64–256 | 400 G IB/RoCE | 50 GB/s | 50–60 kW | DLC + air |
| Large (training cluster) | 1k–10k | 800 G rail-optimised | 200+ GB/s | 60–100+ kW | Full DLC |

## 8. Don't forget

- **Burn-in budget** — plan 1–2 weeks for cluster acceptance testing (gpu-burn, nccl-tests, dgemm)
- **Software stack** — driver, CUDA, NCCL, container runtime versions are co-dependent
- **Operational overhead** — SREs / cluster operators don't scale linearly; budget ~1 FTE per 256 GPUs at minimum
- **Cost of idle time** — a half-utilised cluster is the most common waste; invest in queue & priority management
