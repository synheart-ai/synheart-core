# Consent System

The **Consent Module** manages user permissions and enforces privacy boundaries across all Synheart Core modules.

---

## Core Principles

1. **Explicit Consent**: All data collection requires explicit user consent
2. **Granular Control**: Users control each module independently
3. **Revocable**: Consent can be withdrawn at any time
4. **Enforced**: Missing consent = no data collection, not synthetic data
5. **Transparent**: Users see exactly what each module collects

---

## Consent Types

Synheart Core defines the following consent types:

### Module-Level Consents

| **Consent Type** | **Module** | **What It Enables** | **Default** |
|------------------|------------|---------------------|-------------|
| `biosignals` | Wear Module | HR, HRV, sleep, motion from wearables | `false` |
| `phoneContext` | Phone Module | Device motion, screen state, app context | `false` |
| `behavior` | Behavior Module | Taps, scrolls, typing cadence | `false` |
| `cloudUpload` | Cloud Connector | Upload derived HSI to cloud | `false` |

### Interpretation Module Consents

| **Consent Type** | **Module** | **What It Enables** | **Default** |
|------------------|------------|---------------------|-------------|
| `focusEstimation` | Synheart Focus | Focus score computation | `false` |
| `emotionEstimation` | Synheart Emotion | Emotion/stress inference | `false` |

---

## Consent Flow

### 1. Initial Consent Request

When the SDK initializes, it checks for existing consent. If no consent exists, the SDK:

1. Returns empty/null state until consent is granted
2. Provides consent request callbacks to the app
3. Does NOT collect any data

**Example:**
```dart
await Synheart.initialize(
  userId: 'anon_user_123',
  config: SynheartConfig(
    enableWear: true,  // Requests consent, does not auto-enable
    enablePhone: true,
    enableBehavior: true,
  ),
);

// SDK checks consent status
bool hasConsent = await Synheart.hasConsent('biosignals');

if (!hasConsent) {
  // App shows consent UI
  bool granted = await showConsentDialog('biosignals');

  if (granted) {
    await Synheart.grantConsent('biosignals');
  }
}
```

### 2. Consent Storage

Consent preferences are stored:
- **Locally**: On-device encrypted storage
- **Not synced**: Consent is device-specific
- **Versioned**: Consent tied to SDK version

**Storage Format:**
```json
{
  "userId": "anon_user_123",
  "consents": {
    "biosignals": {
      "granted": true,
      "timestamp": 1704067200,
      "sdkVersion": "1.0.0"
    },
    "phoneContext": {
      "granted": true,
      "timestamp": 1704067200,
      "sdkVersion": "1.0.0"
    },
    "behavior": {
      "granted": false,
      "timestamp": null,
      "sdkVersion": null
    },
    "cloudUpload": {
      "granted": false,
      "timestamp": null,
      "sdkVersion": null
    }
  }
}
```

### 3. Consent Revocation

Users can revoke consent at any time:

```dart
await Synheart.revokeConsent('biosignals');
```

**Effects:**
- Module immediately stops collecting data
- Existing local data is NOT deleted (user controls via separate data deletion API)
- HSI axes dependent on revoked module return `null`
- Cloud uploads stop immediately

---

## Enforcement Mechanisms

### Module-Level Enforcement

Each module checks consent before processing:

```dart
class WearModule {
  Future<void> collectBiosignals() async {
    // Check consent before collection
    if (!await ConsentModule.hasConsent('biosignals')) {
      return;  // No collection
    }

    // Proceed with collection
    final hrData = await fetchHeartRate();
    processHRData(hrData);
  }
}
```

### HSI Runtime Enforcement

HSI Runtime respects consent when computing state:

```dart
HSI computeHSI(SignalBundle signals) {
  final hsi = HSI();

  // Only compute axes if consent granted
  if (ConsentModule.hasConsent('biosignals')) {
    hsi.affect.arousalIndex = computeArousal(signals.wear);
  } else {
    hsi.affect.arousalIndex = null;  // Not 0.0, but null
  }

  if (ConsentModule.hasConsent('behavior')) {
    hsi.engagement.engagementStability = computeEngagement(signals.behavior);
  } else {
    hsi.engagement.engagementStability = null;
  }

  return hsi;
}
```

### Cloud Upload Enforcement

Cloud Connector requires explicit `cloudUpload` consent:

```dart
await Synheart.enableCloud();  // Requests cloudUpload consent

// Before each upload
if (await ConsentModule.hasConsent('cloudUpload')) {
  await uploadHSISnapshot(hsi);
} else {
  // No upload
}
```

---

## Consent UI Guidelines

### Recommended Consent Flow

1. **Contextual Requests**: Ask for consent when the feature is first needed
2. **Clear Explanations**: Explain what data is collected and why
3. **Granular Options**: Allow users to grant/deny each module independently
4. **Easy Revocation**: Provide settings UI to revoke consent

