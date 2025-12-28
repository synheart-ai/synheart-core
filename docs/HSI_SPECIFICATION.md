# HSI Specification

**Human State Interface (HSI)** is the core state representation layer of Synheart Core.

HSI provides a stable, interpretation-agnostic interface for representing human physiological, behavioral, and contextual state.

---

## Core Principle

> **HSI represents human state.**
>
> **Interpretation is downstream and optional.**

HSI does NOT produce:
- emotion labels
- focus scores
- cognitive assessments
- semantic interpretations

HSI produces structured, machine-readable state representations that downstream modules MAY interpret.

---

## Architecture

```
Signal Sources → HSI Runtime → State Representation
                                    ↓
                          Optional Interpretation Modules
```

### Signal Sources
- **Wear Module**: HR, HRV, sleep stages, motion (derived only)
- **Phone Module**: device motion, screen state, app context (hashed)
- **Behavior Module**: taps, scrolls, typing cadence, idle patterns

### HSI Runtime
- Multimodal fusion engine
- Temporal aggregation
- State embedding generation
- Privacy enforcement

---

## HSI Outputs

HSI produces three types of outputs:

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

---

### 2. Time Windows

HSI computes state representations across multiple temporal scales:

| **Window** | **Duration** | **Update Frequency** | **Use Case** |
|------------|--------------|----------------------|--------------|
| **Micro** | 30 seconds | Every 30s | Real-time state monitoring |
| **Short** | 5 minutes | Every 1-2 minutes | Session-level patterns |
| **Medium** | 1 hour | Every 5-10 minutes | Activity trends |
| **Long** | 24 hours | Every 30-60 minutes | Daily rhythms |

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

## HSI API

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

### Subscribe to HSI Updates

```dart
Synheart.onHSIUpdate.listen((hsi) {
  // State axes & indices
  print('Arousal: ${hsi.affect.arousalIndex}');
  print('Engagement: ${hsi.engagement.engagementStability}');

  // Time window metadata
  print('Window: ${hsi.windowType}, Duration: ${hsi.duration}s');

  // State embedding
  print('Embedding: ${hsi.embedding.vector}');
});
```

### HSI Object Structure

```dart
class HSI {
  // Metadata
  String windowType;          // "micro", "short", "medium", "long"
  int duration;               // window duration in seconds
  int timestamp;              // Unix timestamp

  // State axes
  AffectAxis affect;
  EngagementAxis engagement;
  ActivityAxis activity;
  ContextAxis context;

  // Embedding
  StateEmbedding embedding;
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

## Privacy & Security

HSI enforces strict privacy guarantees:

### What HSI DOES NOT expose:
- Raw biosignals (ECG, PPG)
- Message content or URLs
- Specific app names
- User identifiers
- Keyboard content
- Audio or microphone data

### What HSI DOES expose:
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

HSI Runtime uses a multimodal fusion model to combine signals.

**Architecture:**
```
Wear Signals  →  Feature Extractor  →
Phone Signals →  Feature Extractor  →  Fusion Layer  →  HSI Output
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
| HSI update latency | ≤ 100ms | 95th percentile |
| Memory footprint | < 15MB | Resident set size |
| CPU usage | < 2% | Average over 1 hour |
| Battery impact | < 0.5%/hr | Device battery drain |
| Missing sample rate | < 5%/day | Data completeness |

---

## Versioning

HSI follows semantic versioning: `MAJOR.MINOR.PATCH`

**Version:** `1.0.0`

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

---

## Interpretation Modules

HSI provides the foundation for optional interpretation modules that add semantic meaning to state representations.

### Design Principle

> **HSI represents human state. Interpretation is downstream and optional.**

Interpretation modules:
- ✅ Consume HSI outputs (axes, embeddings, metadata)
- ✅ Provide semantic interpretations (emotion labels, focus scores)
- ✅ Are explicitly enabled by applications
- ✅ Operate independently of HSI Runtime
- ❌ Do NOT affect HSI core state representation

### Available Interpretation Modules

#### EmotionHead

**Purpose:** Infers emotional state from physiological signals

**Implementation:** Uses [synheart-emotion](https://github.com/synheart-ai/synheart-emotion) SDK

**Input:** HR/RR intervals from Wear Module + HSI context

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

**Input:** Multimodal HSI (biosignals + behavior + context)

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
- Validated against this HSI specification via CI

---

### Integration Architecture

```
┌──────────────────────────────────────────────────┐
│          Synheart Core (HSI Runtime)             │
│                                                  │
│  Wear Module → HSI Runtime → HSV (base state)   │
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
              Complete HSV with interpretations
```

**Data Flow:**
1. HSI Runtime produces base HSV (axes, embeddings)
2. EmotionHead/FocusHead subscribe to HSV stream (if enabled)
3. Heads use synheart-emotion/focus SDKs for inference
4. Results mapped to HSI-compatible schemas
5. Updated HSV emitted with interpretation data

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
- synheart-emotion validates against this spec (HSI_SPECIFICATION.md)
- synheart-focus validates against this spec
- CI fails if schema incompatibilities detected
- Version matrix maintained in respective SDK READMEs

---

### Adding New Interpretation Modules

To add a new interpretation module:

1. **Create standalone SDK** (e.g., synheart-sleep)
2. **Define output schema** compatible with HSI
3. **Add to this specification** with schema documentation
4. **Implement Head module** in synheart-core (EmotionHead pattern)
5. **Add dependency** to platform SDKs (Dart, Kotlin, Swift)
6. **Add schema validation** CI check
7. **Update documentation** across all repos

**Requirements:**
- ✅ Standalone SDK can work without synheart-core
- ✅ Output schema validated against HSI specification
- ✅ Head module subscribes to HSV stream
- ✅ Results enriched into HSV (not replacing base state)
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
