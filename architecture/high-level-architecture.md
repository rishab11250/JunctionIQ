# High-Level System Architecture

## 1. Architectural Overview

JunctionIQ is designed as a modular, decoupled software intelligence platform for urban intersections. The platform transforms existing roadside optical sensors (CCTV / IP traffic cameras) into structured digital telemetry, computes adaptive signal timings deterministically, and validates timing strategies within a controlled simulation loop before projecting operational insights to an operator dashboard.

The architecture strictly separates:
- **Perception Layer (Python Vision Service):** High-throughput video frame ingestion, neural vehicle inference, vehicle tracking, and lane spatial association.
- **Coordination & Intelligence Layer (Node.js/Express Backend):** Ingests normalized traffic state, executes deterministic timing heuristics, coordinates simulation runs, and manages real-time broadcast channels.
- **Simulation Layer (In-Engine Discrete-Event Simulator):** Clones identical arrival queues under fixed versus adaptive timing regimes to calculate comparative efficiency indicators.
- **Presentation Layer (React Dashboard):** Visualizes intersection status, live queues, time allocation deltas, and telemetry trends.

---

## 2. End-to-End System Topology

```mermaid
flowchart TD
    subgraph SENSING["Physical & Simulated Sensing Layer"]
        CAM["Traffic Camera Feeds<br/>(RTSP / Video File Stream)"]
    end

    subgraph VISION["Perception & Vision Service (Python / Edge)"]
        CAP["Frame Capture & Preprocessing"]
        YOLO["YOLO Object Detector<br/>(Vehicle Detection)"]
        TRACK["ByteTrack / Multi-Object Tracker"]
        ROI["Lane ROI Spatial Mapping<br/>& Queue Estimator"]
        STATE_GEN["Traffic State Aggregator"]
    end

    subgraph BACKEND["Core Coordination Platform (Node.js & TypeScript)"]
        API_GW["REST Ingestion & Control API"]
        STATE_MGR["In-Memory Traffic State Manager"]
        OPT_ENG["Deterministic Adaptive<br/>Optimization Engine"]
        SIM_COORD["Simulation Orchestrator"]
        WS_HUB["WebSocket Event Gateway<br/>(Socket.io)"]
    end

    subgraph SIMULATION["Validation & Benchmark Layer"]
        DES["Traffic Demand Simulator<br/>(Fixed vs Adaptive Baseline)"]
        METRICS_CALC["Comparative Metrics Engine"]
    end

    subgraph CLIENT["Operator Presentation (React & TypeScript)"]
        UI_DASH["Interactive Junction Dashboard"]
        LIVE_VIEW["Lane Congestion & Phase Monitor"]
        ANALYTICS["Comparative Metrics Visualizer"]
    end

    CAM -->|RTSP / MP4| CAP
    CAP --> YOLO
    YOLO --> TRACK
    TRACK --> ROI
    ROI --> STATE_GEN
    STATE_GEN -->|POST /api/traffic/state| API_GW

    API_GW --> STATE_MGR
    STATE_MGR --> OPT_ENG
    OPT_ENG -->|Signal Plan| SIM_COORD
    SIM_COORD --> DES
    DES --> METRICS_CALC
    METRICS_CALC --> SIM_COORD

    STATE_MGR -.->|traffic:update| WS_HUB
    OPT_ENG -.->|signal:update| WS_HUB
    SIM_COORD -.->|simulation:update| WS_HUB

    WS_HUB ==>|WebSocket Streams| UI_DASH
    API_GW -.->|REST Queries| UI_DASH
    UI_DASH --> LIVE_VIEW
    UI_DASH --> ANALYTICS
```

---

## 3. Subsystem Breakdown and Responsibilities

| Subsystem | Core Responsibilities | Technology Stack | Communication Mode |
| :--- | :--- | :--- | :--- |
| **Vision Service** | Ingest video stream, detect vehicular bounding boxes, track identities across frames, project centroids onto lane polygons, calculate density and queue depth. | Python 3.10+, OpenCV, YOLOv8/v10 (CPU/CUDA) | Outbound HTTP POST to Backend (`/api/traffic/state`) |
| **Backend Core** | Central state holding, request validation, executing signal allocation rules, orchestrating simulation runs, handling client sessions. | Node.js, Express, TypeScript | Inbound REST API, Outbound WebSockets |
| **Optimization Engine** | Evaluate lane demand scores using weighted count, queue, and density; produce constrained green phase allocations respecting safe minimums and maximums. | Deterministic TypeScript Algorithm | Synchronous Internal Service Call |
| **Simulation Engine** | Run synthetic traffic arrival batches under identical randomized seeds comparing standard fixed cycles vs adaptive signal allocations. | Discrete-Event Queue Engine (JS/TS) | In-process or Worker Thread invocation |
| **Frontend Portal** | Render four-way junction state, active light phases, queue indicators, and comparative performance deltas. | React 18+, Vite, TailwindCSS, shadcn/ui, Recharts | Socket.io Client & Axios REST Client |

---

## 4. Key Architectural Design Principles

1. **Strict Decoupling of Perception and Optimization:**  
   The vision pipeline operates autonomously as an upstream telemetry producer. The backend does not care whether state arrives from a live physical camera or a synthetic pre-recorded video stream, as long as it adheres to the `TrafficState` JSON contract.

2. **Deterministic & Explainable Decision-Making:**  
   Unlike black-box reinforcement learning models that pose unpredictable safety boundary behaviors, the MVP optimization engine utilizes bounded arithmetic scoring. Every millisecond of green light allocated can be traced to verifiable lane telemetry.

3. **Software-Only Safety Isolation:**  
   The system never directly manipulates physical signal controller hardware (NEMA TS2 or 2070 cabinets). All outputs are published as recommendations and tested through simulated digital twins.

4. **Lean Resource Footprint for Individual Maintainability:**  
   The system deliberately avoids complex distributed message brokers (e.g., Kafka) or cluster orchestrators (e.g., Kubernetes) for the MVP, allowing seamless execution on standard workstations or lightweight cloud instances.
