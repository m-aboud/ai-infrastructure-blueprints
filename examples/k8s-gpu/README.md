# Kubernetes GPU Scheduling Examples

Concrete manifests for common AI workload patterns on a GPU-equipped Kubernetes cluster.

## Prerequisites

- Kubernetes 1.28+
- NVIDIA GPU Operator (or Device Plugin + DCGM Exporter)
- Nodes labelled with `nvidia.com/gpu.product` (auto-applied by GFD)
- For MIG examples: nodes with MIG enabled and `nvidia.com/mig.strategy=mixed` (or `single`)

Verify:
```bash
kubectl get nodes -L nvidia.com/gpu.product,nvidia.com/mig.strategy
kubectl describe node <gpu-node> | grep -i nvidia
```

## Files

| File | Purpose |
|---|---|
| [`00-namespace-quota.yaml`](00-namespace-quota.yaml) | Tenant namespace with ResourceQuota + LimitRange |
| [`10-priority-classes.yaml`](10-priority-classes.yaml) | Production / training / batch priority tiers |
| [`20-training-job.yaml`](20-training-job.yaml) | Multi-GPU training Job with topology-aware placement |
| [`30-inference-mig.yaml`](30-inference-mig.yaml) | Inference Deployment using MIG slices + HPA |

## Apply order

```bash
# Cluster-wide objects first
kubectl apply -f 10-priority-classes.yaml

# Tenant scaffolding
kubectl apply -f 00-namespace-quota.yaml

# Workloads
kubectl apply -f 20-training-job.yaml
kubectl apply -f 30-inference-mig.yaml
```

## What to verify

- `kubectl describe quota -n ai-team-a` — GPU quota consumed matches expectation
- `kubectl get jobs -n ai-team-a -w` — training job progresses
- `kubectl top pod -n ai-team-a` — pod CPU/mem within limits
- DCGM dashboards in Grafana — GPU utilization rises when job is running
- `kubectl get hpa -n ai-team-a` — inference HPA tracks load

## Hardening to do before production

- Add `NetworkPolicy` per workload
- Add `SecurityContext` with `runAsNonRoot`, `readOnlyRootFilesystem` where image supports it
- Replace `image: ghcr.io/example/...` placeholders with your own signed images
- Add Kueue / Volcano for queue-aware scheduling rather than relying on raw Job objects
- Add OPA Gatekeeper / Kyverno policies enforcing resource limits and image provenance
