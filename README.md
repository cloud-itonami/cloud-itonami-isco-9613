# cloud-itonami-isco-9613

Open Occupation Blueprint for **ISCO-08 9613**: Sweepers and Related
Labourers.

This repository designs a forkable OSS business for a street/site-
cleaning (sweeping) route scheduling and logistics coordination
practice: a route scheduling and supply-coordination robot manages
crew/route records under a governor-gated actor, so a street/site-
cleaning crew keeps its own operating records instead of renting a
closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/streetsweep/` implements the
`StreetCleaningActor` as a `langgraph.graph/state-graph`
(`streetsweep.actor`) wired to a `Sweeper and Related Labourer
Advisor` (`streetsweep.advisor`) and an independent
`StreetCleaningGovernor` (`streetsweep.governor`), following the
itonami actor pattern (ADR-2607121000): `:intake -> :advise -> :govern
-> :decide -+-> :commit (:ok?) +-> :request-approval (:escalate?,
human-in-the-loop interrupt) +-> :hold (:hard?)`. HARD invariants
(always hold, never overridable): worker provenance, route
provenance, no-actuation (`:effect` must be `:propose`), a closed
op-allowlist (`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize a cleaning-work-execution
decision (e.g. authorizing a specific sweeping/cleaning operation to
proceed) *or* a route-safety-clearance decision (e.g. declaring a
route cleared for safety), or that would override a route safety
supervisor's judgment. Always-escalate paths (human sign-off
regardless of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Occupation context

ISCO-08 9613 (Sweepers and Related Labourers) is an **elementary
occupation** (ISCO major group 9): street/site-cleaning work
performed outdoors, often near or in roadways. That gives this actor
two independent hazard-scope dimensions: **vehicle-traffic hazard**
(working near or in moving traffic) and **outdoor weather/terrain
exposure** (weather conditions, uneven or hazardous terrain). Both
dimensions always escalate a flagged safety concern to a human — the
governor never auto-commits a safety concern regardless of confidence.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot
performs the physical domain work**. Here a route scheduling/logistics
coordination robot performs crew scheduling, cleaning-log/progress-
record logging and cleaning-supplies/equipment procurement
coordination for a street/site-cleaning crew, under an actor that
proposes actions and an independent **StreetCleaningGovernor** that
gates them. The governor never dispatches hardware itself, never
performs sweeping/cleaning work on the route itself, and never
finalizes a cleaning-work-execution decision or a route-safety-
clearance decision, and never overrides a route safety supervisor's
judgment; `:high`/`:safety-critical` actions (such as a flagged
vehicle-traffic-hazard/weather-terrain-exposure-hazard/equipment-
condition concern, or an above-threshold supply order) require human
sign-off. **This actor coordinates ROUTE SCHEDULING/LOGISTICS ONLY —
it never performs sweeping/cleaning work itself and never makes a
route-safety-clearance decision itself.**

## Core Contract

```text
worker roster + route registration + safety-reporting policy
        |
        v
Sweeper and Related Labourer Advisor -> StreetCleaningGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a cleaning-work-execution decision, finalize a route-safety-
clearance decision (e.g. declaring a route cleared for safety),
override a route safety supervisor's judgment, suppress an operating
record, or disclose sensitive data without governor approval and audit
evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `9613`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
