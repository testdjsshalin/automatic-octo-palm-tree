# SynthCommute: Architecture Diagrams

## Visual System Architecture for Hackathon Presentation

---

## 1. High-Level System Architecture

### Mermaid Diagram (renders in GitHub, VSCode, etc.)

```mermaid
flowchart TB
    subgraph PRESENTATION["PRESENTATION LAYER"]
        MA[Mobile App<br/>React Native]
        WP[Web Portal<br/>React/Next.js]
        OD[Operator Dashboard<br/>Vue.js]
    end

    subgraph GATEWAY["API GATEWAY & EDGE"]
        AG[API Gateway<br/>Kong/Envoy]
        EC[Edge Cache<br/>Redis]
        WS[WebSocket<br/>Manager]
    end

    subgraph SERVICES["MICROSERVICES ECOSYSTEM"]
        JP[Journey<br/>Planner]
        RT[Real-Time<br/>Tracker]
        PE[Predictive<br/>Engine]
        CS[Crowdsource<br/>Service]
        AS[Alert<br/>Service]
        US[User<br/>Service]
        DI[Data<br/>Ingestor]
    end

    subgraph STREAMING["EVENT STREAMING"]
        KF[Apache Kafka<br/>+ KSQL]
    end

    subgraph PROCESSING["DATA PROCESSING & ML"]
        FL[Stream Processing<br/>Apache Flink]
        SP[Batch Processing<br/>Apache Spark]
        FS[Feature Store<br/>Feast]
        MS[Model Serving<br/>TF Serving]
    end

    subgraph STORAGE["STORAGE LAYER"]
        TS[(Time-Series<br/>InfluxDB)]
        GR[(Graph DB<br/>Neo4j)]
        DO[(Document<br/>MongoDB)]
        WH[(Warehouse<br/>BigQuery)]
        RD[(Cache<br/>Redis)]
    end

    subgraph SOURCES["DATA SOURCES"]
        GT[GTFS/<br/>Realtime]
        IO[IoT<br/>Sensors]
        CV[CCTV/<br/>Vision]
        UR[User<br/>Reports]
        IT[Informal<br/>Transit]
        TP[Third-Party<br/>APIs]
    end

    MA & WP & OD --> AG
    AG --> EC
    AG --> WS
    AG --> JP & RT & PE & CS & AS & US

    JP & RT & PE --> KF
    CS & DI --> KF
    AS --> WS

    KF --> FL
    FL --> FS
    FS --> MS
    MS --> PE

    SP --> WH
    FL --> TS & GR & DO

    GT & IO & CV & UR & IT & TP --> DI
    DI --> KF

    PE --> RD
    JP --> GR

    style PRESENTATION fill:#e1f5fe
    style GATEWAY fill:#fff3e0
    style SERVICES fill:#e8f5e9
    style STREAMING fill:#fce4ec
    style PROCESSING fill:#f3e5f5
    style STORAGE fill:#fff8e1
    style SOURCES fill:#efebe9
```

---

## 2. Data Flow Architecture

```mermaid
flowchart LR
    subgraph INGESTION["DATA INGESTION"]
        direction TB
        S1[GTFS Static<br/>Daily Pull]
        S2[GTFS Realtime<br/>30s Pull]
        S3[IoT Sensors<br/>MQTT Stream]
        S4[CCTV Metadata<br/>CV Pipeline]
        S5[User Reports<br/>WebSocket]
        S6[Informal Transit<br/>REST API]
    end

    subgraph VALIDATION["VALIDATION & ENRICHMENT"]
        V1[Schema<br/>Validation]
        V2[Anonymization<br/>Engine]
        V3[Quality<br/>Scoring]
        V4[Geo-Hash<br/>Indexing]
    end

    subgraph KAFKA["KAFKA TOPICS"]
        K1[transit.realtime]
        K2[sensor.processed]
        K3[user.events]
        K4[crowd.metrics]
        K5[anomaly.alerts]
    end

    subgraph FLINK["STREAM PROCESSING"]
        F1[Crowd Density<br/>Calculator]
        F2[Delay<br/>Propagator]
        F3[Transfer Time<br/>Estimator]
        F4[Anomaly<br/>Detector]
        F5[Journey<br/>Tracker]
    end

    subgraph FEATURES["FEATURE STORE"]
        FS1[Real-time<br/>Features]
        FS2[Batch<br/>Features]
    end

    subgraph ML["ML INFERENCE"]
        M1[ETA<br/>Predictor]
        M2[Crowd<br/>Forecaster]
        M3[Risk<br/>Scorer]
        M4[Personalization<br/>Engine]
    end

    subgraph API["API LAYER"]
        A1[Journey<br/>Spectrum]
        A2[Live<br/>Alerts]
    end

    S1 & S2 & S3 & S4 & S5 & S6 --> V1
    V1 --> V2 --> V3 --> V4
    V4 --> K1 & K2 & K3

    K1 & K2 & K3 --> F1 & F2 & F3 & F4 & F5
    F1 --> K4
    F4 --> K5

    F1 & F2 & F3 --> FS1
    FS1 & FS2 --> M1 & M2 & M3 & M4

    M1 & M2 & M3 & M4 --> A1
    K5 --> A2

    style INGESTION fill:#bbdefb
    style VALIDATION fill:#c8e6c9
    style KAFKA fill:#ffcdd2
    style FLINK fill:#e1bee7
    style FEATURES fill:#fff9c4
    style ML fill:#b2dfdb
    style API fill:#d7ccc8
```

