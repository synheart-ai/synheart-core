# Capability System

The **Capability System** enforces access control for Synheart Core features based on app identity and authorization level.

---

## Core Principle

> **Capabilities define what apps CAN access.**
>
> **Consent defines what users ALLOW.**
>
> **Data access requires BOTH.**

```
Data Access = Capability (app-level) AND Consent (user-level)
```

---

## Capability Tiers

Synheart Core defines three capability tiers:

### 1. Core (External Apps)

**Who Gets It:**
- Third-party applications
- External developers
- Public SDK users

**What It Provides:**
- Basic HSV axes and indices
- Standard time windows (30s, 5m, 1h, 24h)
- Limited embedding access (normalized only)
- Standard cloud ingestion (HSI 1.0 format)
- Basic interpretation modules (if enabled)

**What It Restricts:**
- No raw biosignals
- No high-frequency data
- No fusion internals
- No research endpoints

---

### 2. Extended (Synheart Apps)

**Who Gets It:**
- Syni Life
- SWIP
- Pulse Focus
- Other Synheart-owned applications

**What It Provides:**
- Full HSV axes and indices
- Full 64D state embeddings
- Higher-frequency updates
- Advanced app context
- Extended behavior metrics
- Extended cloud endpoints (HSI 1.0 format)
- Advanced interpretation modules

**What It Restricts:**
- No raw biosignals (still derived only)
- No internal fusion vectors
- No research-specific endpoints

---

### 3. Research (Internal Research)

**Who Gets It:**
- Synheart Research team
- Authorized research partners
- Internal tooling and analytics

**What It Provides:**
- Full HSV access
- Raw signal streams (with consent)
- Internal fusion vectors
- Event-level behavior data
- Full app context (unhashed)
- Research cloud endpoints (HSI 1.0 format)
- Unrestricted time windows

**What It Restricts:**
- Still requires user consent
- Still respects privacy boundaries

---

## Module-Level Capabilities

Each module has tier-specific access levels:

### Wear Module

| **Tier** | **Signals** | **Frequency** | **Format** |
|----------|-------------|---------------|------------|
| Core | HR, HRV, sleep stages | 1-minute windows | Aggregates only |
| Extended | HR, HRV, sleep, motion | 30-second windows | Full derived signals |
| Research | Full biosignals | Real-time streams | Raw + derived |

**Example:**
```dart
// Core tier
WearData {
  heartRate: 72,  // 1-min average
  hrv: 45,        // 1-min RMSSD
  sleepStage: 'light'
}

// Extended tier
WearData {
  heartRate: 72,              // 30s average
  heartRateVariability: 45,   // 30s RMSSD
  heartRateTimeSeries: [...], // 30s series
  motion: {...}               // Full motion data
}

// Research tier
WearData {
  heartRate: 72,
  rrIntervals: [...],  // Full RR series
  ppgWaveform: [...],  // Raw PPG (if available)
  motion: {...}
}
```

---

### Phone Module

| **Tier** | **Context** | **Granularity** | **Privacy** |
|----------|-------------|-----------------|-------------|
| Core | Screen state, basic motion | Coarse | Hashed app IDs |
| Extended | Screen, motion, app categories | Medium | App categories |
| Research | Full context, app names | Fine | Full context |

**Example:**
```dart
// Core tier
PhoneContext {
  screenActive: true,
  motionState: 'stationary',
  appCategory: 'hash_abc123'  // Hashed
}

// Extended tier
PhoneContext {
  screenActive: true,
  motionState: 'stationary',
  appCategory: 'productivity',  // Category
  notificationCount: 3          // Metadata only
}

// Research tier
PhoneContext {
  screenActive: true,
  motionState: 'stationary',
  appIdentifier: 'com.example.app',  // Full identifier
  notificationMetadata: [...]
}
```

---

### Behavior Module

| **Tier** | **Metrics** | **Resolution** | **Data** |
|----------|-------------|----------------|----------|
| Core | Basic patterns | Aggregates | Counts only |
| Extended | Extended patterns | Windowed | Timing patterns |
| Research | Full event stream | Event-level | All interactions |

**Example:**
```dart
// Core tier
BehaviorMetrics {
  tapCount: 42,
  scrollCount: 15,
  typingCadence: 'medium'
}

// Extended tier
BehaviorMetrics {
  tapCount: 42,
  tapTimings: [...],     // Timing patterns
  scrollVelocity: [...], // Velocity series
  typingRhythm: {...}    // Detailed rhythm
}

// Research tier
BehaviorEvents {
  events: [
    {type: 'tap', timestamp: 1704067200, position: {x: 100, y: 200}},
    {type: 'scroll', timestamp: 1704067201, delta: 50},
    ...
  ]
}
```

