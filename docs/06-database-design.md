# Database & Persistence Layer Design

## 1. Storage Strategy Overview

JunctionIQ decouples the operational execution of traffic optimization from physical database dependencies. For the Round 1 proposal and initial MVP, the platform utilizes **in-memory data structures and static JSON configurations**.

To support scalable municipal deployments in future project phases, a comprehensive **MongoDB Document Database schema** has been designed.

> [!IMPORTANT]
> The database schemas and collections documented below represent a **PROPOSED FUTURE PERSISTENCE ARCHITECTURE**. The MVP relies on structured in-memory state management and JSON schemas.

---

## 2. MVP In-Memory State Model

During the single-junction MVP, the backend maintains operational state using high-performance in-memory collections:
- `activeStateMap`: A hash map storing the latest `TrafficState` keyed by `intersectionId`.
- `recentSnapshotBuffer`: A fixed-size rolling circular buffer storing the trailing 120 snapshots (1 hour at 30-second cycles) per intersection for instant dashboard telemetry queries.
- `activePlanMap`: A map holding the current active `SignalPlan` and remaining phase seconds.

This design achieves zero I/O latency, zero external database setup prerequisites, and deterministic execution for hackathon evaluation.

---

## 3. Future MongoDB Document Schemas

In future production deployments, MongoDB is proposed as the primary datastore due to its native JSON document model, flexible schema evolution, robust time-series support, and geospatial indexing capabilities.

### 3.1 Collection 1: `intersections`
Stores physical intersection topography, geographic coordinates, lane designations, and safety threshold boundaries.

```json
{
  "_id": "intersection-01",
  "name": "Market St & 5th Ave",
  "location": {
    "type": "Point",
    "coordinates": [-122.4075, 37.7842]
  },
  "lanes": [
    {
      "laneId": "A",
      "approachDirection": "NORTH",
      "capacityVehicles": 45,
      "priorityWeight": 1.0,
      "lengthMeters": 150.0
    },
    {
      "laneId": "B",
      "approachDirection": "SOUTH",
      "capacityVehicles": 45,
      "priorityWeight": 1.0,
      "lengthMeters": 150.0
    },
    {
      "laneId": "C",
      "approachDirection": "EAST",
      "capacityVehicles": 30,
      "priorityWeight": 1.0,
      "lengthMeters": 100.0
    },
    {
      "laneId": "D",
      "approachDirection": "WEST",
      "capacityVehicles": 30,
      "priorityWeight": 1.0,
      "lengthMeters": 100.0
    }
  ],
  "signalConfiguration": {
    "minGreenTimeSeconds": 10,
    "maxGreenTimeSeconds": 75,
    "yellowTimeSeconds": 4,
    "allRedTimeSeconds": 2,
    "defaultCycleDurationSeconds": 120
  },
  "status": "ACTIVE",
  "createdAt": "2026-09-26T00:00:00.000Z",
  "updatedAt": "2026-09-26T00:00:00.000Z"
}
```

### 3.2 Collection 2: `traffic_snapshots`
High-frequency time-series telemetry captures emitted by roadside vision inference nodes.

```json
{
  "_id": "snap-68f4b102",
  "intersectionId": "intersection-01",
  "timestamp": "2026-09-26T09:30:00.000Z",
  "sensorHealth": {
    "cameraStatus": "ONLINE",
    "fps": 29.8,
    "frameDropRate": 0.002
  },
  "lanes": [
    {
      "laneId": "A",
      "vehicleCount": 32,
      "density": 0.78,
      "queueLength": 18,
      "averageSpeedKmph": 12.4,
      "vehicleClassification": { "cars": 24, "buses": 3, "trucks": 1, "motorcycles": 4 }
    },
    {
      "laneId": "B",
      "vehicleCount": 7,
      "density": 0.22,
      "queueLength": 3,
      "averageSpeedKmph": 38.6,
      "vehicleClassification": { "cars": 5, "buses": 0, "trucks": 0, "motorcycles": 2 }
    }
  ]
}
```

### 3.3 Collection 3: `signal_plans`
Durable historical record of every deterministic timing allocation produced by the optimization engine.

