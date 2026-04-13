# Camera-Based Adaptive Traffic Simulation - Explanation for Faculty

## 1) What this project is about

This project demonstrates a **proof-of-concept traffic signal optimization simulation** for a four-way intersection.  
It compares two control strategies:

- **Fixed timing** (traditional approach): each direction gets the same pre-set green duration.
- **Adaptive timing** (proposed approach): green duration is adjusted according to live queue length.

The simulation goal is aligned with the blueprint:

- reduce waiting time (delay),
- reduce queue buildup,
- improve throughput (cars passed),
- prioritize fairness by cycling all directions.

## 2) Blueprint summary (from `daaSimulationBlueprint.pdf`)

The blueprint proposes a camera-based intelligent signal controller with this flow:

`Camera Input -> Vehicle/Pedestrian Detection -> Optimization Engine -> Signal Controller`

It formulates the problem as constrained optimization with:

- decision variable `g_i`: green time for each direction,
- objective: minimize weighted congestion/delay terms,
- constraints: min/max green bounds, cycle time bounds, queue clearance, pedestrian safety, and fairness.

It also mentions a **Genetic Algorithm (GA)** as an optimizer for finding better signal timing plans.

## 3) What the current code implements (`traffic_sim.py`)

The code is a **simulation-only prototype** (not yet camera-integrated hardware control). It models traffic queues and compares fixed vs adaptive control under identical random arrivals.

### Core parameters

- `SIMULATION_TIME = 150` ticks.
- `G_MIN = 3`, `G_MAX = 20` for adaptive green bounds.
- `SATURATION_FLOW = 2` cars can pass per green tick.
- Arrival rates by direction:
  - North `0.3`
  - South `0.3`
  - East `0.1`
  - West `0.8` (intentionally heavy demand lane)

### Intersection state tracked

Each controller instance tracks:

- queue lengths for North/South/East/West,
- cumulative waiting delay (`total_delay`),
- total cars passed (`cars_passed`),
- current green direction and remaining timer.

Two separate instances are created:

- `fixed = Intersection("FIXED")`
- `adaptive = Intersection("ADAPTIVE")`

Both receive the same arrivals each tick, ensuring a fair apples-to-apples comparison.

### Tick-by-tick simulation logic

For each time tick:

1. Generate random arrivals (`0`, `1`, or `2` cars per direction based on probability).
2. Add arrivals to both fixed and adaptive queues.
3. Accumulate waiting delay (sum of queue occupancy over time).
4. Let up to `SATURATION_FLOW` cars pass in the currently green direction.
5. Decrease green timer; if expired, switch to next direction.
6. Print a live side-by-side dashboard in terminal.

### Fixed controller behavior

- Rotates directions cyclically.
- Always assigns `10` seconds green after each switch.
- Does **not** react to queue size.

### Adaptive controller behavior

After switching to a direction:

- reads current queue length `q_len`,
- estimates needed green:
  - `calc_time = max(G_MIN, int(q_len / SATURATION_FLOW) + 2)`
- clips to max:
  - `green_timer = min(G_MAX, calc_time)`

So larger queues get longer green windows (within safety bounds), which is why the heavy West lane is handled better.

## 4) Performance metrics used in this simulation

At the end, the script reports:

- **Total Delay (Wait Time)**: cumulative queue-based waiting.
- **Total Throughput**: number of vehicles that successfully passed.
- **Improvement percentages** of adaptive over fixed.

This directly reflects the blueprint's optimization objectives.

## 5) How this maps to the blueprint

### Implemented from blueprint

- Four-way intersection model.
- Dynamic green allocation.
- Min/max green constraints (`G_MIN`, `G_MAX`).
- Queue-based optimization intent.
- Fixed vs optimized comparison.
- Delay and throughput evaluation.

### Not yet implemented (important to mention in viva/presentation)

- No real camera or computer vision pipeline yet.
- No explicit pedestrian detection/count in decision logic.
- No full constrained optimizer (GA/PSO/RL) loop in code.
- No multi-intersection coordination.

In short: this is a **validated algorithmic simulation prototype** that demonstrates adaptive control benefits and forms a base for full camera + optimization integration.

## 6) Does this follow the algorithmic approach from the PDF?

**Yes, partially (core approach is followed).**

The current code follows the **same optimization logic direction** as the blueprint:

- it uses real-time traffic state (`q_i` equivalent queue lengths),
- it computes adaptive `g_i` (green time) per direction,
- it enforces bounds (`G_MIN`, `G_MAX`) like min/max green constraints,
- it evaluates outcomes via delay reduction and throughput improvement.

However, it does **not yet implement the exact full algorithm stack** mentioned in the PDF:

- no Genetic Algorithm population/crossover/mutation loop,
- no explicit weighted objective function solver over all directions at once,
- no pedestrian term in optimization decision.

So, the project is best described as a **heuristic implementation of the PDF optimization model**, not a full GA-based implementation yet.

## 7) One-minute explanation you can present

"Our project simulates a smart traffic controller for a four-way junction. We compare traditional fixed-time signaling with an adaptive strategy that adjusts green time based on real-time queue lengths. In every simulation tick, both systems receive identical traffic, so comparison is fair. The adaptive controller computes green duration from queue demand while respecting minimum and maximum safety bounds. Results are evaluated using cumulative delay and throughput, and adaptive control consistently improves both, especially on heavy-load approaches like the West lane. This simulation validates the blueprint's core idea and is ready to be extended with real camera-based vehicle/pedestrian detection and advanced optimizers like Genetic Algorithms."

