# Cloud Protocol

The **Cloud Connector** enables secure, consent-gated upload of derived HSI snapshots to the Synheart Platform.

---

## Core Principles

1. **Consent-Gated**: Uploads only occur with explicit user consent
2. **Derived Data Only**: Only HSI representations, never raw signals
3. **Secure Transport**: HTTPS + HMAC authentication
4. **Privacy-Preserving**: No raw content, direct identifiers, or raw biosignals (only pseudonymous IDs for routing)
5. **Tenant-Isolated**: Data routed to correct tenant namespace

---

## Architecture

```
SDK (On-Device)                    Synheart Platform (Cloud)
│                                   │
├─ HSI Runtime                      ├─ Ingestion API
│   ↓                               │   ↓
├─ Cloud Connector                  ├─ Tenant Router
│   ↓                               │   ↓
│  [HMAC Sign]                      ├─ HMAC Validator
│   ↓                               │   ↓
│  HTTPS POST ─────────────────────→├─ HSI Storage
│   ↓                               │   ↓
│  [Response]←──────────────────────├─ Analytics Pipeline
```

---

## Upload Flow

### 1. Prerequisites

Before any upload:
1. User grants `cloudUpload` consent
2. SDK obtains tenant credentials from Synheart Platform
3. SDK generates or retrieves HMAC secret key

### 2. HSI Snapshot Preparation

The SDK prepares an HSI snapshot for upload:

```json
{
  "userId": "anon_user_123",
  "tenantId": "app_xyz_prod",
  "timestamp": 1704067200,
  "windowType": "short",
  "duration": 300,
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
  },
  "embedding": {
    "vector": [0.12, -0.34, 0.56, ..., 0.78],
    "dimension": 64,
    "model": "hsi-fusion-v1"
  },
  "metadata": {
    "sdkVersion": "1.0.0",
    "platform": "ios",
    "deviceType": "iphone"
  }
}
```

### 3. HMAC Signing

The SDK signs the request with HMAC-SHA256:

```dart
String computeHMAC(String payload, String secret, String nonce) {
  final message = '$payload:$nonce';
  final hmac = Hmac(sha256, utf8.encode(secret));
  final digest = hmac.convert(utf8.encode(message));
  return digest.toString();
}
```

**Signature Components:**
- Payload: JSON-encoded HSI snapshot
- Secret: Tenant-specific HMAC key (obtained during SDK initialization)
- Nonce: Time-windowed random nonce (prevents replay attacks)

### 4. HTTP Request

**Endpoint:**
```
POST https://api.synheart.ai/v1/ingest/hsi
```

**Headers:**
```
Content-Type: application/json
X-Synheart-Tenant: app_xyz_prod
X-Synheart-Signature: <HMAC signature>
X-Synheart-Nonce: <time-windowed nonce>
X-Synheart-Timestamp: <Unix timestamp>
X-Synheart-SDK-Version: 1.0.0
```

**Body:**
```json
{
  "userId": "anon_user_123",
  "snapshot": { ... }
}
```

### 5. Server Validation

The Synheart Platform validates:
1. **HMAC signature**: Recomputes HMAC and compares
2. **Nonce freshness**: Checks nonce is within 5-minute window
3. **Tenant authorization**: Verifies tenant has ingestion capability
4. **Schema validation**: Ensures HSI snapshot matches expected format

### 6. Response

**Success (200 OK):**
```json
{
  "status": "accepted",
  "snapshotId": "hsi_snapshot_abc123",
  "timestamp": 1704067205
}
```

**Error (400/401/403):**
```json
{
  "status": "error",
  "code": "invalid_signature",
  "message": "HMAC signature validation failed"
}
```

---

## Security

### HMAC Authentication

**Why HMAC?**
- Prevents tampering with requests
- Prevents replay attacks (via nonce)
- Lightweight (no public key infrastructure)
- Tenant-scoped (each tenant has unique secret)

**HMAC Secret Management:**
- Secrets issued by Synheart Platform during app registration
- Secrets stored securely on-device (Keychain on iOS, KeyStore on Android)
- Secrets rotated periodically (90-day cycle)

### Nonce System

**Nonce Requirements:**
- Time-windowed: valid for 5 minutes
- Unique: generated via secure random
- Checked server-side: prevents replay attacks

