# Real-Time WebSocket Architecture

## 1. Real-Time Communication Strategy

Urban traffic management requires low-latency, bi-directional telemetry streams. While configuration queries and manual interventions use REST endpoints, continuous telemetry and live signal updates are driven by **WebSockets via Socket.io**.

### 1.1 Architectural Advantages
- **Elimination of Polling:** Prevents client polling storms against the Node.js event loop.
- **Selective Room Subscriptions:** Clients join rooms scoped to specific junctions (`intersection:intersection-01`), receiving only relevant telemetry.
- **Graceful Fallback:** Socket.io provides automatic fallback to HTTP long-polling in restrictive enterprise network or firewall environments.

---

## 2. Event Topology and Data Flow

```mermaid
sequenceDiagram
    autonumber
    actor Vision as Vision Ingestion Node
    participant Backend as Node.js Backend
    participant WsHub as Socket.io Server (WebSocket Gateway)
    actor Client as React Dashboard Client

    Client->>WsHub: Connect (Handshake + Client Auth)
    WsHub-->>Client: Connection Established (socket.id)
    
    Client->>WsHub: emit("subscribe:intersection", { intersectionId: "intersection-01" })
    WsHub->>WsHub: Join Socket to Room "intersection-01"

    loop Every Camera Ingestion Cycle (1-2s)
        Vision->>Backend: Ingest New Lane Telemetry
        Backend->>WsHub: Publish to Room "intersection-01"
        WsHub-->>Client: emit("traffic:update", TrafficState)
    end

    opt Triggered by Demand Threshold or Cycle Transition
        Backend->>Backend: Optimization Engine Recomputes Phases
        Backend->>WsHub: Publish to Room "intersection-01"
        WsHub-->>Client: emit("signal:update", SignalPlan)
    end

    opt Triggered by Asynchronous Simulation Execution
        Backend->>WsHub: Publish Simulation Progress
        WsHub-->>Client: emit("simulation:update", { progressPercent: 65, step: "adaptive_run" })
        Backend->>WsHub: Publish Final Benchmark Metrics
        WsHub-->>Client: emit("metrics:update", SimulationResult)
    end
```

---

## 3. WebSocket Event Catalog

| Event Name | Direction | Trigger Condition | Payload Summary |
| :--- | :--- | :--- | :--- |
| `subscribe:intersection` | Client $\to$ Server | Dashboard mounts junction view | `{ "intersectionId": "intersection-01" }` |
| `traffic:update` | Server $\to$ Client | Vision node submits new lane detection data | Full `TrafficState` object including lane counts, densities, and queue depths |
| `signal:update` | Server $\to$ Client | Optimizer generates new adaptive phase allocation | Full `SignalPlan` object including active phase index, remaining green seconds |
| `simulation:update` | Server $\to$ Client | Discrete-event simulation completes a milestone step | `{ "simulationId": string, "progress": number, "currentPhase": string }` |
| `metrics:update` | Server $\to$ Client | Simulation run completes and comparative deltas finish computing | Full `SimulationResult` object with comparative deltas |

---

## 4. Connection Lifecycle & Resilience Design

```mermaid
stateDiagram-v2
    [*] --> Disconnected
    Disconnected --> Connecting : Client Initiates Socket.io Connection
    Connecting --> Connected : Successful Handshake
    Connecting --> Failed : Network Unreachable / CORS Rejection
    Failed --> Connecting : Exponential Backoff Retry (1s, 2s, 4s, max 10s)
    
    Connected --> Subscribed : Emit "subscribe:intersection"
    Subscribed --> ActiveStreaming : Room Joined
    
    ActiveStreaming --> Disconnected : Heartbeat Ping/Pong Timeout (45s)
    ActiveStreaming --> ActiveStreaming : Ingest "traffic:update" / "signal:update"
```

### 4.1 Resilience Safeguards
1. **Heartbeat Monitoring:** Default 25-second ping interval with a 20-second timeout to promptly tear down stale TCP sockets.
2. **Exponential Backoff:** Clients reconnect with randomized jitter to prevent thundering herd scenarios upon backend restarts.
3. **Room Isolation:** Operators viewing Junction A receive zero network traffic from Junction B, ensuring client scalability.
