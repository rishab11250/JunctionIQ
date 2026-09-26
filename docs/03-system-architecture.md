# System Architecture Specification

## 1. Architectural Philosophy

JunctionIQ is engineered around a **decoupled, event-driven, micro-service-ready modular architecture**. Every subsystem is bounded by explicit JSON contracts, ensuring that sensing, intelligence, simulation, and presentation can evolve or scale independently.

---

## 2. The Eight Architectural Layers

```mermaid
flowchart TB
    subgraph L1["Layer 1: Data Input Layer"]
        IN1["RTSP IP Camera Streams / Local MP4 Video Files"]
    end

    subgraph L2["Layer 2: Vision & Perception Layer"]
        V1["OpenCV Decoder"] --> V2["YOLO Vehicle Detector"]
        V2 --> V3["ByteTrack Multi-Object Tracker"]
        V3 --> V4["Polygon ROI Lane Association"]
    end

    subgraph L3["Layer 3: Traffic Intelligence Layer"]
        I1["Lane Vehicle Counter"]
        I2["Queue Depth Analyzer"]
        I3["Density Ratio Estimator"]
        I4["TrafficState Document Formatter"]
    end

    subgraph L4["Layer 4: Backend Orchestration Layer"]
        B1["Express API Gateway"]
        B2["In-Memory State Manager"]
        B3["Event Dispatcher"]
    end

    subgraph L5["Layer 5: Optimization Layer"]
        O1["Demand Score Engine"]
        O2["Safety Boundary Validator (Min/Max Green)"]
        O3["SignalPlan Generator"]
    end

    subgraph L6["Layer 6: Simulation Layer"]
        S1["Discrete-Event Simulation Loop"]
        S2["Fixed vs. Adaptive Twin Benchmark"]
        S3["Metrics Aggregator"]
    end

    subgraph L7["Layer 7: Real-Time Communication Layer"]
        W1["Socket.io WebSocket Server"]
        W2["Room Scoping: intersection-{id}"]
    end

    subgraph L8["Layer 8: Frontend Presentation Layer"]
        F1["React + Vite Single Page Application"]
        F2["Interactive Junction Map & Heatmap"]
        F3["Live Analytics & Recharts Graphs"]
    end

    L1 --> L2
    L2 --> L3
    L3 -->|POST /api/traffic/state| L4
    L4 --> L5
    L5 --> L6
    L4 --> L7
    L7 ==>|WebSockets| L8
    L4 -.->|REST Queries| L8
```

---

## 3. Detailed Data Flow & Interaction Diagrams

### 3.1 Service Communication Protocol
The system uses asymmetric transport protocols matched to workload requirements:
- **Vision Node $\to$ Backend:** Asynchronous HTTP/1.1 or HTTP/2 REST `POST` with strict JSON schema validation.
- **Backend $\to$ Simulation Engine:** In-memory synchronous function invocation or Node.js Worker Thread.
- **Backend $\to$ Frontend Dashboard:** Full-duplex WebSocket connection (`Socket.io`) for low-overhead telemetry broadcast.
- **Frontend Dashboard $\to$ Backend:** Standard REST `GET`/`POST` for configuration, ad-hoc simulation triggers, and manual overrides.

### 3.2 Request & Processing Flow

```mermaid
sequenceDiagram
    autonumber
    participant Cam as Video Feed
    participant Vis as Python Vision
    participant API as Node.js Backend
    participant Optimizer as Optimizer Engine
    participant Sim as Simulator
    participant UI as React UI

    Cam->>Vis: Ingest Raw Video Frames
    Vis->>Vis: Run YOLO + ByteTrack + ROI Mapping
    Vis->>API: POST /api/traffic/state (Lane State JSON)
    API->>API: In-Memory State Cache Update
    API-->>UI: WebSocket emit("traffic:update")
    
    API->>Optimizer: calculateOptimalPhases(trafficState)
    Optimizer->>Optimizer: Score Lane Urgency & Apply Min/Max Bounds
    Optimizer-->>API: Return SignalPlan
    API-->>UI: WebSocket emit("signal:update")

    opt When Simulation Benchmark Triggered
        API->>Sim: runComparativeSimulation(SignalPlan)
        Sim->>Sim: Run Fixed Baseline & Adaptive Twins
        Sim-->>API: Return SimulationResult
        API-->>UI: WebSocket emit("metrics:update")
    end
```

---

## 4. Failure Recovery & Graceful Degradation Flow

Urban civic infrastructure requires resilient failure recovery modes. The following diagram illustrates how JunctionIQ handles camera feed drops, network degradation, or vision node failures:

```mermaid
flowchart TD
    START["Normal Operating State: Adaptive Computer Vision Control"] --> MONITOR{"Heartbeat / Frame Rate Check"}
    
    MONITOR -->|Feed Drops / Timeout > 10s| DETECT_FAIL["Failure Detected: Sensor Health Degraded"]
    DETECT_FAIL --> NOTIFY["Backend Logs Alert & Emits WebSocket Warning to UI"]
    
    DETECT_FAIL --> FALLBACK["AUTOMATIC FALLBACK:<br/>Engage Fail-Safe Fixed-Time Signal Plan"]
    
    FALLBACK --> HOLD["Maintain Fixed Conservative Cycle (e.g. 30s/phase)"]
    
    HOLD --> RECONNECT{"Vision Stream Re-established?"}
    RECONNECT -->|No| HOLD
    RECONNECT -->|Yes: 5 Consecutive Healthy Telemetry Frames| RESTORE["Restore Normal Adaptive Timing"]
    RESTORE --> START
```

### 4.1 Safety Fail-Safe Guarantees
- **Timeout Threshold:** If the backend does not receive a valid `TrafficState` payload within 10 seconds, it automatically demotes the junction to a predefined, safe `FIXED_BASELINE` plan.
- **Minimum Green Enforcements:** Even under extreme asymmetric congestion, no green phase can be compressed below 10 seconds, guaranteeing safe pedestrian clearance.
- **Payload Sanity Bounds:** If camera detection reports an implausible vehicle density ($> 1.0$) or negative counts, the middleware drops the packet and maintains the previous valid state.

---

## 5. Architectural Modularity Benefits

1. **Independent Language Runtimes:** Python is leveraged where C++ GPU bindings and deep learning packages excel; TypeScript/Node.js is utilized where high-concurrency event loops and async I/O dominate.
2. **Pluggable Optimizer Engine:** The deterministic rule-based optimizer can be swapped or augmented in future versions (e.g. with linear programming or research models) without modifying the vision service or frontend.
3. **Decoupled Client Interfaces:** Any consumer (web dashboard, municipal API, mobile alert system) can consume standard WebSocket feeds without altering core backend logic.