---

### HSI Runtime

| **Tier** | **Outputs** | **Embeddings** | **Internals** |
|----------|-------------|----------------|---------------|
| Core | Basic axes | Normalized 64D | No fusion state |
| Extended | Full axes | Full 64D | No fusion state |
| Research | Full axes | Full 64D + fusion | Full fusion vectors |

**Example:**
```dart
// Core tier (internal HSV representation)
HumanStateVector {
  meta: {
    axes: {
      affect: {arousalIndex: 0.72},
      engagement: {engagementStability: 0.68}
    },
    embedding: {
      vector: [...],  // Normalized only
      dimension: 64
    }
  }
}

// Extended tier (internal HSV representation)
HumanStateVector {
  meta: {
    axes: {
      affect: {arousalIndex: 0.72, valenceStability: 0.85},
      engagement: {engagementStability: 0.68, interactionCadence: 0.54},
      activity: {...},
      context: {...}
    },
    embedding: {
      vector: [...],  // Full 64D
      dimension: 64,
      metadata: {...}
    }
  }
}

// Research tier (internal HSV representation)
HumanStateVector {
  meta: {
    axes: {...},
    embedding: {...},
    fusionState: {
      wearVector: [...],
      phoneVector: [...],
      behaviorVector: [...],
      attentionWeights: [...]
    }
  }
}
```

---

### Cloud Connector

| **Tier** | **Endpoints** | **Frequency** | **Batch Size** |
|----------|---------------|---------------|----------------|
| Core | `/v1/ingest/hsi` (HSI 1.0 format) | Standard | 10 snapshots |
| Extended | `/v1/ingest/hsi` (HSI 1.0 format) | Higher | 50 snapshots |
| Research | `/v1/ingest/hsi-research` (HSI 1.0 format) | Unlimited | 200 snapshots |

---

## Capability Enforcement

### 1. Capability Tokens

Apps receive a **capability token** from the Synheart Platform during registration.

**Token Structure (JWT):**
```json
{
  "tenantId": "app_xyz_prod",
  "appIdentifier": "com.example.app",
  "capabilities": {
    "tier": "core",
    "modules": {
      "wear": "core",
      "phone": "core",
      "behavior": "core",
      "hsi": "core",
      "cloud": "core"
    },
    "interpretations": ["focus", "emotion"]
  },
  "issuedAt": 1704067200,
  "expiresAt": 1704153600,
  "signature": "..."
}
```

**Token Verification:**
- SDK validates token signature on initialization
- SDK caches capabilities locally
- SDK checks capabilities before each module operation
- Expired tokens require re-authentication

---

### 2. Runtime Enforcement

Each module checks capabilities before returning data:

```dart
class WearModule {
  Future<WearData> getBiosignals() async {
    final capabilities = await CapabilityManager.getCapabilities();

    switch (capabilities.wear) {
      case 'core':
        return getCoreWearData();  // 1-min aggregates
      case 'extended':
        return getExtendedWearData();  // 30s + motion
      case 'research':
        return getResearchWearData();  // Raw streams
      default:
        throw UnauthorizedError('Invalid wear capability');
    }
  }
}
```

**Enforcement Points:**
- Module initialization
- Data collection
- HSI computation
- Cloud upload
- API responses

---

### 3. Server-Side Validation

Synheart Platform validates capabilities for cloud operations:

```python
def validate_capability(request):
    tenant_id = request.headers.get('X-Synheart-Tenant')
    signature = request.headers.get('X-Synheart-Signature')

    # Validate HMAC signature
    if not verify_hmac(request.body, signature, tenant_id):
        raise AuthenticationError('Invalid signature')

    # Retrieve tenant capabilities
    capabilities = db.get_tenant_capabilities(tenant_id)

    # Check endpoint access
    if request.path == '/v1/ingest/hsi-research':
        if capabilities.tier != 'research':
            raise AuthorizationError('Research tier required')

    return capabilities
```

---

## Capability Upgrades

### Developer Applications

External developers start with **Core** capabilities.

To request **Extended** capabilities:
1. Apply via Synheart Platform console
2. Provide use case justification
3. Undergo security review
4. Sign extended data usage agreement

**Criteria for Extended Access:**
- Established developer account
- Clear product use case
- Privacy & security review
- User benefit justification

