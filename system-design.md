# SynthCommute: System Design Document

## Technical Architecture for Predictive Urban Mobility Platform

---

## 1. Executive Summary

SynthCommute is a cloud-native, AI-powered urban mobility intelligence platform that transforms fragmented transit data into predictive, personalized journey guidance. Unlike conventional navigation tools that provide deterministic routes, SynthCommute employs a **Probabilistic Journey Engine** that synthesizes five distinct data layers—official transit feeds, physical intelligence (crowd/queue sensors), human sensory network (crowdsourced reports), informal transit DNA (auto-rickshaws, shared taxis), and contextual overlays (weather, events)—to generate journey spectrums with confidence intervals. The system is designed for Indian urban scale: 50,000+ requests/second, sub-200ms latency, and graceful degradation when data feeds fail.

**Key Innovations:**
- First platform to integrate formal and informal transit networks
- Monte Carlo simulation-based uncertainty quantification
- Real-time platform reassignment detection via multi-signal fusion
- Dynamic transfer time estimation based on live crowd density
- Reinforcement learning-powered personalization

---

## 2. System Requirements

### 2.1 Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-01 | Multi-modal journey planning (train, bus, metro, auto, walk) | P0 |
| FR-02 | Real-time vehicle tracking and ETA updates | P0 |
| FR-03 | Probabilistic journey options with confidence scores | P0 |
| FR-04 | Push alerts for disruptions, platform changes, delays | P0 |
| FR-05 | Crowdsourced micro-event reporting and validation | P0 |
| FR-06 | Personal preference learning (crowd tolerance, stress avoidance) | P1 |
| FR-07 | Last-mile availability prediction (30-min lookahead) | P1 |
| FR-08 | City planner analytics dashboard | P1 |
| FR-09 | Transit operator real-time ops console | P2 |
| FR-10 | Historical journey analytics for users | P2 |

### 2.2 Non-Functional Requirements

| Metric | Target | Rationale |
|--------|--------|-----------|
| **Throughput** | 50,000 req/sec peak | Mumbai peak hour load |
| **Latency (P99)** | < 200ms | Real-time UX requirement |
| **Alert Latency** | < 30 seconds | Disruption must reach users before they commit |
| **Availability** | 99.9% | Critical daily utility |
| **Data Freshness** | < 60 seconds | Real-time relevance |
| **Prediction Accuracy** | > 90% | User trust threshold |
| **Horizontal Scale** | 10x in 30 mins | Handle demand spikes |

### 2.3 Data Requirements

| Data Type | Volume | Velocity | Variety |
|-----------|--------|----------|---------|
| GTFS Static | ~500 MB/city | Daily update | Structured |
| GTFS Realtime | ~10 KB/sec/city | 30-sec intervals | Structured |
| IoT Sensors | ~1 MB/sec/city | Real-time stream | Semi-structured |
| CCTV Metadata | ~500 KB/sec/city | Real-time stream | Unstructured→Structured |
| User Events | ~100 KB/sec/city | Real-time push | Semi-structured |
| Weather/Events | ~10 KB/sec | 5-min intervals | Structured |

---

## 3. High-Level Architecture

