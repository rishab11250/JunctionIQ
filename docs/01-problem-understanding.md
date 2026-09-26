# Problem Understanding & Analysis

## 1. Problem Statement

Urban road networks worldwide operate near peak capacity during commuter rush hours. Intersections represent the primary choke points in these networks, dictating the flow rate and delay across entire metropolitan corridors. Despite significant advancements in software, computation, and computer vision over the past two decades, the vast majority of physical traffic signals still operate on rigid, predetermined timing cycles that fail to adapt to real-time vehicular demand.

The fundamental problem can be stated as follows:
> **Conventional traffic control allocates signal green time based on historical averages and pre-programmed static schedules rather than real-time vehicular demand, causing severe spatial and temporal mismatches that amplify congestion, driver delay, and unnecessary emissions.**

---

## 2. Current Situation: The Status Quo

In contemporary municipal traffic operations, signal timing typically falls into one of two legacy categories:

1. **Fixed-Time Signal Control:**  
   Signal phases transition on static timers (e.g., 30 seconds green for North-South, 30 seconds green for East-West), running unvaried 24/7 or switching between a few coarse time-of-day presets (e.g., "morning rush", "midday", "evening rush").
2. **Inductive Loop / In-Pavement Detection:**  
   Pavement-embedded magnetic loops detect vehicle presence directly above them to trigger actuation (e.g., extending green time if a car is waiting). However, in-pavement sensors suffer from high installation and civil maintenance costs, degrade quickly under road wear, and only detect presence at a single discrete point—offering zero spatial visibility into queue depth, vehicle classification, or approach density.

---

## 3. Why Fixed-Time Signals Are Limited

The fundamental flaw of fixed-time control is its inability to respond to asymmetrical demand. Consider a typical four-way intersection scenario during an uneven commuter surge:

| Lane Approach | Observed Vehicles | Static Green Allocation | Actual Demand Requirement | Resulting Failure Mode |
| :--- | :--- | :--- | :--- | :--- |
| **Lane A (North)** | 32 vehicles | 30 seconds | High Green Demand (~45s+) | Incomplete queue clearance; cycle failure |
| **Lane B (South)** | 7 vehicles | 30 seconds | Low Green Demand (~12s) | Green light wasted on empty asphalt |
| **Lane C (East)** | 11 vehicles | 30 seconds | Moderate Green Demand (~16s) | Minor inefficiency; underutilized phase |
| **Lane D (West)** | 26 vehicles | 30 seconds | High Green Demand (~38s) | Spilling queues; delayed clearance |

Under fixed timing, **Lane B receives 30 seconds despite clearing its 7 cars in under 12 seconds**, leaving 18 seconds of empty asphalt while **Lane A suffers spillover queues** that compound with every subsequent cycle.

---

## 4. Traffic Demand Variability

Traffic demand is stochastic and dynamic. It varies due to:
- **Diurnal Commuting Shifts:** Morning inbound surges versus evening outbound surges.
- **Micro-Bursts:** Platoon arrivals caused by upstream signal releases or highway off-ramps.
- **Special Events & Weather:** Inclement weather, public events, or construction bottlenecks that deviate drastically from static historical averages.

Static timing plans cannot respond to these transient spikes, converting minor volume surges into persistent gridlock.

---

## 5. Queue Buildup Mechanics

When the arrival rate ($\lambda$) at an intersection approach exceeds the service capacity ($\mu$) provided by the green allocation:
$$\Delta Q = (\lambda - \mu) \cdot \Delta t$$
The queue grows continuously. In fixed-time systems, once a queue spills past the storage bay or blocks upstream access roads, it creates a cascading gridlock phenomenon known as *blocking back*, paralyzing adjacent feeder streets.

---

## 6. Vehicle Idle Time and Environmental Impact

Every second a vehicle stands motionless in a stationary queue:
- The engine operates in inefficient idle mode, consuming fuel without producing forward displacement.
- Brake and tire wear elevate ambient particulate matter.
- Elevated cumulative idle time directly translates into avoidable fuel consumption and carbon dioxide ($\text{CO}_2$) emissions.

By dynamically reducing unnecessary stopped delay, intersections can achieve immediate fuel and emissions improvements without requiring modifications to vehicular drivetrains.

---

## 7. Existing Infrastructure Opportunity: Optical Traffic Cameras

Municipalities and transportation departments have already invested millions of dollars installing CCTV and IP optical traffic cameras at major intersections for manual surveillance and incident monitoring.

However, these cameras remain **passive observation endpoints**—their video streams are monitored intermittently by human operators in traffic management centers, with zero automated feature extraction.

**The Opportunity:**  
By deploying a software intelligence layer on top of these existing RTSP camera feeds, cities can convert passive video feeds into rich digital traffic telemetry without digging up asphalt or deploying proprietary in-pavement hardware.

---

## 8. Problem Constraints & Engineering Boundaries

To formulate an achievable and rigorous engineering proposal, this project adopts the following explicit constraints:
1. **Software-Only Intelligence:** The system does not interface directly with physical, mission-critical traffic signal controllers (NEMA TS2 / 170 / 2070 cabinets) in this phase.
2. **Simulation-Driven Validation:** Because live intersection experimentation on public streets presents severe safety and municipal regulatory liabilities, all adaptive strategies are validated within a controlled, identical simulation environment.
3. **Deterministic Logic Over Black Boxes:** Safety-critical infrastructure requires verifiable, bounded timing logic. The system deliberately rejects opaque, unconstrained reinforcement learning models for the MVP in favor of explainable rule-based heuristics.

---

## 9. MVP Problem Definition

For the Round 1 proposal, the problem is scoped to:
> **Design and specify a software platform capable of ingesting video feeds from a single four-way intersection, extracting lane-by-lane vehicle counts and queue lengths, calculating adaptive phase durations bounded by safety minimums, and proving comparative efficiency against fixed-time baselines within a controlled simulation loop.**

---

## 10. Success Criteria

The conceptual success of the JunctionIQ architecture will be judged on:
1. **Perception Reliability:** Accurate delineation of vehicle bounding boxes across four standard vehicle categories (cars, buses, trucks, motorcycles) mapped to discrete lane polygons.
2. **Deterministic Responsiveness:** Allocation of green phase times that dynamically favor heavily queued approaches while guaranteeing absolute minimum pedestrian/safety green durations to minor approaches.
3. **Simulation Demonstrability:** Demonstrating measurable improvements across average waiting time, queue lengths, and vehicle throughput under identical simulated traffic arrival conditions.
4. **Clean Decoupled Architecture:** Clean API contracts, modular subsystems, and complete documentation enabling independent scalability.

*(Note: Problem analysis is derived strictly from the domain problem framing of urban signalization and does not cite non-verified external statistical claims).*