**Not Available:**
- Research tier not available to external developers
- Raw biosignals not available outside Synheart

---

### Synheart Internal Apps

Internal apps (Syni Life, SWIP) automatically receive **Extended** capabilities.

**Process:**
1. App registered in Synheart Platform
2. Organization validation
3. Extended capability token issued
4. Regular security audits

---

## Capability-Consent Interaction

Capabilities and consent work together:

### Example Scenarios

**Scenario 1: External App, Full Consent**
- App capability: Core
- User consent: All modules granted
- Result: App gets Core-level HSV (basic axes)

**Scenario 2: Internal App, Full Consent**
- App capability: Extended
- User consent: All modules granted
- Result: App gets Extended-level HSV (full embeddings)

**Scenario 3: External App, Partial Consent**
- App capability: Core
- User consent: Biosignals denied, behavior granted
- Result: App gets Core HSV with `affect` axes = `null`

**Scenario 4: Research App, No Consent**
- App capability: Research
- User consent: All modules denied
- Result: No data (consent required regardless of capability)

---

## Capability Auditing

### SDK Logging

SDK logs capability checks:

```dart
logger.info('Capability check', {
  'module': 'wear',
  'requested': 'extended',
  'granted': 'core',
  'result': 'downgraded'
});
```

### Platform Analytics

Synheart Platform tracks capability usage:
- Endpoint access patterns
- Capability tier distribution
- Upgrade requests
- Violations and errors

---

## Security Considerations

### Tampering Prevention

**SDK Protection:**
- Capability tokens signed by Synheart Platform
- Code obfuscation (release builds)
- Runtime integrity checks
- Certificate pinning

**Attack Mitigation:**
- Token replay: Prevented by expiry
- Token forgery: Prevented by signature verification
- Module bypass: Prevented by enforcement at multiple layers
- Man-in-the-middle: Prevented by TLS + cert pinning

### Violation Handling

If capability violation detected:
1. SDK logs error
2. Operation denied
3. Error reported to Synheart Platform
4. Repeated violations → token revoked

---

## API Reference

### Check Capabilities

```dart
// Get current capability tier
CapabilityTier tier = await Synheart.getCapabilityTier();
// Returns: CapabilityTier.core | .extended | .research

// Check module capability
String wearCap = await Synheart.getModuleCapability('wear');
// Returns: 'core' | 'extended' | 'research'

// Check if capability is granted
bool hasExtended = await Synheart.hasCapability('extended');
```

### Capability Events

```dart
// Listen for capability changes
Synheart.onCapabilityChanged.listen((event) {
  print('Capability updated: ${event.tier}');
});
```

---

## Testing

### Mock Capabilities

For testing, override capabilities:

```dart
// Set mock capability for testing
Synheart.setMockCapability(CapabilityTier.extended);

// Test extended features
final data = await Synheart.getWearData();
expect(data.heartRateTimeSeries, isNotNull);  // Only in Extended
```

### Capability Test Matrix

| **Test Case** | **Capability** | **Consent** | **Expected Result** |
|---------------|----------------|-------------|---------------------|
| Basic access | Core | Granted | Basic HSV axes |
| No consent | Core | Denied | All axes = null |
| Extended access | Extended | Granted | Full HSV + embeddings |
| Downgrade | Core | Granted | Core HSV (not Extended) |
| Research access | Research | Granted | Full HSV + fusion |

---

## Migration Guide

### From Core to Extended

When upgrading from Core to Extended:

1. **Request upgrade** via Synheart Platform console
2. **Update SDK** to handle Extended data:
   ```dart
   // Before (Core)
   print(hsv.meta.axes.affect.arousalIndex);

   // After (Extended)
   print(hsv.meta.axes.affect.arousalIndex);
   print(hsv.meta.axes.affect.valenceStability);  // Now available
   print(hsv.meta.embedding.vector);              // Full 64D now available
   ```
3. **Test thoroughly** with Extended data
4. **Update privacy policy** to reflect Extended data access
5. **Deploy with new capability token**

---

## Related Documentation

- [PRD](PRD.md) - Product requirements
- [HSV + HSI Specification](HSV_AND_HSI_SPECIFICATION.md) - State representation
- [Consent System](CONSENT_SYSTEM.md) - User permission model
- [Cloud Protocol](CLOUD_PROTOCOL.md) - Cloud upload specification

---

**Last Updated:** 2025-12-25
**Version:** 1.0.0
**Maintained by:** Synheart AI
