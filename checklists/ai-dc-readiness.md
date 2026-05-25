# AI Data Center Readiness Checklist

> Use as a structured go/no-go before declaring an AI-ready data center production-ready. Every item should be evidenced (test report, photo, screenshot).

## Power
- [ ] Total facility capacity confirmed for design GPU count + 25% headroom
- [ ] A/B paths verified to every GPU rack
- [ ] PDU breakers sized for inrush (simultaneous GPU spin-up)
- [ ] UPS battery autonomy load-tested at peak GPU load
- [ ] Generator load-test successful (full peak load for ≥ 30 min)
- [ ] Generator load margin documented ([`dc-ops generator-margin`](https://github.com/mohammedabood/datacenter-ops-toolkit))
- [ ] ATS transfer time within UPS ride-through
- [ ] EPO scope and exclusions documented and tested

## Cooling
- [ ] CDU plant sized to total IT load + 10% margin
- [ ] N+1 CDU redundancy verified by failure simulation
- [ ] Chilled-water plant capacity confirmed (NOT just CDU capacity)
- [ ] Supply water temperature within ASHRAE class for selected GPU
- [ ] Leak detection installed and tested on every CDU and manifold
- [ ] EPO interlock with leak detection verified
- [ ] Cold-aisle containment intact post-installation
- [ ] Coolant chemistry baseline captured

## Network
- [ ] Cable map complete, labelled, signed by network ops
- [ ] `nccl-tests` pass at design throughput across full cluster
- [ ] PFC / ECN configured and tested (RoCEv2)
- [ ] Subnet manager redundancy verified (InfiniBand)
- [ ] Single-leaf failure simulated; blast radius matches design
- [ ] Single-link failure simulated; jobs continue
- [ ] 24-hour soak test passes without pause storms / XID errors
- [ ] Out-of-band management network isolated and accessible during compute-fabric failure

## Compute
- [ ] All GPU nodes burn-in complete (typically 72 h `gpu-burn`)
- [ ] DCGM baseline (XID, SBE, DBE, NVLink) captured per node
- [ ] BMC / iDRAC / iLO firmware versions standardised
- [ ] Driver / CUDA / NCCL versions standardised across fleet
- [ ] Hardware inventory in CMDB / asset system
- [ ] Spare parts on site (NICs, PSUs, cold-plates, cables)

## Storage
- [ ] Parallel FS aggregate read BW matches design target
- [ ] Checkpoint write test passes at design BW
- [ ] Object store accessible from all GPU nodes
- [ ] Backup / replication strategy documented
- [ ] Quota enforcement tested

## Observability
- [ ] DCGM exporter on every GPU node, scraped by Prometheus
- [ ] GPU dashboards live in Grafana (utilization, temp, power, XID)
- [ ] Job-level metrics surfaced (queue wait, allocation, throughput)
- [ ] Log aggregation working for system + job logs
- [ ] Alerts defined: XID storm, NVLink errors, fabric congestion, cooling degrade, PDU loss
- [ ] All alerts have linked runbooks

## Operations
- [ ] On-call rota published
- [ ] Escalation matrix to vendor support documented
- [ ] Runbooks complete for: GPU XID storm, fabric partition, cooling derate, PDU loss, leak event
- [ ] Job submission and priority policy documented
- [ ] Quota and chargeback model agreed with stakeholders
- [ ] Change-management process for driver / firmware updates defined
- [ ] DR / continuity plan for total-facility loss

## Governance & security
- [ ] Tenant isolation model documented (namespaces, ResourceQuota, NetworkPolicy)
- [ ] Secrets management deployed
- [ ] Image scanning + admission control enforced
- [ ] Audit logging covers job submission, GPU allocation, data access
- [ ] Data classification & residency requirements captured
- [ ] Access reviews scheduled

## Final go / no-go

- [ ] **GO** — all items above are evidenced
- [ ] **CONDITIONAL** — list open items + mitigation
- [ ] **NO-GO** — list blockers + remediation owners

| Role | Name | Decision | Date |
|---|---|---|---|
| DC operations lead |  |  |  |
| Platform / SRE lead |  |  |  |
| Security |  |  |  |
| Sponsor / executive |  |  |  |