### 3.1 Eight-Tier Architecture Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        TIER 1: PRESENTATION LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                   │
│  │  Mobile App  │  │ Web Portal   │  │  Operator    │                   │
│  │ (React Native)│  │ (React)     │  │  Dashboard   │                   │
│  │ iOS/Android  │  │ Planner View │  │  (Vue.js)    │                   │
│  └──────────────┘  └──────────────┘  └──────────────┘                   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   TIER 2: API GATEWAY & EDGE LAYER                       │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  Kong/Envoy Gateway: Auth, Rate Limiting, Routing, WebSocket Mgmt  │ │
│  │  Edge Cache (Redis): Hot routes, recent predictions                │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   TIER 3: MICROSERVICES ECOSYSTEM                        │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │
│  │  Journey   │ │ Real-Time  │ │ Predictive │ │ Crowdsource│           │
│  │  Planner   │ │  Tracker   │ │  Engine    │ │  Service   │           │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘           │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐                          │
│  │   Alert    │ │   User     │ │   Data     │                          │
│  │  Service   │ │  Service   │ │  Ingestor  │                          │
│  └────────────┘ └────────────┘ └────────────┘                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   TIER 4: EVENT STREAMING PLATFORM                       │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  Apache Kafka Cluster + KSQL                                       │ │
│  │  Topics: transit.realtime, sensor.raw, user.events, anomaly.alerts │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 TIER 5: DATA PROCESSING & ML LAYER                       │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │
│  │   Stream   │ │   Batch    │ │  Feature   │ │   Model    │           │
│  │ Processing │ │ Processing │ │   Store    │ │  Serving   │           │
│  │  (Flink)   │ │  (Spark)   │ │  (Feast)   │ │ (TFServing)│           │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      TIER 6: STORAGE LAYER                               │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │
│  │Time-Series │ │   Graph    │ │  Document  │ │  Columnar  │           │
│  │ (InfluxDB) │ │  (Neo4j)   │ │ (MongoDB)  │ │ (BigQuery) │           │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 TIER 7: DATA SOURCES & INTEGRATION                       │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │  GTFS/  │ │   IoT   │ │  CCTV/  │ │  User   │ │Informal │          │
│  │ Realtime│ │ Sensors │ │  Vision │ │ Reports │ │ Transit │          │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 TIER 8: INFRASTRUCTURE & DEVOPS                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  Kubernetes (EKS/GKE) | Terraform | Prometheus/Grafana | ArgoCD    │ │
│  │  Multi-region | Edge nodes at stations | CDN (CloudFront)          │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Core Components Deep Dive

### 4.1 Data Ingestion Orchestrator

**Purpose:** Unified ingestion pipeline for heterogeneous data sources with quality scoring and privacy compliance.

```python
class DataIngestionOrchestrator:
    """
    Coordinates ingestion from 6 source types with different protocols.
    """

    SOURCES = {
        "gtfs_static": {
            "protocol": "HTTP_PULL",
            "frequency": "24h",
            "format": "GTFS_ZIP"
        },
        "gtfs_realtime": {
            "protocol": "HTTP_PULL",
            "frequency": "30s",
            "format": "PROTOBUF"
        },
        "iot_sensors": {
            "protocol": "MQTT_PUSH",
            "frequency": "real-time",
            "format": "JSON"
        },
        "cctv_feeds": {
            "protocol": "RTSP_STREAM",
            "frequency": "real-time",
            "format": "VIDEO→METADATA"
        },
        "user_reports": {
            "protocol": "WEBSOCKET_PUSH",
            "frequency": "real-time",
            "format": "JSON"
        },
        "informal_transit": {
            "protocol": "REST_POLL",
            "frequency": "60s",
            "format": "JSON"
        }
    }

    PIPELINE_STAGES = [
        "schema_validation",
        "anonymization",           # GDPR/PDP compliance
        "temporal_alignment",      # Timezone normalization
        "spatial_indexing",        # GeoHash conversion
        "quality_scoring",         # Data reliability metric
        "kafka_publish"            # Topic routing
    ]
```

**Quality Scoring Algorithm:**
```
quality_score = (
    0.3 * freshness_score +      # Time since generation
    0.3 * completeness_score +   # Required fields present
    0.2 * source_trust_score +   # Historical accuracy of source
    0.2 * consistency_score      # Alignment with other sources
)
```

### 4.2 Probabilistic Journey Engine

**Purpose:** Generate journey options with uncertainty quantification using Monte Carlo simulation.