**Nonce Format:**
```
<timestamp>_<random_hex>
Example: 1704067200_a3f8c9d2e1b4
```

**Server-Side Validation:**
```python
def validate_nonce(nonce, signature_timestamp):
    parts = nonce.split('_')
    nonce_timestamp = int(parts[0])
    current_time = time.time()

    # Check freshness (within 5 minutes)
    if abs(current_time - nonce_timestamp) > 300:
        raise InvalidNonceError("Nonce expired")

    # Check nonce hasn't been used before
    if redis.exists(f"nonce:{nonce}"):
        raise InvalidNonceError("Nonce already used")

    # Mark nonce as used (with 10-minute TTL)
    redis.setex(f"nonce:{nonce}", 600, "1")
```

### Transport Security

- **TLS 1.3**: All requests use HTTPS
- **Certificate Pinning**: SDK pins Synheart Platform certificates
- **No Sensitive Data**: HSI snapshots contain no PII or raw signals

---

## Rate Limiting

### Client-Side Rate Limiting

SDK implements local rate limiting:

| **Window Type** | **Max Upload Frequency** |
|-----------------|--------------------------|
| Micro (30s) | Every 30 seconds |
| Short (5m) | Every 2 minutes |
| Medium (1h) | Every 10 minutes |
| Long (24h) | Every 1 hour |

**Batching:**
- SDK may batch multiple snapshots into a single request
- Max batch size (by capability tier): 10 (Core), 50 (Extended), 200 (Research)
- Max request size: 1 MB

### Server-Side Rate Limiting

Synheart Platform enforces rate limits per tenant:

| **Tier** | **Requests/Minute** | **Requests/Hour** |
|----------|---------------------|-------------------|
| Free | 10 | 200 |
| Developer | 60 | 2,000 |
| Production | 600 | 20,000 |
| Enterprise | Custom | Custom |

**Rate Limit Response (429):**
```json
{
  "status": "error",
  "code": "rate_limit_exceeded",
  "message": "Too many requests",
  "retryAfter": 60
}
```

---

## Data Retention

### Cloud Storage

HSI snapshots uploaded to the cloud are retained:

- **Default**: 90 days
- **Extended**: 1 year (with user consent)
- **Research**: 5 years (research-tier apps only)

### User Deletion

Users can delete cloud data:

```dart
// Request deletion of all cloud data
await Synheart.deleteCloudData();
```

**Deletion Process:**
1. SDK sends deletion request to Synheart Platform
2. Platform marks data for deletion
3. Data purged within 7 days
4. Confirmation sent to user

---

## Tenant Routing

### Tenant Identification

Each app is assigned a unique `tenantId`:

```
Format: <app_identifier>_<environment>
Examples:
  - syni_life_prod
  - external_app_xyz_dev
  - research_lab_staging
```

### Routing Logic

Synheart Platform routes HSI snapshots to tenant-specific storage:

```
api.synheart.ai/v1/ingest/hsi
    ↓
[Validate HMAC & Nonce]
    ↓
[Extract tenantId from header]
    ↓
[Route to tenant namespace]
    ↓
s3://synheart-hsi-prod/<tenantId>/<userId>/<timestamp>.json
```

**Tenant Isolation:**
- Each tenant has isolated storage namespace
- Cross-tenant access is forbidden
- Data queries are tenant-scoped

---

## Capability-Based Upload

Upload endpoints respect capability levels:

| **Capability** | **Endpoint** | **Access** |
|----------------|--------------|------------|
| Core | `/v1/ingest/hsi` | Basic HSI axes |
| Extended | `/v1/ingest/hsi` | Full 64D embeddings |
| Research | `/v1/ingest/hsi-research` | Full fusion vectors |

**Extended Payloads (Extended/Research):**

```json
{
  "userId": "anon_user_123",
  "snapshot": {
    "axes": { ... },
    "embedding": {
      "vector": [...],  // Full 64D (Extended)
      "fusionVectors": {...}  // Internal fusion state (Research only)
    }
  }
}
```

---

## Error Handling

### Client-Side Retry Logic

SDK implements exponential backoff for failed uploads:

