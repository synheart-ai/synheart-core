# RFC-0001: Human State Vector (HSV)

- **Status**: Draft
- **Author**: Synheart Research & Engineering Team
- **Last Updated**: 2025-12-28
- **Target**: Synheart Core v1.x
- **Related Specs**: HSI 1.0, Wear Module RFCs

---

## 1. Abstract

Human State Vector (HSV) is an internal, time-scoped, multi-dimensional representation of human physiological, cognitive, and behavioral state. HSV serves as the core representation layer within Synheart Core, enabling real-time state computation, fusion, and downstream interpretation while remaining independent of external serialization formats and application-specific semantics.

HSV is designed as a pre-interpretive representation: it encodes normalized state dimensions and embeddings without producing semantic labels such as emotions, focus levels, or cognitive assessments. These interpretations, if required, are performed by optional downstream modules.

---

## 2. Motivation

Most human-aware systems tightly couple sensing, inference, and interpretation, leading to brittle architectures, limited interoperability, and privacy risks. HSV introduces a clear separation between:

- State representation (what is measured and when)
- Interpretation (what it means)
- Interface exposure (how it is shared)

By isolating representation as a first-class layer, HSV enables:

- Type-safe on-device computation
- Model and sensor evolution without API breakage
- Privacy-by-design human state systems
- Stable external interfaces (via HSI)

---

## 3. Scope and Non-Goals

### In Scope

- Definition of HSV as an internal state representation
- Temporal and dimensional properties of HSV
- Relationship to HSI and interpretation modules

### Out of Scope

- External wire formats (covered by HSI)
- Model architectures or training procedures
- Semantic interpretation (emotion, focus, intent)
- Raw biosignal access

---

## 4. Definition

Human State Vector (HSV) is an internal, implementation-defined representation that encodes normalized estimates of human physiological, behavioral, and contextual state over explicit time windows.

### Key Properties

- Internal-only: Not exposed directly to external systems
- Time-scoped: Always associated with a defined window
- Multi-dimensional: Multiple state axes may coexist
- Partial: Missing data is represented as null, not inferred
- Privacy-preserving: No raw signals or semantic content

---

## 5. Architectural Role

```
Signal Sources
  (wear, phone, behavior)
        ↓
Feature Extraction
        ↓
HSV Runtime
  (fusion + aggregation)
        ↓
HSV (Internal State)
        ↓
Optional Interpretation Modules
        ↓
Optional Export → HSI 1.0
```

HSV is the boundary between sensing and meaning.

---

## 6. Representation Model

HSV consists of three core components:

### 6.1 State Axes and Indices

Axes define conceptual dimensions (e.g., affect, engagement).
Indices are normalized scalar values computed per window.

#### Constraints

- Range: [0.0, 1.0]
- Nullable if insufficient signal
- Window-specific
- Confidence handled separately

---

### 6.2 Temporal Windows

HSV values are always computed over explicit windows.

| Window Class | Typical Duration | Purpose |
|---|---:|---|
| `micro` | ~30s | Real-time monitoring |
| `short` | ~5m | Session-level patterns |
| `medium` | ~1h | Behavioral trends |
| `long` | ~24h | Circadian rhythms |

Window definitions are implementation-defined but must be explicit.
Canonical labels for interoperability (e.g., when exporting to HSI 1.0) are `micro`, `short`, `medium`, `long`.

---

### 6.3 State Embeddings

HSV may include dense, opaque embeddings representing fused multimodal state.

#### Properties

- Fixed dimensionality (e.g., 64D)
- Normalized (e.g., L2)
- Non-invertible
- Not semantically interpretable without downstream models

Embeddings are optional but recommended for advanced analysis.

---

## 7. Implementation Considerations (Non-Normative)

HSV is typically implemented using type-safe native structures:

- Dart: Classes with strong typing
- Kotlin: Data classes with null safety
- Swift: Structs with value semantics
- Rust/C++: Structs with explicit lifetimes

These representations are implementation-defined and not standardized.

---

## 8. Relationship to HSI

| Layer | Role |
|---|---|
| HSV | Internal state representation |
| HSI 1.0 | External, canonical interface |

### Key Rule

HSV may be exported to HSI, but HSI never defines HSV internals.

Export is one-way and lossy by design.

---

## 9. Interpretation Boundary

HSV does not:

- Label emotions
- Score focus
- Assess cognition
- Infer intent

Interpretation modules:

- Consume HSV outputs
- Produce semantic annotations
- Are explicitly enabled
- Do not modify base HSV state

This separation is a core invariant.

---

## 10. Privacy and Safety

HSV enforces privacy by construction:

- No raw biosignals
- No identifiers
- No content data
- No audio or text
- No cross-user correlation

All values are:

- Aggregated
- Windowed
- Normalized
- Locally computed

---

## 11. Versioning

HSV versioning follows SDK semantic versioning.

### Breaking changes

- Axis renaming or removal
- Range changes
- Embedding dimensionality changes
- Structural API changes

### Non-breaking changes

- New axes
- Improved fusion logic
- Performance optimizations

---

## 12. Summary

HSV defines what human state is represented, not what it means.

It is:

- Internal
- Time-aware
- Privacy-preserving
- Model-agnostic
- Interface-independent

HSV enables human-state-aware systems without coupling representation, interpretation, or exposure.

---