```python
class ProbabilisticJourneyEngine:
    """
    Core brain: transforms route requests into probability distributions.
    """

    def generate_journey_spectrum(
        self,
        origin: GeoPoint,
        destination: GeoPoint,
        departure_time: datetime,
        user_profile: UserPreferences
    ) -> List[JourneyOption]:

        # Step 1: Generate candidate routes using graph search
        base_routes = self.graph_router.find_routes(
            origin, destination,
            modes=["train", "bus", "metro", "walk", "auto"],
            max_transfers=3
        )

        # Step 2: Monte Carlo simulation for each route
        journey_options = []
        for route in base_routes:
            simulations = []
            for _ in range(1000):  # 1000 Monte Carlo runs
                simulated_journey = self._simulate_journey(route, departure_time)
                simulations.append(simulated_journey)

            # Step 3: Compute statistics
            option = JourneyOption(
                route=route,
                eta_distribution=compute_distribution(simulations),
                confidence_interval=compute_ci(simulations, 0.95),
                crowd_prediction=self.crowd_forecaster.predict(route),
                stress_score=self._compute_stress(route, user_profile),
                risk_factors=self._identify_risks(simulations)
            )
            journey_options.append(option)

        # Step 4: Rank by user preferences
        return self._rank_by_preferences(journey_options, user_profile)

    def _simulate_journey(self, route: Route, time: datetime) -> SimulatedJourney:
        """
        Single Monte Carlo run with sampled uncertainties.
        """
        total_time = 0
        segments = []

        for segment in route.segments:
            # Sample from delay distribution
            delay = self.delay_predictor.sample(segment, time + total_time)

            # Sample from transfer time distribution
            if segment.has_transfer:
                transfer_time = self.transfer_estimator.sample(
                    segment.transfer_point,
                    time + total_time
                )

            # Sample from crowd distribution
            crowd_level = self.crowd_forecaster.sample(segment, time + total_time)

            segment_time = segment.base_time + delay + transfer_time
            total_time += segment_time
            segments.append(SimulatedSegment(segment, segment_time, crowd_level))

        return SimulatedJourney(segments, total_time)
```

### 4.3 Real-Time Anomaly Detector

**Purpose:** Detect platform changes, service disruptions, and cascade failures before official announcements.

```python
class AnomalyDetector:
    """
    Multi-signal fusion for disruption detection.
    """

    SIGNAL_WEIGHTS = {
        "crowd_flow_anomaly": 0.25,
        "audio_transcription": 0.20,
        "schedule_deviation": 0.20,
        "social_media_burst": 0.15,
        "device_clustering": 0.10,
        "official_feed_update": 0.10
    }

    def detect_platform_change(self, station_id: str) -> Optional[PlatformAlert]:
        """
        Fuse multiple weak signals to detect platform reassignment.
        """
        signals = {}

        # Signal 1: Crowd flow direction change
        signals["crowd_flow_anomaly"] = self.crowd_analyzer.detect_flow_change(
            station_id,
            expected_platform=self._get_expected_platform(station_id)
        )

        # Signal 2: Audio announcement transcription
        signals["audio_transcription"] = self.audio_processor.transcribe_and_match(
            station_id,
            keywords=["platform", "change", "shifted", "moved"]
        )

        # Signal 3: Schedule deviation (train not arriving at expected platform)
        signals["schedule_deviation"] = self.schedule_tracker.check_deviation(
            station_id
        )

        # Signal 4: Social media mentions
        signals["social_media_burst"] = self.social_listener.check_mentions(
            station_id,
            keywords=["platform change", "wrong platform"]
        )

        # Bayesian fusion
        probability = self._bayesian_fusion(signals)

        if probability > 0.85:
            return PlatformAlert(
                station_id=station_id,
                confidence=probability,
                predicted_platform=self._infer_new_platform(signals),
                evidence=signals
            )
        return None
```

### 4.4 Crowdsource Validator

**Purpose:** Validate user-submitted reports using trust scoring and consensus mechanisms.

