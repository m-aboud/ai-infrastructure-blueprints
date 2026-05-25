# AI-Ready Data Center Blueprint

> A planning blueprint for greenfield or retrofit data centers expected to host significant GPU workloads (training, fine-tuning, large-scale inference).

## Why AI workloads break traditional DC assumptions

| Traditional enterprise DC | AI workload reality |
|---|---|
| 5–10 kW per rack typical | 30–80+ kW per rack for dense GPU |
| Air cooling sufficient | Liquid (DLC) or hybrid often required |
| North-south traffic dominates | East-west traffic dominates (gradient sync, all-reduce) |
| Hours-long jobs are exceptional | Multi-day training jobs are routine |
| 99.99% availability driven by business | Availability driven by job-checkpoint frequency + scale |
| Storage IOPS optimised | Storage **bandwidth** optimised (parallel reads) |

## 1. Power & density

**Design questions**
- What is the target rack density (kW per rack)?
- Is supply gear (transformers, switchboards) sized for that density × rack count?
- Are A/B power paths available to every GPU rack?
- Is the UPS topology and battery runtime sized for orderly checkpoint + shutdown?
- Is generator capacity validated for the new peak load (use [`dc-ops generator-margin`](https://github.com/mohammedabood/datacenter-ops-toolkit))?

**Common pitfalls**
- Specifying rack density without validating upstream MV/LV gear
- Forgetting transformer derating in hot climates
- Under-sizing PDU breakers for inrush during simultaneous GPU spin-up

## 2. Cooling

**Design questions**
- Air cooling (with rear-door heat exchangers), direct liquid cooling (DLC), or immersion?
- What is the supply water temperature (ASHRAE W3/W4 unlocks free cooling more often)?
- Is the chilled-water plant sized for **total heat rejection**, not just peak IT load?
- Is the cold-aisle containment intact post-retrofit?
- Are leak-detection and shut-off valves on every DLC manifold?

**See:** [`liquid-cooling-planning.md`](liquid-cooling-planning.md)

## 3. Network fabric

**Design questions**
- East-west bandwidth target (per-GPU): 200 / 400 / 800 Gbps?
- RoCEv2 (Ethernet) or InfiniBand (NDR / XDR)?
- Topology: fat-tree, dragonfly, rail-optimised?
- Oversubscription ratio (1:1 typical for training fabric)?
- Storage fabric: separate or converged?

**See:** [`ai-network-fabric.md`](ai-network-fabric.md)

## 4. GPU orchestration

**Design questions**
- Bare-metal Slurm, Kubernetes, or both?
- For Kubernetes: NVIDIA GPU Operator, MIG strategy, multi-tenancy model?
- Queue / scheduler: Kueue, Volcano, Run:ai, native?
- How are gang-scheduling and topology-aware placement handled?
- How is preemption handled for spot/lower-priority jobs?

**See:** [`examples/k8s-gpu/`](../examples/k8s-gpu/)

## 5. Storage

**Tiers**
- **Hot scratch** — local NVMe per GPU node, RAID0 or none
- **Warm parallel** — Lustre / WekaFS / GPFS / DAOS for training datasets
- **Cold object** — S3-compatible for checkpoints, model artefacts, datasets at rest

**Design questions**
- Aggregate read bandwidth required = `dataset throughput × concurrent jobs`
- Is the storage network separated from the compute fabric?
- Is checkpoint write throughput sized for the largest planned model?

## 6. Observability

**GPU-specific signals**
- `DCGM_FI_DEV_GPU_UTIL` — GPU utilisation
- `DCGM_FI_DEV_FB_USED` — framebuffer (VRAM) used
- `DCGM_FI_DEV_POWER_USAGE` — per-GPU power draw
- `DCGM_FI_DEV_GPU_TEMP` — temperature
- `DCGM_FI_DEV_XID_ERRORS` — hardware faults
- `DCGM_FI_DEV_NVLINK_REPLAY_ERRORS` — NVLink health
- Job-level: queue wait time, allocation time, training throughput (samples/sec)

**Recommended stack:** DCGM Exporter → Prometheus → Grafana (with NVIDIA dashboards) + Loki for job logs. See [observability-platform](https://github.com/mohammedabood/observability-platform).

## 7. Security & governance

- Tenant isolation: namespaces, NetworkPolicies, ResourceQuotas, MIG
- Secrets management (Vault / SealedSecrets / external-secrets)
- Image scanning + admission control (Kyverno / OPA Gatekeeper)
- Audit trail: who ran what, on what GPUs, for how long, at what cost
- Data residency requirements (especially in regulated markets)

## 8. Operational readiness

Before go-live:

- [ ] All GPU nodes pass burn-in (typically 72 h with `gpu-burn` or vendor DCT)
- [ ] DCGM XID/SBE/DBE counters baseline captured
- [ ] Cooling MTTF margin validated at peak load
- [ ] UPS battery autonomy load-tested
- [ ] Generator load-test successful (use [`dc-ops generator-margin`](https://github.com/mohammedabood/datacenter-ops-toolkit))
- [ ] Network fabric pass `nccl-tests` at design throughput
- [ ] Checkpoint write-test passes at design bandwidth
- [ ] Multi-job all-reduce stable for 24 h
- [ ] Runbooks complete for: GPU XID storm, fabric partition, cooling derate, PDU loss
- [ ] On-call rota and escalation matrix published

See also [`checklists/ai-dc-readiness.md`](../checklists/ai-dc-readiness.md).
