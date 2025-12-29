# HSV and HSI Specification

**Human State Vector (HSV)** is the core vector representation layer of Synheart Core.

## Terminology

The Synheart Core SDK uses two complementary representations:

- **HSV (Human State Vector)**: An internal, time-scoped, multi-dimensional representation that encodes estimates of human physiological, cognitive, and behavioral state for local computation and inference
  - Language-agnostic state representation implemented in each platform's native types
  - Dart: Classes with strong typing
  - Kotlin: Data classes with null safety
  - Swift: Structs with value semantics
  - Purpose: Fast, type-safe on-device processing

- **HSI 1.0 (Human State Interface)**: Cross-platform JSON format for interoperability
  - Format: JSON validated against canonical schema
  - Purpose: Wire format for external systems, cloud storage, cross-platform communication

**Key Insight**: HSV represents the same conceptual state space across all platforms, while HSI 1.0 provides a common serialization format.

This document describes **both** HSV (language-agnostic state representation) and HSI 1.0 (JSON wire format) specifications.

---

## Core Principle

> **HSV represents human state.**
>
> **Interpretation is downstream and optional.**

HSV does NOT produce:
- emotion labels
- focus scores
- cognitive assessments
- semantic interpretations

HSV produces structured, machine-readable state representations that downstream modules MAY interpret.
HSV can be exported to HSI 1.0 format for external systems.

---

## Architecture

```
Signal Sources → HSV Runtime → HSV (Internal Representation)
                                    ↓
                          Optional Interpretation Modules
                                    ↓
                          Optional: Export to HSI 1.0 (External)
```

### Signal Sources
- **Wear Module**: HR, HRV, sleep stages, motion (derived only)
- **Phone Module**: device motion, screen state, app context (hashed)
- **Behavior Module**: taps, scrolls, typing cadence, idle patterns

### HSV Runtime
- Multimodal fusion engine
- Temporal aggregation
- State embedding generation
- Privacy enforcement
- Produces HSV (Human State Vector)

---

## HSV Outputs (Internal Representation)

HSV produces three types of outputs:

### 1. State Axes & Indices

Numerical representations of physiological and behavioral state dimensions.

| **Axis** | **Index** | **Range** | **Description** |
|----------|-----------|-----------|-----------------|
| **Affect** | `arousalIndex` | 0.0 - 1.0 | Physiological arousal level |
| **Affect** | `valenceStability` | 0.0 - 1.0 | Stability of affective state |
| **Engagement** | `engagementStability` | 0.0 - 1.0 | Consistency of interaction patterns |
| **Engagement** | `interactionCadence` | 0.0 - 1.0 | Rhythm of digital interactions |
| **Activity** | `motionIndex` | 0.0 - 1.0 | Physical activity level |
| **Activity** | `postureStability` | 0.0 - 1.0 | Postural stability from motion |
| **Context** | `screenActiveRatio` | 0.0 - 1.0 | Screen on/off ratio |
| **Context** | `sessionFragmentation` | 0.0 - 1.0 | App switching frequency |

**Properties:**
- All indices are normalized to [0.0, 1.0]
- Indices are computed per time window
- Missing signals result in `null` values (not 0.0)
- Privacy-safe: no raw content, identifiers, or semantic data

Axes define conceptual dimensions; indices are normalized scalar realizations computed per time window.

---

### 2. Time Windows

HSV Runtime computes state representations across multiple temporal scales:

| **Window** | **Duration** | **Update Frequency** | **Use Case** |
|------------|--------------|----------------------|--------------|
| **micro** | 30 seconds | Every 30s | Real-time state monitoring |
| **short** | 5 minutes | Every 1-2 minutes | Session-level patterns |
| **medium** | 1 hour | Every 5-10 minutes | Activity trends |
| **long** | 24 hours | Every 30-60 minutes | Daily rhythms |

**Window Structure:**
```json
{
  "windowType": "short",
  "duration": 300,
  "timestamp": 1704067200,
  "axes": {
    "affect": {
      "arousalIndex": 0.72,
      "valenceStability": 0.85
    },
    "engagement": {
      "engagementStability": 0.68,
      "interactionCadence": 0.54
    },
    "activity": {
      "motionIndex": 0.42,
      "postureStability": 0.91
    },
    "context": {
      "screenActiveRatio": 0.83,
      "sessionFragmentation": 0.31
    }
  }
}
```

---

### 3. State Embeddings

**64-dimensional dense vector** representing fused multimodal state.

**Properties:**
- Dimension: 64D float32 vector
- Normalized: unit norm (L2 normalization)
- Temporal resolution: computed per window
- Privacy-preserving: non-invertible representation
 - Opaque: not intended for direct semantic interpretation without downstream models