```python
class CrowdsourceValidator:
    """
    Trust-scored crowdsourcing with consensus validation.
    """

    def validate_report(self, report: UserReport) -> ValidationResult:
        # Step 1: Check user trust score
        user_trust = self.trust_store.get_score(report.user_id)

        # Step 2: Check spatial-temporal plausibility
        plausibility = self._check_plausibility(report)

        # Step 3: Check consensus with other reports
        similar_reports = self._find_similar_reports(
            report,
            time_window=timedelta(minutes=10),
            spatial_radius=500  # meters
        )
        consensus_score = len(similar_reports) / self.CONSENSUS_THRESHOLD

        # Step 4: Compute final validation score
        validation_score = (
            0.4 * user_trust +
            0.3 * plausibility +
            0.3 * min(consensus_score, 1.0)
        )

        if validation_score > 0.7:
            # Accept and propagate
            self._propagate_to_system(report)
            self._update_user_trust(report.user_id, positive=True)
            return ValidationResult.ACCEPTED
        elif validation_score > 0.4:
            # Hold for more consensus
            return ValidationResult.PENDING
        else:
            # Reject
            self._update_user_trust(report.user_id, positive=False)
            return ValidationResult.REJECTED
```

### 4.5 Personalization Engine

**Purpose:** Learn user preferences through implicit feedback and reinforcement learning.

```python
class PersonalizationEngine:
    """
    RL-based preference learning from journey history.
    """

    PREFERENCE_DIMENSIONS = [
        "crowd_tolerance",      # 0 (avoid) to 1 (don't care)
        "standing_tolerance",   # 0 (must sit) to 1 (don't care)
        "time_vs_comfort",      # 0 (comfort) to 1 (speed)
        "predictability",       # 0 (risk-taking) to 1 (certainty)
        "cost_sensitivity",     # 0 (don't care) to 1 (minimize)
        "walking_tolerance"     # 0 (avoid) to 1 (prefer)
    ]

    def update_preferences(self, user_id: str, journey: CompletedJourney):
        """
        Update preference model based on implicit feedback.
        """
        # Implicit signals
        signals = {
            "selected_option_rank": journey.selected_rank,
            "deviated_from_suggestion": journey.deviated,
            "repeated_similar_choice": self._check_pattern(user_id, journey),
            "time_of_day": journey.departure_time.hour,
            "journey_duration": journey.actual_duration
        }

        # RL update (contextual bandit)
        self.model.update(
            state=self._encode_context(journey),
            action=journey.selected_option,
            reward=self._compute_reward(journey)
        )

    def _compute_reward(self, journey: CompletedJourney) -> float:
        """
        Reward = f(accuracy, user satisfaction signals)
        """
        accuracy_reward = 1.0 - abs(
            journey.predicted_duration - journey.actual_duration
        ) / journey.predicted_duration

        satisfaction_signal = (
            1.0 if not journey.deviated else 0.5
        )

        return 0.6 * accuracy_reward + 0.4 * satisfaction_signal
```

---

## 5. Data Architecture

### 5.1 Stream Processing Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Data Sources │───▶│    Kafka     │───▶│    Flink     │───▶│   Feature    │
│              │    │   Topics     │    │   Jobs       │    │    Store     │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                               │
                                               ▼
                    ┌──────────────────────────────────────────────────────┐
                    │                    Flink Jobs                         │
                    ├──────────────────────────────────────────────────────┤
                    │  1. crowd_density_calculator (5-min windows)         │
                    │  2. delay_propagation_tracker (event-driven)         │
                    │  3. transfer_time_estimator (30-sec updates)         │
                    │  4. anomaly_detector (pattern matching)              │
                    │  5. journey_tracker (per-user state)                 │
                    └──────────────────────────────────────────────────────┘
```

### 5.2 Feature Store Schema

```yaml
# Real-time features (< 1 minute freshness)
realtime_features:
  - name: "station_crowd_level"
    entity: "station_id"
    freshness: "30s"
    aggregation: "avg(device_count) over 5min"

  - name: "vehicle_delay_minutes"
    entity: "vehicle_id"
    freshness: "30s"
    source: "gtfs_realtime"

  - name: "transfer_walk_time_current"
    entity: "transfer_point_id"
    freshness: "60s"
    aggregation: "percentile_90(walk_times) over 10min"

  - name: "last_mile_availability_index"
    entity: "exit_point_id"
    freshness: "60s"
    computation: "available_autos / historical_avg"

# Batch features (daily refresh)
batch_features:
  - name: "station_crowd_pattern"
    entity: "station_id"
    granularity: "15min_buckets"
    lookback: "30days"

  - name: "route_reliability_score"
    entity: "route_id"
    computation: "on_time_arrivals / total_trips"
    lookback: "7days"
