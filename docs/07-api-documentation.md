# REST & WebSocket API Specification

## 1. Specification Overview

> [!NOTE]
> This document provides the **formal API interface specification** for the JunctionIQ platform. All implementations in later project phases must adhere to these contracts.

All REST endpoints return responses wrapped inside the standard `ApiResponseEnvelope`:
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:00.000Z",
  "requestId": "req-71829abc-43fe",
  "data": { ... },
  "error": null
}
```

---

## 2. REST API Endpoints

### 2.1 System Health
`GET /api/health`
- **Purpose:** Verifies operational readiness of backend services and memory utilization.
- **Request Body:** None
- **Query Parameters:** None
- **Status Codes:** `200 OK`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:00.000Z",
  "requestId": "req-98f24a1b",
  "data": {
    "status": "UP",
    "uptimeSeconds": 14205,
    "memoryUsageMb": 48.2,
    "version": "1.0.0-mvp"
  },
  "error": null
}
```

---

### 2.2 List Intersections
`GET /api/intersections`
- **Purpose:** Retrieves a list of all monitored intersection entities.
- **Request Body:** None
- **Query Parameters:** None
- **Status Codes:** `200 OK`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:00.000Z",
  "requestId": "req-98f24a1c",
  "data": [
    {
      "id": "intersection-01",
      "name": "Main Junction (North / South / East / West)",
      "laneCount": 4,
      "status": "ONLINE"
    }
  ],
  "error": null
}
```

---

### 2.3 Get Intersection Details
`GET /api/intersections/:id`
- **Purpose:** Retrieves geometric topography, lane limits, and safety boundaries for a junction.
- **Query Parameters:** None
- **Validation Rules:** `:id` must be non-empty alphanumeric string.
- **Status Codes:** `200 OK`, `404 Not Found`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:00.000Z",
  "requestId": "req-98f24a1d",
  "data": {
    "id": "intersection-01",
    "name": "Main Junction",
    "lanes": [
      { "laneId": "A", "direction": "NORTH", "capacity": 40 },
      { "laneId": "B", "direction": "SOUTH", "capacity": 40 },
      { "laneId": "C", "direction": "EAST", "capacity": 40 },
      { "laneId": "D", "direction": "WEST", "capacity": 40 }
    ],
    "signalConfiguration": {
      "minGreenSeconds": 10,
      "maxGreenSeconds": 75,
      "yellowSeconds": 4,
      "allRedSeconds": 2
    }
  },
  "error": null
}
```

---

### 2.4 Ingest Traffic State
`POST /api/traffic/state`
- **Purpose:** Ingests real-time vehicle counts, density, and queues from the vision node.
- **Validation Rules:** Conforms strictly to `traffic-state-schema.json`. Density must be between $0.0$ and $1.0$. Counts must be non-negative integers.
- **Status Codes:** `200 OK`, `400 Bad Request`, `429 Too Many Requests`
- **Request Example:**
```json
{
  "intersectionId": "intersection-01",
  "timestamp": "2026-09-26T09:30:00.000Z",
  "sensorHealth": {
    "cameraStatus": "ONLINE",
    "fps": 28.5,
    "frameDropRate": 0.01
  },
  "lanes": [
    {
      "laneId": "A",
      "approachDirection": "NORTH",
      "vehicleCount": 32,
      "density": 0.78,
      "queueLength": 18
    },
    {
      "laneId": "B",
      "approachDirection": "SOUTH",
      "vehicleCount": 7,
      "density": 0.22,
      "queueLength": 3
    }
  ]
}
```
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:00.120Z",
  "requestId": "req-98f24a1e",
  "data": {
    "status": "ACCEPTED",
    "ingestedLanes": 2,
    "optimizationTriggered": true
  },
  "error": null
}
```

---

### 2.5 Get Latest Traffic State
`GET /api/traffic/state/:intersectionId`
- **Purpose:** Fetches the most recent traffic state snapshot for a junction.
- **Status Codes:** `200 OK`, `404 Not Found`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:02.000Z",
  "requestId": "req-98f24a1f",
  "data": {
    "intersectionId": "intersection-01",
    "timestamp": "2026-09-26T09:30:00.000Z",
    "lanes": [
      { "laneId": "A", "vehicleCount": 32, "density": 0.78, "queueLength": 18 },
      { "laneId": "B", "vehicleCount": 7, "density": 0.22, "queueLength": 3 }
    ]
  },
  "error": null
}
```