```json
{
  "_id": "plan-adp-20260926-093005",
  "intersectionId": "intersection-01",
  "timestamp": "2026-09-26T09:30:05.000Z",
  "strategy": "ADAPTIVE_RULE_BASED",
  "cycleDurationSeconds": 120,
  "rationale": "High congestion on North approach (Lane A: 32 veh). Scaled green time to 46s while enforcing 14s minimum on South approach.",
  "phases": [
    {
      "phaseIndex": 1,
      "activeLanes": ["A"],
      "greenTimeSeconds": 46,
      "yellowTimeSeconds": 4,
      "allRedTimeSeconds": 2,
      "demandScore": 28.4
    },
    {
      "phaseIndex": 2,
      "activeLanes": ["B"],
      "greenTimeSeconds": 14,
      "yellowTimeSeconds": 4,
      "allRedTimeSeconds": 2,
      "demandScore": 7.1
    }
  ]
}
```

### 3.4 Collection 4: `simulation_runs`
Audit and benchmark execution log comparing synthetic baseline models against adaptive logic.

```json
{
  "_id": "sim-run-20260926-101",
  "intersectionId": "intersection-01",
  "scenario": "Morning_Peak_Four_Way_Asymmetric",
  "strategy": "COMPARATIVE_DUAL_RUN",
  "simulationDurationSeconds": 1800,
  "startedAt": "2026-09-26T09:30:10.000Z",
  "endedAt": "2026-09-26T09:30:12.000Z",
  "status": "COMPLETED",
  "fixedPlanBaselineId": "plan-fix-default-120",
  "adaptivePlanId": "plan-adp-20260926-093005"
}
```

### 3.5 Collection 5: `metrics`
Stores aggregated performance indicators linked to specific simulation executions or hourly operational rollups.

```json
{
  "_id": "met-run-101",
  "simulationId": "sim-run-20260926-101",
  "intersectionId": "intersection-01",
  "baselineMetrics": {
    "averageWaitTimeSec": 72.4,
    "averageQueueLength": 31.2,
    "throughputPerHour": 840.0,
    "totalIdleTimeSeconds": 48200.0,
    "estimatedFuelLiters": 124.5,
    "estimatedEmissionsKgCO2": 286.3
  },
  "adaptiveMetrics": {
    "averageWaitTimeSec": 43.1,
    "averageQueueLength": 18.5,
    "throughputPerHour": 1120.0,
    "totalIdleTimeSeconds": 29100.0,
    "estimatedFuelLiters": 75.2,
    "estimatedEmissionsKgCO2": 172.9
  },
  "comparativeDeltaPercent": {
    "waitTime": -40.47,
    "queueLength": -40.71,
    "throughput": 33.33,
    "idleTime": -39.63
  }
}
```

---

## 4. Indexing & Query Optimization Strategy

| Collection | Proposed Index Pattern | Optimization Target |
| :--- | :--- | :--- |
| `intersections` | `{ "location": "2dsphere" }` | Geospatial boundary queries across municipal corridors |
| `traffic_snapshots` | `{ "intersectionId": 1, "timestamp": -1 }` | Fast retrieval of latest trailing time-series windows |
| `traffic_snapshots` | `{ "timestamp": 1 }` (TTL: 7 days) | Automated background garbage collection of high-volume raw telemetry |
| `signal_plans` | `{ "intersectionId": 1, "timestamp": -1 }` | Auditing historical plan transitions and schedule verifications |
| `simulation_runs` | `{ "intersectionId": 1, "status": 1 }` | Filtering completed simulation benchmark experiments |

---

## 5. Retention Policies & Future Data Growth Model

- **Telemetry Volume Projection:** At 1 snapshot every 2 seconds, a single 4-way intersection produces approximately 43,200 documents (~15 MB uncompressed JSON) daily.
- **Automated Data Downsampling:**
  - **Days 0–7:** High-frequency raw snapshots retained via MongoDB TTL index for live operational inspection and anomaly debugging.
  - **Days 8+:** Automated cron jobs aggregate raw snapshots into 15-minute statistical summaries (min queue, max queue, average throughput) stored in an `analytics_hourly` collection, reducing long-term storage consumption by 98%.