**Structure:**
```json
{
  "embedding": {
    "vector": [0.12, -0.34, 0.56, ..., 0.78],
    "dimension": 64,
    "model": "hsi-fusion-v1",
    "timestamp": 1704067200,
    "windowType": "short"
  }
}
```

**Use Cases:**
- Temporal pattern analysis
- State similarity comparison
- Anomaly detection
- Clustering and segmentation
- Input to interpretation modules

---

## HSV API (Internal Representation)

### Initialization

```dart
await Synheart.initialize(
  userId: 'anon_user_123',
  config: SynheartConfig(
    enableWear: true,
    enablePhone: true,
    enableBehavior: true,
  ),
);
```

### Subscribe to HSV Updates

```dart
// Subscribe to HSV (internal representation)
Synheart.onHSVUpdate.listen((hsv) {
  // State axes & indices
  print('Arousal: ${hsv.meta.axes.affect.arousalIndex}');
  print('Engagement: ${hsv.meta.axes.engagement.engagementStability}');

  // Time window metadata
  print('Window: ${hsv.meta.embedding.windowType}');

  // State embedding
  print('Embedding: ${hsv.meta.embedding.vector}');
});
```

### HSV Object Structure (Internal)

```dart
class HumanStateVector {
  String version;
  int timestamp;

  // Base HSV (core state, always present)
  MetaState meta;

  // Optional enrichments (derived, opt-in; do not alter base HSV computation)
  EmotionState? emotion;
  FocusState? focus;

  // Behavioral and context data
  BehaviorState behavior;
  ContextState context;
}

class MetaState {
  String sessionId;
  DeviceInfo device;
  double samplingRateHz;

  // State axes
  HSIAxes axes;

  // State embedding
  StateEmbedding embedding;
}

class HSIAxes {
  AffectAxis affect;
  EngagementAxis engagement;
  ActivityAxis activity;
  ContextAxis context;
}

class AffectAxis {
  double? arousalIndex;
  double? valenceStability;
}

class EngagementAxis {
  double? engagementStability;
  double? interactionCadence;
}

class ActivityAxis {
  double? motionIndex;
  double? postureStability;
}

class ContextAxis {
  double? screenActiveRatio;
  double? sessionFragmentation;
}

class StateEmbedding {
  List<double> vector;        // 64D float32
  int dimension;              // always 64
  String model;               // "hsi-fusion-v1"
  int timestamp;
  String windowType;
}
```

---

## HSI 1.0 Export (External Format)

### Purpose

HSI 1.0 is the canonical JSON format for exporting HSV to external systems, enabling cross-platform interoperability.

### When to Use

- **HSV (Internal)**: On-device processing, real-time UI, app logic
- **HSI 1.0 (External)**: Cloud storage, external APIs, cross-system integration, validation against schema

### Export API

```dart
// Convert HSV to HSI 1.0 format
final hsi10 = hsv.toHSI10(
  producerName: 'My App',
  producerVersion: '1.0.0',
  // HSI 1.0 schema: UUID string
  instanceId: '0b6f3ac9-62f5-4c9f-9f0d-4c4b3f6b2a3b',
);

// Serialize to JSON
final json = hsi10.toJson();
```

### HSI 1.0 Payload Structure

```json
{
  "hsi_version": "1.0",
  "observed_at_utc": "2025-12-28T00:00:10Z",
  "computed_at_utc": "2025-12-28T00:00:11Z",
  "producer": {
    "name": "My App",
    "version": "1.0.0",
    "instance_id": "0b6f3ac9-62f5-4c9f-9f0d-4c4b3f6b2a3b"
  },
  "window_ids": ["w1"],
  "windows": {
    "w1": {
      "start": "2025-12-27T23:59:40Z",
      "end": "2025-12-28T00:00:10Z",
      "label": "micro"
    }
  },
  "axes": {
    "affect": {
      "readings": [
        {
          "axis": "arousal",
          "score": 0.72,
          "confidence": 0.8,
          "window_id": "w1",
          "direction": "higher_is_more"
        }
      ]
    },
    "engagement": {
      "readings": [
        {
          "axis": "engagement_stability",
          "score": 0.68,
          "confidence": 0.8,
          "window_id": "w1",
          "direction": "higher_is_more"
        }
      ]
    },
    "behavior": {
      "readings": [
        {
          "axis": "interaction_intensity",
          "score": 0.54,
          "confidence": 0.8,
          "window_id": "w1",
          "direction": "higher_is_more"
        }
      ]
    }
  },
  "embeddings": [
    {
      "vector": [0.12, -0.34, ...],
      "dimension": 64,
      "encoding": "float32",
      "confidence": 0.85,
      "window_id": "w1",
      "model": "hsi-fusion-v1"
    }
  ],
  "privacy": {
    "contains_pii": false,
    "raw_biosignals_allowed": false,
    "derived_metrics_allowed": true
  }
}
```

