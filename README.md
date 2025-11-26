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
  synheart_core: ^0.1.0
```
📖 **Repository**: [synheart-core-sdk-dart](https://github.com/synheart-ai/synheart-core-sdk-dart)

### Android SDK (Kotlin)
```kotlin
dependencies {
    implementation("ai.synheart:core-sdk:0.1.0")
}
```
📖 **Repository**: [synheart-core-sdk-kotlin](https://github.com/synheart-ai/synheart-core-sdk-kotlin)

### iOS SDK (Swift)
**Swift Package Manager:**
```swift
dependencies: [
    .package(url: "https://github.com/synheart-ai/synheart-core-sdk-swift.git", from: "0.1.0")
]
```
📖 **Repository**: [synheart-core-sdk-swift](https://github.com/synheart-ai/synheart-core-sdk-swift)

## 📂 Repository Structure

This repository serves as the **source of truth** for shared resources across all SDK implementations:

```
synheart-core-sdk/
├── docs/                          # Technical documentation
│   ├── ARCHITECTURE.md            # System architecture
│   ├── API_REFERENCE.md           # API documentation
│   └── MODULES.md                 # Module documentation
│
├── examples/                      # Cross-platform example applications
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
    // Module configuration
    enableWear: true,
    enablePhone: true,
    enableBehavior: true,
  ),
);

// Subscribe to HSI state updates
Synheart.onStateUpdate.listen((state) {
  print('Focus: ${state.focus.focusScore}');
  print('Emotion: ${state.emotion.stressIndex}');
  print('Behavior: ${state.behavior.distractionScore}');
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

// Subscribe to updates
Synheart.onStateUpdate.collect { state ->
    println("Focus: ${state.focus.focusScore}")
    println("Emotion: ${state.emotion.stressIndex}")
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

// Subscribe to updates
Synheart.onStateUpdate.sink { state in
    print("Focus: \(state.focus.focusScore)")
    print("Emotion: \(state.emotion.stressIndex)")
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
│      (HR, HRV, sleep, motion from wearables or cloud sync)
│
├── Phone Module
│      (motion, screen, app context)
│
├── Behavior Module
│      (interaction patterns: taps, scrolls, typing cadence)
│
├── HSI Runtime (On-device)
│      - fusion engine
│      - state windows (30s, 5m, 1h)
│      - embedding model (64D)
│
├── Focus Engine
│      (focus_score, flow_likelihood)
│
├── Emotion Engine
│      (stress, calm, valence, arousal)
│
├── Consent Module
│      (captures user permissions, enforces masking)
│
├── Cloud Connector
│      (secure ingestion)
│
└── Syni Hooks
       (HSI → persona conditioning)
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

- [Architecture Guide](docs/ARCHITECTURE.md) - Detailed system architecture
- [API Reference](docs/API_REFERENCE.md) - Complete API documentation
- [Modules Guide](docs/MODULES.md) - Module-specific documentation

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

## 🔗 Related Projects

- [Synheart Focus](https://github.com/synheart-ai/synheart-focus) - Cognitive concentration inference
- [Synheart Emotion](https://github.com/synheart-ai/synheart-emotion) - Physiological emotion inference
- [Synheart Behavior](https://github.com/synheart-ai/synheart-behavior) - Digital behavioral signal capture
- [Synheart Wear](https://github.com/synheart-ai/synheart-wear) - Wearable device integration

---

**Author**: Israel Goytom  
**Organization**: Synheart Research & Engineering

