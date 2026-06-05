# Common Procurement and Planning Guide

This shared guide replaces repeated procurement, resource-planning, and timeline boilerplate across the AM/ architecture files.

## When to use this guide

Use this document whenever you need a reusable framework for:

- procurement and budget planning
- staffing and capacity sizing
- rollout sequencing and implementation timelines
- common vendor, support, and lifecycle considerations

## Procurement framework

### 1. Start with business constraints

Before selecting a platform, capture:

- target workloads and expected growth
- recovery objectives and support windows
- compliance constraints
- rack, power, cooling, and WAN limitations
- preferred vendors and purchasing channels

### 2. Build a bill-of-materials checklist

A repeatable BOM usually includes:

- primary compute, network, or storage platform components
- redundant power supplies and rails
- optics, DACs, patch leads, and spare transceivers
- out-of-band management access
- licensing, subscriptions, and support contracts
- at least one spare for the most failure-prone field-replaceable items

### 3. Budget categories to estimate

| Category | Typical examples |
| --- | --- |
| Hardware | servers, switches, storage shelves, appliances |
| Software | hypervisor, backup, monitoring, security, support subscriptions |
| Facilities | rack space, power, cooling, remote hands |
| Network | circuits, optics, patching, firewalls, load balancers |
| Operations | monitoring, backup, logging, change-management overhead |
| People | engineering time, project management, training, after-hours support |

### 4. Procurement guardrails

- Standardize on as few hardware families as practical.
- Prefer identical node builds inside the same cluster or platform tier.
- Confirm hardware compatibility lists and firmware baselines before purchase.
- Budget for maintenance accessories, not only the primary platform.
- Include lead-time risk in the rollout plan.
- Buy support coverage that matches workload criticality.

## Resource planning

### Capacity planning principles

1. Size for steady-state demand plus failure tolerance.
2. Reserve headroom for maintenance, growth, and burst events.
3. Track CPU, RAM, storage, network, and operational concurrency separately.
4. Align staffing plans with platform complexity, not just node count.

### Core sizing questions

- What is the average and peak workload?
- What failure event must the environment survive?
- How much capacity must remain after losing one node, one switch, or one zone?
- What growth buffer is needed for 12 to 18 months?
- Which teams own day-2 operations and on-call support?

### Common planning heuristics

| Area | Planning heuristic |
| --- | --- |
| Compute | keep spare headroom for maintenance and one realistic failure domain |
| Memory | avoid planning right up to steady-state limits |
| Storage | include usable capacity, replication overhead, snapshots, and rebuild space |
| Network | reserve port count, IP space, and uplink growth margin |
| Operations | account for logging, backup, monitoring, and patch windows from day one |

### Staffing guidance

- Small environments still need clear primary and backup owners.
- Shared platforms usually need infra, security, and operations representation.
- Complex stacks need training time included in the delivery plan.
- If 24x7 support is expected, plan for handoff, alerting, and escalation coverage early.

## Planning and timeline template

### Recommended delivery phases

| Phase | Typical outputs |
| --- | --- |
| Discovery | inventory, dependencies, risks, stakeholder map |
| Design | architecture decisions, BOM, addressing, standards, rollback plan |
| Procurement | purchase orders, lead-time tracking, receiving checklist |
| Build | base platform, automation, validation evidence |
| Pilot | controlled test workload, performance baseline, operational rehearsal |
| Rollout | phased production deployment or migration waves |
| Stabilization | runbook updates, monitoring tuning, handoff |

### Sample six-week implementation rhythm

| Week | Goal |
| --- | --- |
| 1 | finalize scope, dependencies, BOM, and approvals |
| 2 | place orders, prepare addresses, naming, and standards |
| 3 | rack, cable, bootstrap, and validate base connectivity |
| 4 | build shared services and automation baselines |
| 5 | pilot workload deployment and rollback rehearsal |
| 6 | production rollout and stabilization |

### Timeline risks to track

- long hardware lead times
- missing dependencies between layers
- undocumented firewall or DNS requirements
- under-sized maintenance windows
- incomplete rollback plans
- training gaps before go-live

## Reusable checklists

### Procurement readiness checklist

- scope approved
- budget approved
- BOM reviewed
- vendor quote validated
- support tiers confirmed
- lead times captured
- spare strategy defined
- firmware and compatibility notes recorded

### Capacity readiness checklist

- baseline workload measured
- failure-domain assumption documented
- growth buffer agreed
- storage overhead included
- network port/IP headroom reserved
- staffing owners assigned

### Rollout readiness checklist

- prerequisites complete
- change windows approved
- validation plan written
- rollback plan documented
- communications plan ready
- monitoring and backup enabled
- handoff owner assigned

## Applying this shared guide

Each AM file keeps its design-specific architecture, advanced configuration, runbook, and cross-reference sections. Use this guide for the repeatable commercial and project-planning material, then return to the layer-specific file for implementation detail.