```dart
Future<void> uploadWithRetry(HSISnapshot snapshot) async {
  int attempts = 0;
  int maxAttempts = 3;
  int baseDelay = 1000;  // 1 second

  while (attempts < maxAttempts) {
    try {
      await _upload(snapshot);
      return;  // Success
    } catch (e) {
      attempts++;
      if (attempts >= maxAttempts) {
        // Store locally for later retry
        await _storeForRetry(snapshot);
        throw e;
      }

      // Exponential backoff: 1s, 2s, 4s
      int delay = baseDelay * pow(2, attempts - 1);
      await Future.delayed(Duration(milliseconds: delay));
    }
  }
}
```

### Error Codes

| **Code** | **Description** | **Client Action** |
|----------|-----------------|-------------------|
| `invalid_signature` | HMAC validation failed | Regenerate signature |
| `invalid_nonce` | Nonce expired or reused | Generate new nonce |
| `rate_limit_exceeded` | Too many requests | Backoff and retry |
| `invalid_tenant` | Tenant not found | Check tenant credentials |
| `schema_validation_failed` | Invalid HSI format | Check SDK version |

---

## Offline Support

### Local Queueing

When network is unavailable, SDK queues HSI snapshots locally:

```dart
class UploadQueue {
  List<HSISnapshot> _queue = [];

  Future<void> enqueue(HSISnapshot snapshot) async {
    _queue.add(snapshot);
    await _persistQueue();

    // Limit queue size (max 100 snapshots)
    if (_queue.length > 100) {
      _queue.removeAt(0);  // FIFO
    }
  }

  Future<void> flush() async {
    while (_queue.isNotEmpty && _isOnline()) {
      final snapshot = _queue.first;
      try {
        await _upload(snapshot);
        _queue.removeAt(0);
        await _persistQueue();
      } catch (e) {
        break;  // Stop on failure
      }
    }
  }
}
```

### Network State Monitoring

SDK monitors network state and automatically flushes queue when online:

```dart
NetworkConnectivity.onConnectivityChanged.listen((status) {
  if (status == ConnectivityStatus.connected) {
    _uploadQueue.flush();
  }
});
```

---

## Performance Targets

| **Metric** | **Target** | **Measurement** |
|------------|------------|-----------------|
| Upload latency | ≤ 80ms | 95th percentile |
| Request size | < 10 KB | Per snapshot |
| Batch size | ≤ 1 MB | Per request |
| Queue size | ≤ 100 snapshots | Local storage |
| Retry attempts | ≤ 3 | Per snapshot |

---

## API Reference

### Enable Cloud Upload

```dart
// Request cloudUpload consent and enable uploads
await Synheart.enableCloud();

// Check upload status
bool isEnabled = await Synheart.isCloudEnabled();

// Disable cloud uploads
await Synheart.disableCloud();
```

### Manual Upload

```dart
// Force upload of current HSI snapshot
await Synheart.uploadNow();

// Flush upload queue
await Synheart.flushUploadQueue();
```

### Upload Callbacks

```dart
// Listen for upload events
Synheart.onUploadSuccess.listen((event) {
  print('Uploaded snapshot: ${event.snapshotId}');
});

Synheart.onUploadError.listen((error) {
  print('Upload failed: ${error.message}');
});
```

---

## Testing

### Mock Server

For testing, use the Synheart Platform sandbox:

```
Endpoint: https://sandbox.synheart.ai/v1/ingest/hsi
Tenant: test_tenant_sandbox
Secret: test_hmac_secret_xyz
```

### Unit Tests

```dart
test('HMAC signature is valid', () {
  final payload = '{"userId":"test","snapshot":{}}';
  final secret = 'test_secret';
  final nonce = '1704067200_abc123';

  final signature = computeHMAC(payload, secret, nonce);
  expect(signature, isNotEmpty);
});

test('Upload fails without consent', () async {
  await Synheart.revokeConsent('cloudUpload');

  expect(
    () => Synheart.uploadNow(),
    throwsA(isA<ConsentRequiredError>())
  );
});
```

---

## Related Documentation

- [PRD](PRD.md) - Product requirements
- [HSI Specification](HSI_SPECIFICATION.md) - State representation format
- [Consent System](CONSENT_SYSTEM.md) - Permission management
- [Capability System](CAPABILITY_SYSTEM.md) - Access level enforcement

---

**Last Updated:** 2025-12-25
**Version:** 1.0.0
**Maintained by:** Synheart AI