---

### 2.6 Trigger Optimization Run
`POST /api/optimization/run`
- **Purpose:** Manually forces recalculation of the adaptive signal plan based on latest state.
- **Request Body:**
```json
{
  "intersectionId": "intersection-01",
  "forceRecalculation": true
}
```
- **Status Codes:** `200 OK`, `404 Not Found`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:05.000Z",
  "requestId": "req-98f24a20",
  "data": {
    "planId": "plan-adp-20260926-093005",
    "intersectionId": "intersection-01",
    "strategy": "ADAPTIVE_RULE_BASED",
    "cycleDurationSeconds": 120,
    "phases": [
      { "phaseIndex": 1, "activeLanes": ["A"], "greenTimeSeconds": 46 },
      { "phaseIndex": 2, "activeLanes": ["B"], "greenTimeSeconds": 14 }
    ]
  },
  "error": null
}
```

---

### 2.7 Get Active Signal Plan
`GET /api/signals/:intersectionId`
- **Purpose:** Returns the current operating signal timing schedule.
- **Status Codes:** `200 OK`, `404 Not Found`
- **Response Example:** Conforms to `signal-plan-schema.json`.

---

### 2.8 Trigger Comparative Simulation
`POST /api/simulation/run`
- **Purpose:** Runs a discrete-event benchmark comparing fixed vs. adaptive timing under an identical scenario.
- **Request Body:**
```json
{
  "intersectionId": "intersection-01",
  "scenarioName": "Morning_Peak_Four_Way",
  "durationSeconds": 1800
}
```
- **Status Codes:** `202 Accepted`, `400 Bad Request`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:10.000Z",
  "requestId": "req-98f24a21",
  "data": {
    "simulationId": "sim-run-20260926-101",
    "status": "IN_PROGRESS",
    "estimatedCompletionSeconds": 2
  },
  "error": null
}
```

---

### 2.9 Get Simulation Status
`GET /api/simulation/:id`
- **Purpose:** Polls or inspects status of an asynchronous simulation execution.
- **Status Codes:** `200 OK`, `404 Not Found`
- **Response Example:**
```json
{
  "success": true,
  "timestamp": "2026-09-26T09:30:12.000Z",
  "requestId": "req-98f24a22",
  "data": {
    "simulationId": "sim-run-20260926-101",
    "status": "COMPLETED",
    "startedAt": "2026-09-26T09:30:10.000Z",
    "endedAt": "2026-09-26T09:30:12.000Z"
  },
  "error": null
}
```

---

### 2.10 Get Simulation Comparative Metrics
`GET /api/metrics/:simulationId`
- **Purpose:** Returns granular metrics comparing fixed-time baseline with adaptive control.
- **Status Codes:** `200 OK`, `404 Not Found`
- **Response Example:** Conforms to `simulation-result-schema.json`.

---

## 3. Real-Time WebSocket Specifications

Clients connect to the Socket.io namespace `/traffic` and subscribe to room `intersection:{id}`.

### 3.1 Event: `traffic:update`
- **Trigger:** Emitted every time a valid `POST /api/traffic/state` payload is processed.
- **Payload:** Structured `TrafficState` object showing latest vehicle counts, densities, and queue depths.

### 3.2 Event: `signal:update`
- **Trigger:** Emitted whenever the optimizer reallocates phase durations or advances active phase.
- **Payload:** Structured `SignalPlan` object containing current phase countdown and updated split times.

### 3.3 Event: `simulation:update`
- **Trigger:** Emitted during progress intervals of a running discrete-event simulation.
- **Payload:** `{ "simulationId": "sim-run-101", "progressPercent": 75, "step": "adaptive_batch" }`.

### 3.4 Event: `metrics:update`
- **Trigger:** Emitted immediately upon completion of comparative simulation run.
- **Payload:** Full `SimulationResult` object with comparative delta calculations.
