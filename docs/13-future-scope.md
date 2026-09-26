# Future Scope & Long-Term Roadmap

## 1. Evolutionary Roadmap

The JunctionIQ architecture has been intentionally designed with modular interfaces to facilitate progressive enhancements. This document outlines ten future research and engineering trajectories.

> [!NOTE]
> Every capability detailed below represents a **FUTURE RESEARCH OR ENGINEERING TARGET**. None of these features are claimed as implemented within the current Round 1 proposal.

---

## 2. Detailed Future Capabilities Matrix

### 2.1 Multi-Intersection Corridor Coordination
- **Current Limitation:** The MVP optimizes a single isolated four-way junction without awareness of adjacent intersections.
- **Future Capability:** Implement decentralized offset coordination ("Green Wave" corridor synchronization) between sequential intersections along an arterial roadway.
- **Potential Benefit:** Minimizes frequent stop-and-go acceleration cycles across commuter corridors, boosting corridor-level vehicular throughput.

### 2.2 Predictive Traffic Forecasting
- **Current Limitation:** Signal allocations react strictly to instantaneous vehicle presence and short-term trailing queues.
- **Future Capability:** Integrate lightweight temporal forecasting models (e.g. LSTM or Temporal Convolutional Networks) to predict platoon arrivals 2–5 minutes in advance based on upstream corridor trends.
- **Potential Benefit:** Allows signal plans to preemptively transition before major vehicle platoons reach the stop line, reducing deceleration delay.

### 2.3 Emergency Vehicle Preemption (EVP)
- **Current Limitation:** All vehicular classes are optimized based on standard passenger car equivalent weights; emergency vehicles receive no priority.
- **Future Capability:** Train fine-grained computer vision classifiers to identify emergency vehicles (ambulances, fire engines, police cruisers) by visual livery and flashing beacons, triggering an immediate safety clearance phase.
- **Potential Benefit:** Decreases emergency service response times and improves junction transit safety during urgent dispatches.

### 2.4 Transit & Public Bus Priority (TSP)
- **Current Limitation:** Public buses are treated as weighted vehicles ($2.5\times$ car equivalent) but do not hold scheduled priority.
- **Future Capability:** Ingest General Transit Feed Specification Realtime (GTFS-RT) feeds or visual bus identifiers to extend green phases slightly when a transit vehicle is approaching behind schedule.
- **Potential Benefit:** Promotes high-capacity public transit efficiency and improves municipal bus schedule adherence.

### 2.5 Pedestrian & Active Mobility Sensing
- **Current Limitation:** Pedestrian safety is maintained via static conservative minimum green allocations ($\ge 10\text{ seconds}$); pedestrian volume is not measured.
- **Future Capability:** Incorporate dedicated crosswalk ROI polygons detecting pedestrians, cyclists, and mobility scooters to dynamically extend crosswalk clearance times only when active vulnerable road users are present.
- **Potential Benefit:** Balances pedestrian crossing safety with vehicular flow, avoiding lengthy pedestrian phases when crosswalks are empty.

### 2.6 Incident & Anomaly Detection
- **Current Limitation:** The perception system tracks vehicle movement and counts but does not identify abnormal road conditions.
- **Future Capability:** Implement automated visual detection of vehicle breakdowns, collisions, stalled vehicles, or debris obstructing lane paths, immediately transmitting automated alerts to municipal traffic management centers.
- **Potential Benefit:** Drastically shortens emergency incident response times and prevents secondary collisions caused by unexpected lane blockages.

### 2.7 Historical Analytics & Municipal Heatmap Portal
- **Current Limitation:** MVP stores rolling in-memory snapshots for immediate telemetry queries.
- **Future Capability:** Deploy a scalable MongoDB Atlas time-series lake coupled with an automated ETL pipeline that compiles daily, weekly, and seasonal congestion heatmaps.
- **Potential Benefit:** Equips municipal urban planners with empirical data to justify road widening, transit lane allocation, or cycleway construction.

### 2.8 Roadside Edge Containerization
- **Current Limitation:** Vision service is structured as a standalone Python process running on a workstation.
- **Future Capability:** Package the vision perception pipeline into an optimized Docker/container image leveraging NVIDIA Jetson TensorRT or Intel OpenVINO runtime accelerators for turn-key deployment in roadside environmental cabinets.
- **Potential Benefit:** Enables zero-touch roadside deployment and eliminates continuous high-bandwidth video streaming across cellular uplinks.

### 2.9 Physical Traffic Controller Integration (NTCIP / NEMA TS2)
- **Current Limitation:** Platform generates software-only recommendations evaluated inside a discrete simulation twin.
- **Future Capability:** Develop an authenticated hardware adapter module communicating over the National Transportation Communications for ITS Protocol (NTCIP 1202) to interface with physical controller cabinets (NEMA TS2 / Type 2070).
- **Potential Benefit:** Transitions the software intelligence from advisory simulation mode into physical on-street actuated signal operations under municipal supervision.

### 2.10 Reinforcement Learning (RL) Research
- **Current Limitation:** Optimization uses an explainable, deterministic rule-based scoring formula.
- **Future Capability:** Conduct controlled academic research utilizing Deep Q-Networks (DQN) or Proximal Policy Optimization (PPO) in simulated SUMO environments with safety-bounded action spaces.
- **Potential Benefit:** Explores whether machine learning policies can uncover non-linear optimization strategies superior to rule-based heuristics under non-standard traffic regimes.