```

### 5.3 Database Selection Rationale

| Database | Use Case | Why This Choice |
|----------|----------|-----------------|
| **InfluxDB** | Sensor metrics, crowd density time-series | Optimized for time-series writes, downsampling, retention policies |
| **Neo4j** | Transit network graph, pathfinding | Native graph traversal, Cypher queries for multi-hop routes |
| **MongoDB** | User profiles, journey logs, crowdsource reports | Flexible schema, geospatial indexing, horizontal scaling |
| **BigQuery** | Historical analytics, model training data | Petabyte-scale, SQL interface, ML integration |
| **Redis** | Hot cache, session state, real-time leaderboards | Sub-millisecond reads, pub/sub for WebSocket fanout |

---

## 6. ML/AI Pipeline

### 6.1 Model Inventory

| Model | Architecture | Input | Output | Latency Target |
|-------|--------------|-------|--------|----------------|
| **ETA Predictor** | Physics-informed LSTM | Route segments, time, weather | Duration distribution | < 50ms |
| **Crowd Forecaster** | Temporal Fusion Transformer | Historical patterns, events, weather | Crowd level (0-1) | < 30ms |
| **Disruption Risk Scorer** | Bayesian Network | Multi-signal evidence | Risk probability | < 20ms |
| **Platform Change Detector** | Multi-modal Fusion (BERT + CNN) | Audio, video, schedule | Platform ID + confidence | < 100ms |
| **Personalization Agent** | Contextual Bandit (LinUCB) | User history, context | Preference weights | < 10ms |

### 6.2 MLOps Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Feature    │───▶│   Training   │───▶│   Model      │───▶│   Model      │
│   Pipeline   │    │   Pipeline   │    │   Registry   │    │   Serving    │
│   (Feast)    │    │   (Kubeflow) │    │   (MLflow)   │    │ (TFServing)  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                           │                                       │
                           │                                       ▼
                    ┌──────────────┐                       ┌──────────────┐
                    │   Experiment │                       │  A/B Testing │
                    │   Tracking   │                       │  & Monitoring│
                    │   (MLflow)   │                       │  (Evidently) │
                    └──────────────┘                       └──────────────┘
```

**Training Schedule:**
- ETA Predictor: Daily retraining with 30-day sliding window
- Crowd Forecaster: Weekly retraining
- Personalization Agent: Online learning (continuous)
- Platform Change Detector: Monthly retraining + fine-tuning on new stations

---

## 7. Key Workflows

### 7.1 User Journey Planning Flow

```
User                    API Gateway          Journey Planner        Predictive Engine
  │                          │                      │                      │
  │──── Request Journey ────▶│                      │                      │
  │                          │──── Authenticate ───▶│                      │
  │                          │                      │                      │
  │                          │                      │── Get User Profile ─▶│
  │                          │                      │                      │
  │                          │                      │── Get Live Context ─▶│
  │                          │                      │   (crowd, delays)    │
  │                          │                      │                      │
  │                          │                      │◀─ Context Response ──│
  │                          │                      │                      │
  │                          │                      │── Run Simulations ──▶│
  │                          │                      │   (1000 MC runs)     │
  │                          │                      │                      │
  │                          │                      │◀─ Journey Spectrum ──│
  │                          │                      │                      │
  │                          │◀─ Ranked Options ───│                      │
  │                          │                      │                      │
  │◀─── Journey Options ────│                      │                      │
  │     (with confidence)    │                      │                      │
```

### 7.2 Real-Time Alert Propagation Flow

```
Anomaly Detector        Alert Service         Kafka              Mobile App
       │                      │                 │                     │
       │── Anomaly Detected ─▶│                 │                     │
       │   (platform change)  │                 │                     │
       │                      │                 │                     │
       │                      │── Find Affected │                     │
       │                      │   Users ───────▶│                     │
       │                      │                 │                     │
       │                      │◀─ User List ───│                     │
       │                      │                 │                     │
       │                      │── Publish Alert▶│                     │
       │                      │                 │                     │
       │                      │                 │── Push Notification▶│
       │                      │                 │   (< 30 seconds)    │
       │                      │                 │                     │
       │                      │                 │                     │◀─ User Sees Alert
```

