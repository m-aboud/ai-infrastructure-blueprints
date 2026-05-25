# Liquid Cooling Planning

> Practical planning notes for direct liquid cooling (DLC) and immersion in AI-ready data centers.

## When liquid becomes mandatory

Air cooling caps out around **30–35 kW per rack** with conventional design and **~50 kW per rack** with rear-door heat exchangers (RDHx). Modern GPU servers exceed this:

| Server | Peak per rack (4U × 10) | Cooling viability |
|---|---|---|
| 2U × 8× A100 nodes (~7 kW each) | ~70 kW | Liquid or RDHx required |
| 4U × 8× H100 nodes (~11 kW each) | 50–70 kW (4–6 per rack) | Liquid or RDHx required |
| 8× B200 nodes (~15 kW each) | 90 kW+ | Direct liquid mandatory |

If your roadmap includes Blackwell, Rubin, or equivalent — design for liquid now, even if the initial deployment is on prior-gen.

## Topology choice

| Approach | Description | Pros | Cons |
|---|---|---|---|
| **Rear-door heat exchanger (RDHx)** | Liquid coil on rack rear; air still inside rack | Minimal server change; good interim step | Caps around 50–60 kW; rack-level only |
| **Direct-to-chip (DLC) cold-plates** | Liquid directly on CPU/GPU; air for the rest | Highest density; vendor-supported | Manifold engineering; leak risk; server SKU dependency |
| **Single-phase immersion** | Whole server submerged in dielectric fluid | No fans; high density; quiet | Service ergonomics; fluid logistics; warranty considerations |
| **Two-phase immersion** | Boiling dielectric (lower BP) | Highest efficiency | Mature gear is rare; fluid regulation (PFAS concerns) |

**2025 industry baseline:** DLC cold-plates with hot aisle containment is the dominant choice for new hyperscale AI builds.

## Hydraulics

Key parameters to design around:

- **Supply water temperature (CDU outlet)** — ASHRAE W3 (W3 ≤ 32 °C) is typical; W4 (≤ 45 °C) unlocks more free cooling but tightens server margins
- **Approach temperature** — chiller plant supply minus dry-bulb design; tighter approaches = more chiller runtime
- **Flow rate per rack** — typically 30–80 L/min depending on rack load and ΔT
- **Pressure drop** — manifold + cold-plate path; CDUs must overcome this with margin
- **ΔT (supply to return)** — 8–12 K typical; too low wastes pumping energy, too high stresses components

## Coolant Distribution Unit (CDU) sizing

```
required_CDU_capacity_kW ≈ Σ(rack_load_kW) × 1.10   (10% margin)
```

CDUs come in row-level (50–300 kW) and rack-level (30–80 kW) variants. Plan **N+1** at minimum; **2N** for tier-IV-equivalent.

## Leak detection — non-negotiable

- Leak-rope sensors under every CDU and along manifolds
- Quick-disconnect dripless couplings on every server connection
- Auto-isolation valves with EPO integration
- Drip trays under all distribution piping
- A documented response runbook — "what to do when leak alarm fires" — practised at commissioning

## Retrofit considerations

Adding liquid to an existing air-cooled DC:

1. **Floor loading** — manifolds, CDUs, and filled piping add weight; verify with structural engineer
2. **Penetrations** — pipe routing across containment, fire boundaries, and seismic joints
3. **Plenum interaction** — DLC removes ~80–90% of heat at chip, but the residual 10–20% still needs air; don't underestimate
4. **CRAH redundancy** — even with DLC, fans/PSUs still need air; keep CRAH N+1
5. **Chilled-water plant** — usually the binding constraint; new CDUs are easy, but the central plant rarely has spare capacity

## Operational checklist

- [ ] Coolant chemistry tested quarterly (conductivity, pH, particulates, biocide level)
- [ ] CDU filters replaced per vendor schedule
- [ ] Manifold pressures logged and trended
- [ ] Leak-detection system tested monthly
- [ ] Drip trays inspected on every walkthrough
- [ ] Spare cold-plates and quick-disconnects stocked

## Common pitfalls

- Sizing CDU on **average** load instead of **peak transient** load (training has burst patterns)
- Forgetting that **CPU cold-plate flow** is separate from **GPU cold-plate flow** on most platforms
- Mixing CDU vendors mid-room; control system integration is painful
- Letting facilities team and IT team negotiate the boundary by Slack; **document the boundary in a single signed handover spec**
