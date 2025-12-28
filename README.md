# Synheart Core SDK

**Unified SDK for all Synheart features — single integration point for human-state intelligence**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Platform Support](https://img.shields.io/badge/platforms-Dart%20%7C%20Kotlin%20%7C%20Swift-blue.svg)](#-sdks)

The Synheart Core SDK is the single, unified integration point for developers who want to collect HSI-compatible data, process human state on-device, generate focus/emotion signals, integrate with Syni, upload derived HSI snapshots to the cloud (with user consent), and visualize state dashboards.

## 🚀 Features

- **🔗 Unified API**: Single SDK for all Synheart features
- **🧠 HSI Runtime**: On-device human state fusion and inference
- **📱 Multi-Module**: Wear, Phone, Behavior, HSI, Consent, Cloud, Syni
- **⚡ On-Device Processing**: All inference happens locally
- **🔒 Privacy-First**: Zero raw data, consent-gated, capability-based
- **🌐 Multi-Platform**: Flutter/Dart, Android/Kotlin, iOS/Swift
- **🎯 Capability System**: Core/Extended/Research modes

## 📦 SDKs

The Core SDK is available for mobile platforms:

### Flutter/Dart SDK
```yaml
dependencies:
  synheart_core: ^1.0.0
```
📖 **Repository**: [synheart-core-sdk-dart](https://github.com/synheart-ai/synheart-core-sdk-dart)

### Android SDK (Kotlin)
```kotlin
dependencies {
    implementation("ai.synheart:core-sdk:1.0.0")
}
```
📖 **Repository**: [synheart-core-sdk-kotlin](https://github.com/synheart-ai/synheart-core-sdk-kotlin)

### iOS SDK (Swift)
**Swift Package Manager:**
```swift
dependencies: [
    .package(url: "https://github.com/synheart-ai/synheart-core-sdk-swift.git", from: "1.0.0")
]
```
📖 **Repository**: [synheart-core-sdk-swift](https://github.com/synheart-ai/synheart-core-sdk-swift)

## 📂 Repository Structure

This repository serves as the **source of truth** for shared resources across all SDK implementations:

```
synheart-core/
├── docs/                          # Technical documentation (specs)
│   ├── HSI_SPECIFICATION.md
│   ├── CONSENT_SYSTEM.md
│   └── CLOUD_PROTOCOL.md
├── examples/                      # Example apps 
├── scripts/                       # Build and deployment scripts
└── CONTRIBUTING.md                # Contribution guidelines for all SDKs
```

**Platform-specific SDK repositories** (maintained separately):
- [synheart-core-sdk-dart](https://github.com/synheart-ai/synheart-core-sdk-dart) - Flutter/Dart SDK
- [synheart-core-sdk-kotlin](https://github.com/synheart-ai/synheart-core-sdk-kotlin) - Android/Kotlin SDK
- [synheart-core-sdk-swift](https://github.com/synheart-ai/synheart-core-sdk-swift) - iOS/Swift SDK

## 🎯 Quick Start

### Flutter/Dart

```dart
import 'package:synheart_core/synheart_core.dart';

// Initialize Core SDK
await Synheart.initialize(
  userId: 'anon_user_123',
  config: SynheartConfig(
    enableWear: true,
    enablePhone: true,
    enableBehavior: true,
  ),
);

// Subscribe to HSI updates (core state representation)
Synheart.onHSIUpdate.listen((hsi) {
  print('Arousal Index: ${hsi.affect.arousalIndex}');
  print('Engagement Stability: ${hsi.engagement.engagementStability}');
});

// Optional: Enable interpretation modules
await Synheart.enableFocus();
Synheart.onFocusUpdate.listen((focus) {
  print('Focus Score: ${focus.estimate.score}');
});

await Synheart.enableEmotion();
Synheart.onEmotionUpdate.listen((emotion) {
  print('Stress Index: ${emotion.stressIndex}');
});

// Enable cloud upload (with consent)
await Synheart.enableCloud();
```

### Kotlin/Android

```kotlin
import ai.synheart.core.Synheart
import ai.synheart.core.SynheartConfig

// Initialize
Synheart.initialize(
    userId = "anon_user_123",
    config = SynheartConfig(
        enableWear = true,
        enablePhone = true,
        enableBehavior = true
    )
)

// Subscribe to HSI updates (core state representation)
Synheart.onHSIUpdate.collect { hsi ->
    println("Arousal Index: ${hsi.affect.arousalIndex}")
    println("Engagement Stability: ${hsi.engagement.engagementStability}")
}

// Optional: Enable interpretation modules
Synheart.enableFocus()
Synheart.onFocusUpdate.collect { focus ->
    println("Focus Score: ${focus.estimate.score}")
}

// Enable cloud
Synheart.enableCloud()
```

### Swift/iOS

```swift
import SynheartCore

// Initialize
Synheart.initialize(
    userId: "anon_user_123",
    config: SynheartConfig(
        enableWear: true,
        enablePhone: true,
        enableBehavior: true
    )
)

// Subscribe to HSI updates (core state representation)
Synheart.onHSIUpdate.sink { hsi in
    print("Arousal Index: \(hsi.affect.arousalIndex)")
    print("Engagement Stability: \(hsi.engagement.engagementStability)")
}

// Optional: Enable interpretation modules
Synheart.enableFocus()
Synheart.onFocusUpdate.sink { focus in
    print("Focus Score: \(focus.estimate.score)")
}

// Enable cloud
Synheart.enableCloud()
```

## 🏗️ Architecture

### Module System

The Core SDK consolidates all Synheart signal channels:

```
Synheart Core SDK
│
├── Wear Module
│      (HR, HRV, sleep, motion — derived signals only)
│
├── Phone Module
│      (motion, screen state, coarse app context)
│
├── Synheart Behavior (Module)
│      (interaction patterns: taps, scrolls, typing cadence)
│
├── HSI Runtime (On-device)
│      - multimodal fusion
│      - state axes & indices
│      - time windows (30s, 5m, 1h, 24h)
│      - 64D state embedding
│
├── Interpretation Modules (Optional)
│      ├── EmotionHead (uses synheart-emotion package)
│      │     (affect modeling - optional, explicit enable)
│      │     Powered by: synheart-emotion SDK
│      └── FocusHead (uses synheart-focus package)
│            (engagement/focus estimation - optional, explicit enable)
│            Powered by: synheart-focus SDK
│
├── Consent Module
│      (permissions, masking, enforcement)
│
├── Cloud Connector
│      (secure, consent-gated uploads)
│
└── Syni Hooks
       (HSI context + optional interpretations)
```

### Capability System

Each module reads capability flags from Auth:

| Module | Core | Extended | Research |
|--------|------|----------|----------|
| Wear | derived biosignals | higher freq | raw streams |
| Phone | motion, screen | advanced app context | full context |
| Behavior | basic metrics | extended metrics | event-level streams |
| HSI | basic state | full embedding | full fusion vectors |
| Connector | ingest | extended endpoints | research endpoints |

Only Synheart apps (Syni Life, SWIP, Platform) get extended/research capabilities. External apps get core only.

## 📚 Documentation

- [Product Requirements](docs/PRD.md) - Product specification and goals
- [HSI Specification](docs/HSI_SPECIFICATION.md) - State axes, indices, and embeddings
- [Capability System](docs/CAPABILITY_SYSTEM.md) - Access level enforcement
- [Consent System](docs/CONSENT_SYSTEM.md) - Permission model and enforcement
- [Cloud Protocol](docs/CLOUD_PROTOCOL.md) - Secure ingestion protocol

## 🔒 Privacy & Security

- **Zero Raw Content**: No text, mic, URLs, messages
- **On-Device Processing**: All inference happens locally
- **No Raw Biosignals**: Only derived signals externally
- **Consent-Gated**: All cloud uploads require explicit consent
- **Capability-Enforced**: Feature access tied to app signature and tenant ID

## ⚡ Performance

- **CPU**: < 2%
- **Memory**: < 15MB
- **Battery**: < 0.5%/hr
- **HSI Updates**: ≤ 100ms latency
- **Cloud Upload**: ≤ 80ms request time

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

Apache 2.0 License - see [LICENSE](LICENSE) for details.

## 🔗 Related Projects & Dependencies

### Core Dependencies

Synheart Core depends on the following SDKs as implementation layers:

- **[Synheart Emotion](https://github.com/synheart-ai/synheart-emotion)** - Powers EmotionHead module
  - Provides emotion inference from biosignals (HR/RR)
  - Used by: EmotionHead for affect modeling
  - Detects: Amused, Calm, Stressed states
  - Schema validated against HSI specification

- **[Synheart Focus](https://github.com/synheart-ai/synheart-focus)** - Powers FocusHead module
  - Provides cognitive concentration inference
  - Used by: FocusHead for engagement/focus estimation
  - Outputs: Focus score, cognitive load, clarity
  - Schema validated against HSI specification

### Supporting Libraries

- **[Synheart Wear](https://github.com/synheart-ai/synheart-wear)** - Wearable device integration
  - Used by: Wear Module for biosignal collection
  - Supports: Apple Watch, Garmin, WHOOP, etc.

- **[Synheart Behavior](https://github.com/synheart-ai/synheart-behavior)** - Digital behavioral signal capture
  - Used by: Behavior Module for interaction patterns
  - Tracks: Taps, scrolls, typing cadence, idle patterns

### Dependency Architecture

```
Runtime Dependencies (package):
  synheart-core → synheart-emotion (EmotionHead implementation)
  synheart-core → synheart-focus (FocusHead implementation)
  synheart-core → synheart-wear (Wear Module)
  synheart-core → synheart-behavior (Behavior Module)

Schema Validation (no code dependency):
  synheart-emotion ← validates against HSI_SPECIFICATION.md
  synheart-focus ← validates against HSI_SPECIFICATION.md
```

**Key Principle:**
- synheart-emotion and synheart-focus remain **standalone SDKs**
- They can be used independently without synheart-core
- synheart-core uses them as implementation layers for EmotionHead and FocusHead
- Their output schemas are validated against HSI specification for compatibility

---

**Author**: Israel Goytom  
**Organization**: Synheart AI <3 