---

## 8. Edge Case Handling

### 8.1 Platform Reassignment Detection

```python
def detect_platform_reassignment(station_id, train_id):
    """
    Multi-signal fusion when official feeds lag.
    """
    evidence = {
        # Signal 1: Crowd flow reversed direction
        "crowd_reversal": crowd_analyzer.detect_reversal(
            station_id,
            expected_direction=get_expected_direction(station_id, train_id)
        ),

        # Signal 2: Audio announcement keyword detection
        "audio_keyword": audio_processor.detect_keywords(
            station_id,
            keywords=["platform", "change", str(train_id)]
        ),

        # Signal 3: Device clustering on different platform
        "device_clustering": device_tracker.detect_cluster_shift(
            station_id,
            from_platform=expected_platform,
            to_platform=None  # Detect automatically
        ),

        # Signal 4: Schedule shows arrival but no vehicle detected
        "phantom_arrival": schedule_tracker.detect_phantom(
            station_id, train_id
        )
    }

    # Bayesian network inference
    probability, new_platform = bayesian_network.infer(evidence)

    if probability > 0.85:
        return Alert(
            type="PLATFORM_CHANGE",
            confidence=probability,
            new_platform=new_platform,
            message=f"Platform likely shifted to {new_platform}. {int(probability*100)}% confidence."
        )
```

### 8.2 Graceful Degradation

```python
class GracefulDegradation:
    """
    Fallback strategies when data feeds fail.
    """

    FALLBACK_CHAIN = {
        "gtfs_realtime": [
            "historical_pattern_at_time",
            "static_schedule_with_buffer",
            "user_reports_consensus"
        ],
        "crowd_sensors": [
            "historical_crowd_pattern",
            "proxy_from_social_media",
            "assume_peak_if_rush_hour"
        ],
        "informal_transit": [
            "historical_availability",
            "weather_adjusted_estimate",
            "show_walking_alternative"
        ]
    }

    def get_with_fallback(self, data_type, location, time):
        primary = self.fetch_primary(data_type, location, time)
        if primary.is_valid():
            return primary

        for fallback in self.FALLBACK_CHAIN[data_type]:
            result = self.fetch_fallback(fallback, location, time)
            if result.is_valid():
                result.mark_as_fallback(fallback)
                return result

        return UnavailableData(reason="All sources failed")
```

---

## 9. Scalability & Resilience

### 9.1 Horizontal Scaling Strategy

| Component | Scaling Trigger | Scale Unit | Max Scale |
|-----------|-----------------|------------|-----------|
| API Gateway | CPU > 70% | +2 pods | 50 pods |
| Journey Planner | Request queue > 1000 | +4 pods | 100 pods |
| Flink Jobs | Kafka lag > 10s | +1 taskmanager | 20 taskmanagers |
| Model Serving | Latency P99 > 150ms | +2 replicas | 30 replicas |

### 9.2 Geographic Distribution

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MULTI-REGION DEPLOYMENT                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   REGION: MUMBAI (Primary)              REGION: DELHI (Secondary)        │
│   ┌─────────────────────┐               ┌─────────────────────┐         │
│   │  K8s Cluster        │               │  K8s Cluster        │         │
│   │  - All services     │               │  - Read replicas    │         │
│   │  - Primary DBs      │◀────Sync─────▶│  - Replica DBs      │         │
│   │  - Model serving    │               │  - Model serving    │         │
│   └─────────────────────┘               └─────────────────────┘         │
│              │                                    │                      │
│              ▼                                    ▼                      │
│   ┌─────────────────────┐               ┌─────────────────────┐         │
│   │  Edge Nodes         │               │  Edge Nodes         │         │
│   │  - Churchgate       │               │  - New Delhi        │         │
│   │  - Dadar            │               │  - Connaught Place  │         │
│   │  - Andheri          │               │  - Rajiv Chowk      │         │
│   └─────────────────────┘               └─────────────────────┘         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 9.3 Circuit Breaker Pattern

