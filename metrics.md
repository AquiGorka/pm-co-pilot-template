# PM Co-Pilot Metrics: Throughput, Quality, Time-to-Deliver

> Status: v1, work in progress. Schema and definitions will refine as the data accumulates.

Three performance indicators are tracked from the `prompts/` corpus: **throughput**, **quality**, and **time-to-deliver**. This document defines how each is computed so any external claim about the project's working speed and quality can be backed by real numbers.

The signal lives in the progress file frontmatter (`Status:`, `Type:`, `Started:`, `Done:`). See `prompts/README.md` for the schema and conventions.

## Definitions

### Throughput
**What it measures:** initiatives shipped. Distinct workstreams taken from "this is the goal" to "delivered".

**Formula:**
```
throughput = count(prompts where Status == "done" AND Type == "original")
```

**Notes.** Only `original` prompts count. A refinement extends an already-counted initiative; a fixup corrects an already-counted initiative; neither is a new shipped thing for throughput purposes. Counting all done prompts is more flattering and inflatable; counting originals is the honest, conservative number.

### Quality
**What it measures:** share of initiatives that landed without needing correction.

**Formula:**
```
quality = 1 - (count(done fixups) / count(done originals))
```

**Why this denominator.** Per-initiative quality. A single original that spawns multiple fixups drags the number down once and stays down; that is the honest signal. Using `total prompts` as the denominator inflates the score whenever an initiative spawns many extensions and is the more flattering, less honest framing.

**Range.** Can go negative if the rate of fixups exceeds the rate of originals (e.g., chains of repeated correction without new initiatives starting). Treat negative as "below zero quality" rather than clamping; the negative number is itself a signal that the IC threads are not landing first-pass and the spec / process needs work.

**Reading the number.**
- 1.0 = no original required a fixup. Clean delivery throughout.
- 0.5 = half the originals needed at least one corrective follow-up.
- 0.0 = every original needed a fixup, on average.

### Time-to-Deliver
**What it measures:** average wall-clock days from prompt sent to deliverable accepted, per prompt.

**Formula:**
```
time_to_deliver = average(Done - Started) across { prompts where Status == "done" }
```

**Notes.** Per-prompt average, not per-initiative. Calendar days, not active dev days. Blocked time counts. If a prompt takes 14 calendar days because it was blocked on a third-party dependency for 12 of them, that is 14 days of delivery time from the customer's perspective.

## When to use `done` vs `blocked`

`blocked` is for prompts that genuinely cannot progress AND have unresolved IC work. It is not for prompts whose IC work is finished and that are now waiting on an external response (an email reply, a platform review, a vendor approval, a regulatory body, a third-party committee).

When the IC's work on a prompt is finished and the only thing remaining is the async response, **mark the prompt `done`**. When the response lands and triggers new work, open a new prompt in the chain (`<topic>-N+1`), typed `refinement` if extending scope or `fixup` if correcting a defect.

This convention keeps `time_to_deliver` honest. A prompt that sits in `blocked` for weeks while waiting on a third party would otherwise drag the average upward without reflecting any actual development cycle.

## What's not measured (yet)

The three indicators above are first-order signals. They will not catch:

- **Spec quality.** A clean prompt that produces a clean delivery looks identical to a vague prompt that the IC happened to interpret well. The fixup rate captures part of this but not all.
- **Surface area.** A 10-line CSS tweak and a 600-line BLG document both count as 1 prompt. We are accepting this for now; bucket size normalization is a v2 problem.
- **Blocker-attributable delays.** If time-to-deliver creeps up because of an external dependency (e.g., a third-party platform's review queue, a vendor approval, a regulatory body), the metric punishes you for it. A future schema field could distinguish active dev days from blocked days; treat as a refinement. The async-blocker convention above partially mitigates this by closing prompts as `done` once the IC's work is finished.

## How to refresh the numbers

For now, manual. Walk `prompts/`, parse each progress file, compute the three numbers. A `scripts/kpi.py` to automate this is a planned refinement (TODO).

When the script lands, output should be a single line for headline use plus an audit table for sanity:

```
throughput=N  quality=0.XX  time_to_deliver=Y.Y days  (N done originals, M done refinements, K done fixups, L done out of P total)
```

## Schema migration note

Existing progress files in repos that pre-date this schema will be missing `Type:`, `Started:`, and `Done:`. Backfill is optional; the KPI calculation should treat missing fields as exclusion (not as `original` or `now`), so the metrics start producing useful numbers from the point the schema is adopted. Backfilling is a bookkeeping exercise that can happen incrementally.