---

## 3. Probabilistic Journey Engine Architecture

```mermaid
flowchart TB
    subgraph INPUT["USER INPUT"]
        O[Origin]
        D[Destination]
        T[Departure Time]
        P[User Preferences]
    end

    subgraph CONTEXT["CONTEXT GATHERING"]
        C1[Live Crowd<br/>Density]
        C2[Current<br/>Delays]
        C3[Weather<br/>Conditions]
        C4[Active<br/>Disruptions]
        C5[User<br/>History]
    end

    subgraph ROUTING["ROUTE GENERATION"]
        R1[Multi-Modal<br/>Graph Search]
        R2[A* Algorithm<br/>with Constraints]
        R3[Candidate<br/>Routes]
    end

    subgraph SIMULATION["MONTE CARLO SIMULATION"]
        S1[Sample Delay<br/>Distributions]
        S2[Sample Crowd<br/>Distributions]
        S3[Sample Transfer<br/>Times]
        S4[1000 Simulation<br/>Runs]
    end

    subgraph ANALYSIS["STATISTICAL ANALYSIS"]
        A1[Compute ETA<br/>Distribution]
        A2[Calculate<br/>Confidence Intervals]
        A3[Identify<br/>Risk Factors]
        A4[Compute<br/>Stress Score]
    end

    subgraph OUTPUT["JOURNEY SPECTRUM"]
        J1[FASTEST<br/>85% confidence<br/>42-58 mins]
        J2[RELIABLE<br/>92% confidence<br/>45-55 mins]
        J3[LOW-STRESS<br/>95% confidence<br/>50-60 mins]
    end

    O & D & T & P --> C1 & C2 & C3 & C4 & C5
    C1 & C2 & C3 & C4 & C5 --> R1
    R1 --> R2 --> R3
    R3 --> S1 & S2 & S3
    S1 & S2 & S3 --> S4
    S4 --> A1 & A2 & A3 & A4
    A1 & A2 & A3 & A4 --> J1 & J2 & J3

    style INPUT fill:#e3f2fd
    style CONTEXT fill:#f3e5f5
    style ROUTING fill:#e8f5e9
    style SIMULATION fill:#fff3e0
    style ANALYSIS fill:#fce4ec
    style OUTPUT fill:#e0f7fa
```

---

## 4. ML Pipeline Architecture

```mermaid
flowchart LR
    subgraph DATA["DATA COLLECTION"]
        D1[Journey<br/>Logs]
        D2[Sensor<br/>Data]
        D3[Crowd<br/>Metrics]
        D4[Disruption<br/>Events]
    end

    subgraph FEATURES["FEATURE ENGINEERING"]
        F1[Temporal<br/>Features]
        F2[Spatial<br/>Features]
        F3[Network<br/>Features]
        F4[User<br/>Features]
    end

    subgraph TRAINING["MODEL TRAINING"]
        T1[ETA Model<br/>LSTM]
        T2[Crowd Model<br/>Transformer]
        T3[Risk Model<br/>Bayesian Net]
        T4[Personal Model<br/>Contextual Bandit]
    end

    subgraph REGISTRY["MODEL REGISTRY"]
        R1[Version<br/>Control]
        R2[Metadata<br/>Store]
        R3[Artifact<br/>Storage]
    end

    subgraph SERVING["MODEL SERVING"]
        S1[TensorFlow<br/>Serving]
        S2[A/B Testing<br/>Router]
        S3[Shadow<br/>Mode]
    end

    subgraph MONITORING["MONITORING"]
        M1[Prediction<br/>Accuracy]
        M2[Latency<br/>Metrics]
        M3[Data<br/>Drift]
        M4[Model<br/>Drift]
    end

    D1 & D2 & D3 & D4 --> F1 & F2 & F3 & F4
    F1 & F2 & F3 & F4 --> T1 & T2 & T3 & T4
    T1 & T2 & T3 & T4 --> R1 & R2 & R3
    R1 & R2 & R3 --> S1 & S2 & S3
    S1 --> M1 & M2 & M3 & M4
    M3 & M4 -.->|Retrain Trigger| T1 & T2 & T3 & T4

    style DATA fill:#e1f5fe
    style FEATURES fill:#fff3e0
    style TRAINING fill:#e8f5e9
    style REGISTRY fill:#f3e5f5
    style SERVING fill:#fce4ec
    style MONITORING fill:#fff8e1
```