### Example Consent Dialog

```
┌─────────────────────────────────────┐
│  Enable Biosignal Monitoring?       │
├─────────────────────────────────────┤
│                                     │
│  This allows Synheart to:           │
│  • Read heart rate from your watch  │
│  • Analyze sleep patterns           │
│  • Detect physical activity         │
│                                     │
│  Synheart does NOT access:          │
│  • Raw ECG or PPG data              │
│  • Location or GPS                  │
│  • Direct identifiers (name, email, phone) │
│                                     │
│  [Learn More]  [Deny]  [Allow]      │
└─────────────────────────────────────┘
```

### Settings UI

Apps should provide a settings screen where users can:
- View current consent status
- Revoke any consent
- Delete local data
- Export or delete cloud data

---

## Consent & Capability Interaction

Consent and Capabilities are separate but related:

| **Concept** | **Enforces** | **Scope** |
|-------------|--------------|-----------|
| **Consent** | User permission to collect data | Per-user, per-device |
| **Capability** | App permission to access features | Per-app, server-issued |

**Example:**
- External app has `Core` capability → can access basic HSI
- External app has user consent for `biosignals` → can compute `arousalIndex`
- External app does NOT have user consent for `behavior` → `engagementStability` is `null`
- Internal app has `Extended` capability + user consent → can access full 64D embeddings

**Enforcement:**
```
Data Collection = Consent (user) AND Capability (app)
```

---

## Data Retention & Deletion

### Local Data

Synheart Core stores minimal local data:
- HSI snapshots (last 24 hours, rolling window)
- Consent preferences
- Module state caches

**User Controls:**
```dart
// Delete all local data
await Synheart.deleteLocalData();

// Delete data for specific module
await Synheart.deleteModuleData('biosignals');
```

### Cloud Data

If `cloudUpload` consent is granted, HSI snapshots are uploaded.

**User Controls:**
```dart
// Stop cloud uploads
await Synheart.revokeConsent('cloudUpload');

// Delete cloud data (requires API call to Synheart Platform)
await Synheart.deleteCloudData();
```

---

## Privacy Guarantees

### What Consent DOES NOT Allow

Even with full consent, Synheart Core NEVER collects:

- ❌ Raw biosignals (ECG, PPG waveforms)
- ❌ Message content or URLs
- ❌ Keyboard input (content)
- ❌ Specific app names (only hashed context)
- ❌ Audio or microphone data
- ❌ Location or GPS
- ❌ Photos or media files
- ❌ Contact lists or identifiers

### What Consent DOES Allow

With appropriate consent, Synheart Core collects:

- ✅ Derived biosignals (HR, HRV, sleep stages)
- ✅ Device motion patterns
- ✅ Screen on/off state
- ✅ Interaction timing (taps, scrolls)
- ✅ Hashed app context (non-reversible)
- ✅ HSI state representations

---

## Consent Versioning

When the SDK is updated, consent may need to be re-requested if:

1. New data types are collected
2. Privacy policy changes
3. Module behavior changes

**Version Check:**
```dart
// Check if consent is valid for current SDK version
bool isValid = await ConsentModule.isConsentValid('biosignals');

if (!isValid) {
  // Re-request consent
  await showConsentDialog('biosignals');
}
```

---

## API Reference

### Consent Management

```dart
// Check consent status
bool hasConsent = await Synheart.hasConsent('biosignals');

// Grant consent
await Synheart.grantConsent('biosignals');

// Revoke consent
await Synheart.revokeConsent('biosignals');

// Get all consent statuses
Map<String, bool> consents = await Synheart.getConsentStatus();

// Delete local data
await Synheart.deleteLocalData();

// Delete cloud data
await Synheart.deleteCloudData();
```

### Consent Callbacks

```dart
// Listen for consent changes
Synheart.onConsentChanged.listen((event) {
  print('Consent changed: ${event.consentType} = ${event.granted}');
});
```

---

## Testing Consent Enforcement

### Unit Tests

```dart
test('HSI returns null when consent revoked', () async {
  // Grant consent
  await Synheart.grantConsent('biosignals');

  // Get HSI
  final hsi1 = await Synheart.getLatestHSI();
  expect(hsi1.affect.arousalIndex, isNotNull);

  // Revoke consent
  await Synheart.revokeConsent('biosignals');

  // Get HSI again
  final hsi2 = await Synheart.getLatestHSI();
  expect(hsi2.affect.arousalIndex, isNull);  // Must be null
});
```

---

## Related Documentation

- [PRD](PRD.md) - Product requirements
- [HSI Specification](HSI_SPECIFICATION.md) - State representation
- [Capability System](CAPABILITY_SYSTEM.md) - App-level permissions
- [Cloud Protocol](CLOUD_PROTOCOL.md) - Cloud upload specification

---

**Last Updated:** 2025-12-25
**Version:** 1.0.0
**Maintained by:** Synheart AI
