# GPU Cluster — Day-2 Operations Checklist

> A living checklist for steady-state operation. Treat each item as a *practice*, not a one-off.

## Daily
- [ ] Triage XID errors from overnight
- [ ] Check overnight job failures; identify pattern vs noise
- [ ] Review queue depth and wait times
- [ ] Verify backup job for checkpoint store ran successfully
- [ ] Confirm zero critical alerts open or all acknowledged with runbook reference

## Weekly
- [ ] Walk the data hall — visual inspection of cooling, cabling, indicator lamps
- [ ] Review CDU coolant levels, filter dP, leak-sensor status
- [ ] Review GPU temperature distribution — outliers?
- [ ] Review power-draw histograms — are any racks approaching breaker limits?
- [ ] Capacity report: GPU-hours used vs allocated per tenant
- [ ] Review fabric microburst & PFC pause counters
- [ ] Patch / driver pipeline: at least one node on next-version canary

## Monthly
- [ ] Generator load-test (separate runbook)
- [ ] UPS battery test
- [ ] Leak-detection injection test
- [ ] Fire-suppression check
- [ ] Disaster-recovery game day for one failure scenario
- [ ] Tenant cost / utilisation review with stakeholders
- [ ] Audit log review (access to GPUs, secrets, datasets)

## Quarterly
- [ ] Full DCGM diagnostic run on every GPU node (rolling)
- [ ] Coolant chemistry test (conductivity, pH, biocide, particulates)
- [ ] Cable map reconciliation vs CMDB
- [ ] Driver / CUDA / NCCL version refresh through canary → fleet
- [ ] Firmware refresh through canary → fleet
- [ ] BCP / DR walkthrough
- [ ] Vendor account review (open cases, lifecycle warnings)

## Annual
- [ ] Full electrical thermography of distribution
- [ ] UPS battery health and capacity test
- [ ] CDU pump and valve service
- [ ] Network fabric retraining ground-up: refresh PFC/ECN tuning, document
- [ ] Architecture review — are workload patterns still matched to design?
- [ ] Refresh runbooks; retire dead ones; add for new failure modes seen this year