```python
class CircuitBreaker:
    """
    Prevent cascade failures when dependencies fail.
    """

    STATES = ["CLOSED", "OPEN", "HALF_OPEN"]

    def __init__(self, failure_threshold=5, recovery_timeout=30):
        self.state = "CLOSED"
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.last_failure_time = None

    def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = "HALF_OPEN"
            else:
                raise CircuitOpenError("Service unavailable")

        try:
            result = func(*args, **kwargs)
            if self.state == "HALF_OPEN":
                self.state = "CLOSED"
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
            raise e
```

---

## 10. Security & Privacy

### 10.1 Privacy Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          PRIVACY-BY-DESIGN                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   RAW DATA          ANONYMIZATION         AGGREGATION        ANALYTICS   │
│   ┌────────┐        ┌────────────┐       ┌───────────┐      ┌────────┐  │
│   │ Device │───────▶│ Strip PII  │──────▶│ Aggregate │─────▶│ Publish│  │
│   │ Pings  │        │ GeoHash(6) │       │ by Zone   │      │ Metrics│  │
│   └────────┘        │ Round Time │       │ (100m²)   │      └────────┘  │
│                     └────────────┘       └───────────┘                   │
│                           │                                              │
│                           ▼                                              │
│                     ┌────────────┐                                       │
│                     │ TTL: 24hrs │                                       │
│                     │ Then DELETE│                                       │
│                     └────────────┘                                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 10.2 Security Controls

| Layer | Control | Implementation |
|-------|---------|----------------|
| **API** | Authentication | OAuth 2.0 + JWT |
| **API** | Rate Limiting | 100 req/min per user |
| **Transport** | Encryption | TLS 1.3 everywhere |
| **Data** | Encryption at Rest | AES-256 |
| **Access** | RBAC | K8s ServiceAccounts + IAM |
| **Audit** | Logging | All API calls logged |
| **Compliance** | Certifications | ISO 27001, SOC 2 (roadmap) |

---

## 11. Implementation Roadmap

### Phase 1: MVP (Months 1-3)
- Core journey planning with official GTFS data
- Basic crowd prediction (historical patterns)
- Mobile app (Android/iOS) for Mumbai suburban rail
- Single-region deployment

**Success Criteria:** 10,000 DAU, 85% prediction accuracy

### Phase 2: Intelligence (Months 4-6)
- Real-time anomaly detection
- Informal transit integration (auto-rickshaws)
- Crowdsourcing with trust scoring
- Expand to Delhi, Bangalore

**Success Criteria:** 100,000 DAU, 90% accuracy, <30s alert latency

### Phase 3: Scale (Months 7-12)
- Full ML suite operational
- Predictive disruption management
- City planner dashboard
- API marketplace for transit operators
- 10+ cities

**Success Criteria:** 1M DAU, B2B contracts with 3 transit agencies

---

## Appendix A: Technology Stack Summary

| Layer | Technology | Version |
|-------|------------|---------|
| Mobile | React Native | 0.72+ |
| Web | React + Next.js | 14+ |
| API Gateway | Kong | 3.4+ |
| Backend | Python (FastAPI) + Go | 3.11+ / 1.21+ |
| Streaming | Apache Kafka | 3.5+ |
| Stream Processing | Apache Flink | 1.17+ |
| ML Serving | TensorFlow Serving | 2.13+ |
| Feature Store | Feast | 0.35+ |
| Time-Series DB | InfluxDB | 2.7+ |
| Graph DB | Neo4j | 5.x |
| Document DB | MongoDB | 7.0+ |
| Cache | Redis | 7.2+ |
| Orchestration | Kubernetes | 1.28+ |
| IaC | Terraform | 1.5+ |
| CI/CD | ArgoCD + GitHub Actions | Latest |
| Monitoring | Prometheus + Grafana | Latest |

---

*Document Version: 1.0*
*Last Updated: February 2026*
*Authors: SynthCommute Engineering Team*