### Contract Notes (HSI 1.0)

- HSI 1.0 is a **contract-only** payload defined by the canonical schema and RFC in the `synheart/hsi` repository.
- `producer.instance_id` is a **UUID** (RFC 3339 timestamps + UUID formats are schema-validated).
- Axis readings are grouped into these top-level domains under `axes`:
  - `axes.affect.readings[]`
  - `axes.engagement.readings[]`
  - `axes.behavior.readings[]`
- Each reading’s `axis` name is `lower_snake_case`.
- A reading MAY include `score: null` when unavailable (e.g., access control / missing sources), but `null` **must not** be interpreted as 0.0 and should include an explanatory `meta` entry (strict validation).

### Schema Validation

HSI 1.0 payloads can be validated against the canonical JSON Schema:
- **Schema Location**: `/hsi/schema/hsi-1.0.schema.json`
- **JSON Schema Version**: Draft 2020-12
- **Validation**: Required for external system integration

### Hybrid Architecture Benefits

- **HSV**: Fast on-device processing with type safety
- **HSI 1.0**: Standards-compliant external format
- **Conversion**: One-way export from HSV → HSI 1.0
- **Flexibility**: Use HSV internally, export HSI 1.0 when needed

---

## Privacy & Security

HSV enforces strict privacy guarantees:

### What HSV/HSI DOES NOT expose:
- Raw biosignals (ECG, PPG)
- Message content or URLs
- Specific app names
- User identifiers
- Keyboard content
- Audio or microphone data

### What HSV/HSI DOES expose:
- Derived physiological indices (HR, HRV aggregates)
- Behavioral patterns (interaction timing, not content)
- Hashed app context (non-reversible)
- Temporal state representations

### Consent Enforcement:
- All modules require explicit consent
- Consent can be revoked at any time
- Missing consent = `null` values, not synthetic data
- Cloud upload requires separate consent

---

## Fusion Model

The HSV runtime applies multimodal fusion models to generate Human State Vectors (HSVs) from heterogeneous signals.

**Architecture:**
```
Wear Signals  →  Feature Extractor  →
Phone Signals →  Feature Extractor  →  Fusion Layer  →  HSV Output
Behavior Data →  Feature Extractor  →
```

**Fusion Approach:**
- Attention-based multimodal fusion
- Temporal convolution for smoothing
- Missing modality handling (graceful degradation)
- Adaptive weighting based on signal quality

**Model Properties:**
- On-device inference
- Latency: < 100ms
- Model size: < 10MB
- CPU usage: < 2%

---

## Performance Targets

| **Metric** | **Target** | **Measurement** |
|------------|------------|-----------------|
| HSV update latency | ≤ 100ms | 95th percentile |
| Memory footprint | < 15MB | Resident set size |
| CPU usage | < 2% | Average over 1 hour |
| Battery impact | < 0.5%/hr | Device battery drain |
| Missing sample rate | < 5%/day | Data completeness |

---

## Versioning

### HSV (Internal)

HSV follows semantic versioning tied to SDK releases.

**Breaking changes** (major version bump):
- Changes to axis names or ranges
- Removal of axes or indices
- Embedding dimension changes
- API signature changes

**Non-breaking changes** (minor/patch):
- Addition of new axes
- Improved fusion model
- Performance optimizations
- Bug fixes

### HSI 1.0 (External)

HSI uses independent versioning for the canonical format.

**Current Version:** `1.0`

- HSI version is specified in `hsi_version` field of payload
- Breaking changes to HSI format increment major version (e.g., 1.0 → 2.0)
- HSV can export to multiple HSI versions for backward compatibility

---

## Interpretation Modules

HSV provides the foundation for optional interpretation modules that add semantic meaning to state representations.

### Design Principle

> **HSV represents human state. Interpretation is downstream and optional.**

Interpretation modules:
- ✅ Consume HSV outputs (axes, embeddings, metadata)
- ✅ Provide semantic interpretations (emotion labels, focus scores)
- ✅ Are explicitly enabled by applications
- ✅ Operate independently of HSI export (serialization) layer
- ❌ Do NOT affect HSV core state representation

### Available Interpretation Modules

#### EmotionHead

**Purpose:** Infers emotional state from physiological signals

