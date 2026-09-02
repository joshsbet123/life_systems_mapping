# Life Milestones Systems Map — Codebook v0.2

Draft date: 2026-09-02. Supersedes v0.1 (which covered `milestone_variants.csv` only). All scores remain author assertions; see `evidence_status` on every row.

## What changed from v0.1

- Added the **need** layer: jobs to be done at milestone level, each with a start trigger and an end state. A need must be satisfiable by more than one touchpoint but not by every touchpoint.
- Added **conditions**: things a milestone causes that have no end state (grief, financial vulnerability). Kept separate so they are not forced into the need shape.
- Added the **touchpoint** layer beneath needs. Sector is now an attribute of the touchpoint, not a spine. `actor_type` and `current_providers` are what carry the cross-sector logic.
- Added three **edge tables**. Sector footprint lives in edges, as decided in v0.1.
- `milestone_variants.csv` is unchanged.

## Tables

| file | one row per | key |
|---|---|---|
| `milestone_variants.csv` | milestone variant | `variant_id` |
| `needs.csv` | need | `need_id` |
| `conditions.csv` | condition | `condition_id` |
| `touchpoints.csv` | touchpoint (one way of meeting one need) | `touchpoint_id` |
| `edges_variant_need.csv` | variant → need link | `edge_id` |
| `edges_need_touchpoint.csv` | need → touchpoint link | `edge_id` |
| `edges_milestone_milestone.csv` | variant → variant link | `edge_id` |

Every foreign key must resolve: `touchpoints.need_id` → `needs.need_id`; edge `from_variant`/`to_variant` → `milestone_variants.variant_id`; edge `to_need`/`need_id` → `needs.need_id`; edge `touchpoint_id` → `touchpoints.touchpoint_id`.

## Shared vocabularies

- `visibility` / `end_state_visibility`: `registered` / `partial` / `invisible` (as v0.1).
- `evidence_status`: `asserted` / `observed_panel` / `validated_aggregate` (as v0.1).
- `actor_type`: `state` / `regulated_firm` / `professional_service` / `employer` / `third_sector` / `informal`. Sets which duties apply (statutory, Consumer Duty, professional, none).
- `sector`: `government` / `energy_utilities` / `housing_property` / `banking_credit` / `pensions_investments` / `insurance` / `legal` / `employment` / `health_care` / `telecoms_media` / `transport_vehicle` / `funeral_death_services`. Grouping label on touchpoints only; expected to blur and free to change.
- `edge_type` (need → touchpoint): `obligation` (must be done) / `choice` (optional, discretionary) / `consequence` (happens to the person; no action required to open it).
- `edge_type` (variant → variant): `triggers` / `enables` / `precedes` / `coincides` / `precludes`.
- `trigger` (variant → need): `always` / `conditional`. If `conditional`, `condition` must be non-empty.

## needs.csv

| column | definition |
|---|---|
| `need_id` | snake_case key. Milestone-independent: the same need may be triggered by several milestones. |
| `definition` | Must carry inclusion and exclusion criteria. |
| `start_trigger` | What opens the need. |
| `end_state` | The observable state that means the need is satisfied. If it cannot be written, the row belongs in `conditions.csv`. |
| `end_state_visibility` | Whether the end state leaves a record. Determines whether completion is measurable in public data. |
| `bounded` | `yes` / `no`. `no` is permitted temporarily; a persistent `no` means the row is a condition. |

## touchpoints.csv

| column | definition |
|---|---|
| `touchpoint_id` | `actor.action` style key. |
| `need_id` | The need this touchpoint serves. One need per touchpoint in v0.2; revisit if a touchpoint clearly serves two. |
| `actor_type` | Kind of counterparty (see vocabulary). |
| `sector` | Grouping label for the actor. |
| `current_providers` | Free text; empirical and time-varying. To be replaced by a sourced provider register with its own refresh cadence. |
| `mandatory` | `yes` / `no` / `conditional`. Whether the person must engage with this touchpoint at all. |

## Edge tables

- `lag_months`: months from the milestone trigger date to the need or touchpoint opening. Negative values are pre-event (only possible where `lead_time` is not `shock`).
- `window_months`: how long the need or edge typically stays open after it opens.
- `should_but_dont` / `do_but_shouldnt`: the two behaviour gaps at this touchpoint. Free text in v0.2; to become a behaviours table once patterns repeat.
- `driver_or_barrier`: the behavioural mechanism claimed. Free text in v0.2.

## Queries the tables are designed to answer

1. **Need sequence for a milestone**: `edges_variant_need` filtered on `from_variant`, ordered by `lag_months`.
2. **Touchpoint sequence by actor**: join 1 to `touchpoints` via `need_id`; group by `actor_type` or `sector`.
3. **Footprint view**: filter 2 to a list of `current_providers` or sectors; the lowest `lag_months` is the leading touchpoint.
4. **Unserved white space**: needs with no touchpoint whose `actor_type` is `regulated_firm` or `professional_service`.
5. **Unfinished white space**: needs where a `should_but_dont` exists and `end_state_visibility` is `partial` or `invisible`.
6. **Inter-milestone paths**: `edges_milestone_milestone` walked from any variant.

## Population status

- Bereavement: `parent_anticipated` and `partner_anticipated` fully populated for variant → need. `parent_sudden`, `partner_sudden` and `child` carry only a placeholder row and must be populated explicitly before validation (no inheritance between variants, by design).
- First property purchase: variants exist; needs and touchpoints not yet populated. Next milestone to build.

## Ambiguities carried forward from v0.1

1–5 as listed in v0.1 remain open.

## New ambiguities

6. `settle_household_accounts` differs materially between parent (close) and partner (transfer) variants but is one need. Either split the need or accept that behaviour differences live on the variant → need edge.
7. `current_providers` mixes generic categories ("energy suppliers") with named firms ("Octopus Legacy"). Needs a rule before it becomes a query field.
8. `driver_or_barrier` is unstructured. A closed vocabulary (e.g. COM-B or a friction/salience/norms set) would make it queryable.