---

## 5. User Journey Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant M as Mobile App
    participant G as API Gateway
    participant J as Journey Planner
    participant P as Predictive Engine
    participant F as Feature Store
    participant ML as ML Models
    participant C as Cache

    U->>M: Request journey (A to B)
    M->>G: POST /api/journey
    G->>G: Authenticate & Rate Limit
    G->>J: Forward request

    J->>C: Check cache (hot routes)
    C-->>J: Cache miss

    par Parallel Context Fetch
        J->>F: Get real-time features
        J->>P: Get user profile
    end

    F-->>J: Crowd, delays, weather
    P-->>J: User preferences

    J->>J: Generate candidate routes (A*)

    loop Monte Carlo (1000 runs)
        J->>ML: Sample predictions
        ML-->>J: Delay, crowd, transfer samples
    end

    J->>J: Compute statistics & rank
    J->>C: Store in cache (5 min TTL)
    J-->>G: Journey spectrum response
    G-->>M: JSON response
    M-->>U: Display 3 options

    Note over U,M: User selects "Reliable" option

    U->>M: Select option & start journey
    M->>G: POST /api/journey/start
    G->>J: Start journey tracking

    loop Real-time Updates
        J->>F: Monitor live conditions
        F-->>J: Updated metrics
        alt Disruption Detected
            J->>M: Push alert (WebSocket)
            M->>U: "Platform changed to 4"
        end
    end
```

---

## 6. Anomaly Detection Flow

```mermaid
flowchart TB
    subgraph SIGNALS["INPUT SIGNALS"]
        S1[Crowd Flow<br/>Direction]
        S2[Audio<br/>Announcement]
        S3[Schedule<br/>Deviation]
        S4[Device<br/>Clustering]
        S5[Social Media<br/>Mentions]
    end

    subgraph PROCESSING["SIGNAL PROCESSING"]
        P1[Flow Vector<br/>Analysis]
        P2[Speech-to-Text<br/>+ NLP]
        P3[Expected vs<br/>Actual Arrival]
        P4[Geo-Cluster<br/>Detection]
        P5[Burst<br/>Detection]
    end

    subgraph FUSION["BAYESIAN FUSION"]
        B1[Prior<br/>Probabilities]
        B2[Likelihood<br/>Calculation]
        B3[Posterior<br/>Inference]
    end

    subgraph DECISION["DECISION ENGINE"]
        D1{Probability<br/>> 85%?}
        D2[Generate<br/>Alert]
        D3[Continue<br/>Monitoring]
    end

    subgraph OUTPUT["ALERT OUTPUT"]
        O1[Push to<br/>Affected Users]
        O2[Update<br/>Journey Plans]
        O3[Log for<br/>Model Training]
    end

    S1 --> P1
    S2 --> P2
    S3 --> P3
    S4 --> P4
    S5 --> P5

    P1 & P2 & P3 & P4 & P5 --> B1
    B1 --> B2 --> B3
    B3 --> D1

    D1 -->|Yes| D2
    D1 -->|No| D3
    D3 -.->|Continue| S1

    D2 --> O1 & O2 & O3

    style SIGNALS fill:#e3f2fd
    style PROCESSING fill:#fff3e0
    style FUSION fill:#e8f5e9
    style DECISION fill:#fce4ec
    style OUTPUT fill:#e0f7fa