**Implementation:** Uses [synheart-emotion](https://github.com/synheart-ai/synheart-emotion) SDK

**Input:** HR/RR intervals from Wear Module + HSV context

**Output Schema:**
```dart
class EmotionState {
  double stress;        // 0.0 - 1.0
  double calm;          // 0.0 - 1.0
  double engagement;    // 0.0 - 1.0 (amused)
  double activation;    // 0.0 - 1.0 (derived)
  double valence;       // -1.0 to 1.0 (derived)
}
```

**Emotion Categories:**
- **Stressed**: High arousal, negative valence
- **Calm**: Low arousal, neutral/positive valence
- **Amused**: High arousal, positive valence

**Usage:**
```dart
await Synheart.enableEmotion();

Synheart.onEmotionUpdate.listen((emotion) {
  print('Stress: ${emotion.stress}');
  print('Calm: ${emotion.calm}');
});
```

**Schema Validation:**
- EmotionResult from synheart-emotion maps to EmotionState
- Validated against this HSI specification via CI
- Breaking changes require coordinated releases

---

#### FocusHead

**Purpose:** Estimates cognitive concentration and engagement

**Implementation:** Uses [synheart-focus](https://github.com/synheart-ai/synheart-focus) SDK

**Input:** Multimodal HSV (biosignals + behavior + context)

**Output Schema:**
```dart
class FocusState {
  double score;           // 0.0 - 1.0 (overall focus)
  double cognitiveLoad;   // 0.0 - 1.0
  double clarity;         // 0.0 - 1.0
  double distraction;     // 0.0 - 1.0
}
```

**Usage:**
```dart
await Synheart.enableFocus();

Synheart.onFocusUpdate.listen((focus) {
  print('Focus Score: ${focus.score}');
  print('Cognitive Load: ${focus.cognitiveLoad}');
});
```

**Schema Validation:**
- FocusResult from synheart-focus maps to FocusState
- Validated against this specification via CI

---

### Integration Architecture

```
┌──────────────────────────────────────────────────┐
│          Synheart Core (HSV Runtime)             │
│                                                  │
│  Wear Module → HSV Runtime → Base HSV           │
│                                    │             │
│                                    ├─► EmotionHead (synheart-emotion)
│                                    │    └─► EmotionState
│                                    │             │
│                                    └─► FocusHead (synheart-focus)
│                                         └─► FocusState
│                                                  │
└──────────────────────────────────────────────────┘
                      │
                      ▼
              Enriched HSV (Base HSV + interpretations)
```

**Data Flow:**
1. HSV Runtime produces Base HSV (axes, embeddings)
2. EmotionHead/FocusHead subscribe to HSV stream (if enabled)
3. Heads use synheart-emotion/focus SDKs for inference
4. Results mapped to HSV-compatible schemas
5. Enriched HSV emitted (Base HSV + appended annotations)
6. Optional: Export to HSI 1.0 for external systems

---

### Dependency Management

**Runtime Dependencies:**
```yaml
# synheart-core-dart/pubspec.yaml
dependencies:
  synheart_emotion: ^0.2.3  # EmotionHead implementation
  synheart_focus: ^0.1.0    # FocusHead implementation
```

**Schema Compatibility:**
- synheart-emotion validates against this spec
- synheart-focus validates against this spec
- CI fails if schema incompatibilities detected
- Version matrix maintained in respective SDK READMEs

---

### Adding New Interpretation Modules

To add a new interpretation module:

1. **Create standalone SDK** (e.g., synheart-sleep)
2. **Define output schema** compatible with HSV
3. **Add to this specification** with schema documentation
4. **Implement Head module** in synheart-core (EmotionHead pattern)
5. **Add dependency** to platform SDKs (Dart, Kotlin, Swift)
6. **Add schema validation** CI check
7. **Update documentation** across all repos

**Requirements:**
- ✅ Standalone SDK can work without synheart-core
- ✅ Output schema validated against this specification
- ✅ Head module subscribes to HSV stream
- ✅ Results appended as annotations to produce Enriched HSV (not modifying Base HSV)
- ✅ Explicitly enabled by applications
- ✅ Graceful degradation if module unavailable

---

## Related Documentation

- [PRD](PRD.md) - Product requirements
- [Capability System](CAPABILITY_SYSTEM.md) - Access level enforcement
- [Consent System](CONSENT_SYSTEM.md) - Permission model
- [Cloud Protocol](CLOUD_PROTOCOL.md) - Cloud upload specification

---

**Last Updated:** 2025-12-25
**Version:** 1.0.0
**Maintained by:** Synheart AI
