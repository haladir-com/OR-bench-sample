# OR-Bench Sample

A curated sample of operations research optimization tasks for evaluating LLM constraint satisfaction and reasoning capabilities.

## Overview

This dataset contains expert-level OR tasks across 10 classic operations research domains, each requiring models to generate feasible solutions that satisfy complex interacting constraints. A sample is included.

## Domains

| Domain | Description |
|--------|-------------|
| **facility_location** | Decide which facilities to open and assign customers to minimize cost while respecting capacity, distance, and logical constraints (exclusions, conditionals) |
| **vehicle_routing** | Route vehicles to serve customers within time windows, respecting capacity, max distance, incompatibility, and must-visit constraints |
| **production_mix** | Determine product quantities to maximize profit given resource limits, minimum/maximum production bounds, and ratio constraints |
| **project_planning** | Schedule tasks over a horizon with precedence, resource capacity, and deadline constraints |
| **order_fulfillment** | Fulfill customer orders from stock, vendors, or manufacturing within deadlines and budgets |
| **portfolio** | Allocate budget across assets to optimize return/risk within sector exposure and diversification constraints |
| **crew_assignment** | Assign certified workers to tasks respecting max hours, certifications, and workload balance |
| **job_shop** | Schedule operations on machines with precedence and capability constraints to minimize makespan |
| **shift_scheduling** | Assign employees to shifts based on skills, availability, and staffing requirements |
| **bin_packing** | Pack items into bins respecting weight/volume capacities and compatibility constraints |

## Example Prompt

Here's what a model actually sees for a facility location task:

```
You are solving a facility location optimization problem.

SCENARIO: Automotive dealership network: RoadForge Auto Group must decide which
regional Parts Distribution & Reconditioning Centers to open to supply OEM parts,
accessories, and certified pre-owned (CPO) reconditioning kits to a network of
dealerships. Each dealership must be served by exactly one open center within 50
distance units.

=== CUSTOMERS ===
- RoadForge Downtown Motors (id: c1): at (12.0, 18.0), demand=70
- RoadForge Riverfront Auto (id: c2): at (18.0, 12.0), demand=60
- RoadForge Northpoint Hyundai (id: c3): at (25.0, 22.0), demand=55
- RoadForge Lakeside Ford (id: c4): at (20.0, 35.0), demand=65
- RoadForge Cedar Valley Kia (id: c5): at (8.0, 40.0), demand=50
... (17 dealerships total)

=== POTENTIAL FACILITIES ===
- GearPoint Central PDC (id: f1): at (16.0, 16.0), capacity=500, fixed_cost=$42000, var_cost=$3.6/unit
- TorqueTown West PDC (id: f2): at (-15.0, 15.0), capacity=280, fixed_cost=$18000, var_cost=$3.1/unit
- AxleEdge North Center (id: f3): at (10.0, 45.0), capacity=240, fixed_cost=$16000, var_cost=$3.4/unit
... (9 facilities total)

=== SCENARIO CONSTRAINTS ===
- Maximum facilities to open: 5
- Maximum distance from customer to facility: 50.0

=== ADDITIONAL CONSTRAINTS ===
- GearPoint Central PDC must remain operational (existing OEM contract)
- Each dealership must be served by a single fulfillment center
- Open at least 3 facilities for redundancy
- TorqueTown West and WrenchWorks Northwest cannot both operate (union jurisdiction)
- If ChromeCove Airport is opened, PistonPort East must also be opened (linehaul dependency)
- RecallReady Overflow Yard must remain closed (hazmat permit denied)

=== YOUR TASK ===
Minimize total cost (fixed costs + variable costs) while serving every dealership
within max_distance, respecting capacity limits, and satisfying all logical constraints.
```

The model must return structured JSON:
```json
{
  "reasoning": "Analysis of the problem and solution approach...",
  "open_facilities": ["f1", "f2", "f4", "f5"],
  "assignments": [
    {"customer_id": "c1", "facility_id": "f1", "amount": 70},
    {"customer_id": "c2", "facility_id": "f1", "amount": 60},
    ...
  ],
  "total_cost": 98500.0
}
```

## Task Structure

Each task in `tasks.json` contains:

```json
{
  "id": "facility_location_7ce62b6b",
  "difficulty": "expert",
  "scenario": {
    "domain": "facility_location",
    "description": "...",
    // Domain-specific entities (customers, facilities, vehicles, etc.)
  },
  "constraints": [
    {
      "description": "Human-readable constraint description",
      "_spec": {"fn": "constraint_type", "args": [...]}
    }
  ],
  "instruction": "Optimization objective and constraint summary",
  "feasible": true,
  "solution": { /* Ground-truth optimal/near-optimal solution */ }
}
```

## Constraint Types by Domain

### Facility Location
- `must_open` - Facility must be opened
- `single_source` - Each customer served by exactly one facility
- `min_facilities` / `max_facilities` - Bounds on open facilities
- `exclusion` - Mutually exclusive facilities
- `conditional` - If A opens, B must open

### Vehicle Routing
- `time_window` - Customer must be visited within time range
- `max_stops` / `max_distance` - Route limits
- `must_visit` - Customer must be on specific vehicle
- `incompatible` - Customers cannot share a route

### Production Mix
- `min_production` / `max_production` - Per-product bounds
- `ratio` - Product quantity ratios
- `resource_limit` - Resource consumption limits

### (See full schema for other domains)

## Evaluation Criteria

Models are evaluated on:

### Feasibility
A solution is **feasible** if it satisfies all constraints (capacity limits, time windows, logical constraints, etc.). Binary pass/fail.

### Optimality Gap
Measures how close the proposed objective is to the ground-truth optimal solution:

```
optimality_gap = (proposed_objective - optimal_objective) / optimal_objective
```

| Domain | Objective | Direction |
|--------|-----------|-----------|
| facility_location | total_cost | minimize |
| vehicle_routing | total_distance | minimize |
| production_mix | total_profit | maximize |
| project_planning | project_duration | minimize |
| order_fulfillment | total_cost | minimize |
| portfolio | expected_return | maximize |
| crew_assignment | total_cost | minimize |
| job_shop | makespan | minimize |
| shift_scheduling | total_cost | minimize |
| bin_packing | bins_used | minimize |

A solution is considered **optimal** if:
1. It is feasible (all constraints satisfied)
2. `|optimality_gap| < 0.001` (within 0.1% of ground-truth)

**Reporting**: Both feasibility and optimality rates are calculated as a percentage of total tasks (not optimal/feasible). A task that is infeasible cannot be optimal.

Ground-truth solutions are computed using OR-Tools solvers (SCIP for MIP, CP-SAT for scheduling).

## Results

| Model | Feasibility% | Optimality% | % Optimal |
|-------|-------------|-------------|-----------|
| openai/gpt-5.2-pro | 65.3% | 95.2% | 30.7% |
| openai/gpt-5.2 | 64.0% | 93.8% | 21.0% |
| gemini-3-pro-preview | 44.5% | 94.8% | 8.5% |
| openai/o4-mini | 50.5% | 89.0% | 8.5% |
| anthropic/claude-opus-4.6 | 49.0% | 96.0% | 21.5% |
| anthropic/claude-opus-4.5 | 50.5% | 94.0% | 12.5% |