```

---

## 7. Deployment Architecture

```mermaid
flowchart TB
    subgraph USERS["USERS"]
        U1[Mobile Users]
        U2[Web Users]
        U3[Operators]
    end

    subgraph CDN["CDN LAYER"]
        CF[CloudFront/<br/>Cloudflare]
    end

    subgraph LB["LOAD BALANCING"]
        ALB[Application<br/>Load Balancer]
    end

    subgraph MUMBAI["REGION: MUMBAI (Primary)"]
        subgraph K8S_MUM["Kubernetes Cluster"]
            API_M[API Services<br/>x10 pods]
            ML_M[ML Services<br/>x5 pods]
            STREAM_M[Stream Processing<br/>x8 taskmanagers]
        end
        subgraph DB_MUM["Databases"]
            PG_M[(Primary DBs)]
            REDIS_M[(Redis Cluster)]
        end
        subgraph EDGE_MUM["Edge Nodes"]
            E1[Churchgate<br/>Station]
            E2[Dadar<br/>Station]
            E3[Andheri<br/>Station]
        end
    end

    subgraph DELHI["REGION: DELHI (Secondary)"]
        subgraph K8S_DEL["Kubernetes Cluster"]
            API_D[API Services<br/>x5 pods]
            ML_D[ML Services<br/>x3 pods]
        end
        subgraph DB_DEL["Databases"]
            PG_D[(Read Replicas)]
            REDIS_D[(Redis Replica)]
        end
    end

    subgraph KAFKA_CLUSTER["KAFKA CLUSTER"]
        KF1[Broker 1]
        KF2[Broker 2]
        KF3[Broker 3]
    end

    U1 & U2 & U3 --> CF
    CF --> ALB
    ALB --> API_M & API_D

    API_M --> ML_M
    API_M --> REDIS_M
    API_M --> PG_M

    API_D --> ML_D
    API_D --> REDIS_D
    API_D --> PG_D

    PG_M -.->|Sync| PG_D
    REDIS_M -.->|Sync| REDIS_D

    STREAM_M --> KF1 & KF2 & KF3

    E1 & E2 & E3 --> API_M

    style USERS fill:#e1f5fe
    style CDN fill:#fff3e0
    style LB fill:#e8f5e9
    style MUMBAI fill:#e3f2fd
    style DELHI fill:#fce4ec
    style KAFKA_CLUSTER fill:#fff8e1
```

---

## 8. Database Schema Overview

```mermaid
erDiagram
    USER ||--o{ JOURNEY : takes
    USER ||--o{ PREFERENCE : has
    USER ||--o{ REPORT : submits

    JOURNEY ||--|{ SEGMENT : contains
    JOURNEY ||--o{ PREDICTION : has

    SEGMENT ||--o| STATION : starts_at
    SEGMENT ||--o| STATION : ends_at
    SEGMENT ||--o| VEHICLE : uses

    STATION ||--o{ PLATFORM : has
    STATION ||--o{ CROWD_METRIC : measured_at

    VEHICLE ||--o{ POSITION : tracked_at
    VEHICLE ||--o| ROUTE : operates_on

    REPORT ||--o| STATION : about
    REPORT ||--o| VALIDATION : receives

    USER {
        uuid id PK
        string email
        float trust_score
        datetime created_at
    }

    JOURNEY {
        uuid id PK
        uuid user_id FK
        point origin
        point destination
        datetime departure
        string status
    }

    STATION {
        uuid id PK
        string name
        point location
        string geohash
    }

    CROWD_METRIC {
        uuid id PK
        uuid station_id FK
        float density
        datetime measured_at
    }

    PREDICTION {
        uuid id PK
        uuid journey_id FK
        int eta_p50
        int eta_p95
        float confidence
    }
```

---

## 9. Component Interaction Matrix

```
                    ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
                    │ API  │Journey│Real- │Predict│Crowd │Alert │Feature│ ML  │
                    │Gateway│Planner│Time  │Engine│source│Service│Store │Serve│
┌───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ API Gateway       │  -   │ SYNC │ SYNC │  -   │ SYNC │ WS   │  -   │  -   │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Journey Planner   │ SYNC │  -   │ SYNC │ SYNC │  -   │ ASYNC│ SYNC │ SYNC │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Real-Time Tracker │ SYNC │ SYNC │  -   │ SYNC │  -   │ ASYNC│ SYNC │  -   │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Predictive Engine │  -   │ SYNC │ SYNC │  -   │  -   │  -   │ SYNC │ SYNC │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Crowdsource Svc   │ SYNC │  -   │  -   │  -   │  -   │ ASYNC│  -   │  -   │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Alert Service     │ WS   │ ASYNC│ ASYNC│  -   │ ASYNC│  -   │  -   │  -   │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ Feature Store     │  -   │ SYNC │ SYNC │ SYNC │  -   │  -   │  -   │ SYNC │
├───────────────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤
│ ML Serving        │  -   │ SYNC │  -   │ SYNC │  -   │  -   │ SYNC │  -   │
└───────────────────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

Legend: SYNC = Synchronous HTTP/gRPC | ASYNC = Kafka | WS = WebSocket
```

---

## 10. Quick Reference: ASCII Architecture (for terminals)

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                         SYNTHCOMMUTE ARCHITECTURE                              ║
╠═══════════════════════════════════════════════════════════════════════════════╣
║                                                                                ║
║   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                       ║
║   │  Mobile App │    │  Web Portal │    │  Operator   │   PRESENTATION        ║
║   └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                       ║
║          │                  │                  │                               ║
║          └──────────────────┼──────────────────┘                               ║
║                             ▼                                                  ║
║   ┌─────────────────────────────────────────────────────┐                     ║
║   │              API GATEWAY (Kong/Envoy)                │   GATEWAY          ║
║   │         Auth | Rate Limit | WebSocket | Cache        │                     ║
║   └──────────────────────────┬──────────────────────────┘                     ║
║                              │                                                 ║
║   ┌──────────────────────────┼──────────────────────────┐                     ║
║   │                          ▼                          │                     ║
║   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │   MICROSERVICES     ║
║   │  │ Journey │ │Real-Time│ │Predictive│ │Crowdsrc │   │                     ║
║   │  │ Planner │ │ Tracker │ │ Engine  │ │ Service │   │                     ║
║   │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘   │                     ║
║   └───────┼───────────┼───────────┼───────────┼────────┘                     ║
║           │           │           │           │                               ║
║   ┌───────┴───────────┴───────────┴───────────┴────────┐                     ║
║   │              APACHE KAFKA (Event Bus)               │   STREAMING         ║
║   └──────────────────────────┬──────────────────────────┘                     ║
║                              │                                                 ║
║   ┌──────────────────────────┼──────────────────────────┐                     ║
║   │  ┌─────────┐ ┌─────────┐ │ ┌─────────┐ ┌─────────┐  │                     ║
║   │  │  Flink  │ │  Spark  │ │ │ Feature │ │   ML    │  │   DATA/ML          ║
║   │  │ Stream  │ │  Batch  │◄┘ │  Store  │ │ Serving │  │                     ║
║   │  └────┬────┘ └────┬────┘   └────┬────┘ └────┬────┘  │                     ║
║   └───────┼───────────┼─────────────┼───────────┼───────┘                     ║
║           │           │             │           │                              ║
║   ┌───────┴───────────┴─────────────┴───────────┴───────┐                     ║
║   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │                     ║
║   │  │InfluxDB │ │  Neo4j  │ │ MongoDB │ │BigQuery │   │   STORAGE           ║
║   │  │TimeSeries│ │  Graph  │ │Document │ │Warehouse│   │                     ║
║   │  └─────────┘ └─────────┘ └─────────┘ └─────────┘   │                     ║
║   └─────────────────────────────────────────────────────┘                     ║
║                              ▲                                                 ║
║   ┌──────────────────────────┼──────────────────────────┐                     ║
║   │  ┌─────┐ ┌─────┐ ┌─────┐ │ ┌─────┐ ┌─────┐ ┌─────┐  │                     ║
║   │  │GTFS │ │ IoT │ │CCTV │ │ │User │ │3rd  │ │Infml│  │   DATA SOURCES     ║
║   │  │Feed │ │Sensr│ │Feed │ │ │Rpts │ │Party│ │Trnst│  │                     ║
║   │  └─────┘ └─────┘ └─────┘   └─────┘ └─────┘ └─────┘  │                     ║
║   └─────────────────────────────────────────────────────┘                     ║
║                                                                                ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## How to Use These Diagrams

### For GitHub/GitLab/VSCode:
The Mermaid diagrams will render automatically. Just view the markdown file.

### For Presentations (PowerPoint/Google Slides):
1. Go to [mermaid.live](https://mermaid.live)
2. Paste the Mermaid code
3. Export as PNG/SVG
4. Import into your slides

### For Draw.io/Excalidraw:
Use the ASCII diagram as a reference and recreate with drag-and-drop tools.

### For Hackathon Demo:
The sequence diagram (#5) is perfect for explaining the user flow during a live demo.

---

*Diagrams Version: 1.0*
*Compatible with: Mermaid 9.x+*
