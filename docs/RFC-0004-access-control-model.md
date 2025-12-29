# RFC-0004: Access Control Model for Synheart Core

- **Status**: Draft
- **Authors**: Synheart AI
- **Last Updated**: 2025-12-29
- **Target Version**: Synheart Core 1.x
- **Related RFCs**:
  - **RFC-0001**: Human State Vector (HSV)
  - **RFC-0002**: Consent System
  - **RFC-0003**: Capability System

---

## 1. Abstract

This RFC defines the Access Control Model for Synheart Core.
It formalizes how capabilities (app-level authorization) and consent (user-level permission) combine to determine:
- Whether data may be collected
- Whether state may be computed
- What outputs are returned
- How denials and downgrades are represented

This model is the single source of truth for access decisions across SDKs, modules, runtime computation, and cloud ingestion.

---

## 2. Design Goals

The Access Control Model MUST be:
1. Deterministic – same inputs always yield the same outcome
2. Explicit – denials are visible and machine-readable
3. Composable – capability and consent evolve independently
4. Enforceable – works at SDK, runtime, and server layers
5. Auditable – outcomes explain why access was granted or denied

---

## 3. Core Principle (Normative)

Access is granted only when BOTH authorization and permission are present:

```
Access Allowed
  = Capability(app, module, verb)
  AND Consent(user, consent_type)
```

- Capability defines the maximum power an app may exercise (RFC-0003)
- Consent defines the current permission a user grants (RFC-0002)
- Neither can override the other

---

## 4. Access Decision Model

### 4.1 Inputs

| Input | Source | Scope |
|---|---|---|
| Capability tier | Platform token | Per-app |
| Module capability | Platform token | Per-module |
| Allowed verbs | Platform token | Per-module |
| Consent state | Local store | Per-user, per-device |
| Operation | Runtime | collect / compute / store / export / infer |

---

### 4.2 Decision Function (Normative)

```
IF capability_missing
  → DENY (reason = capability_insufficient)

IF consent_missing_or_denied
  → DENY (reason = consent_denied)

IF operation_not_allowed_by_capability
  → DENY (reason = capability_insufficient)

ELSE
  → ALLOW
```

Denial MUST NOT fall back to synthetic or inferred data.

---

## 5. Enforcement Pipeline

The model MUST be enforced at multiple layers:
1. SDK initialization (token validation + consent cache load)
2. Module entry (capability check + consent check)
3. Data collection (blocked if either check fails)
4. HSV/HSI computation (restricted outputs set to null + reason codes attached)
5. Cloud upload (requires cloud capability + cloudUpload consent + server revalidation)

No single layer is sufficient on its own.

---

## 6. Outcome Semantics (Normative)

All denied or downgraded outputs MUST be explicit.
Silent omission is NOT permitted.

### 6.1 Output Rules

| Condition | Output |
|---|---|
| Allowed | Full value |
| Consent denied/missing | null + consent_denied |
| Capability insufficient | null + capability_insufficient |
| Dependency missing | null + dependency_missing |

### 6.2 Example Output

```json
"engagementStability": {
  "value": null,
  "reason": "consent_denied",
  "dependsOn": ["behavior"]
}
```

---

## 7. Downgrade Rules (Normative)

When partial access is available:
- Higher-tier requests MUST downgrade to the highest allowed tier
- Downgrades MUST be logged
- Downgraded outputs MUST be clearly marked

Downgrade type semantics MUST follow RFC-0003 (§7): null / redacted / derived.

---

## 8. Provenance in HSI (Normative)

When exporting HSI 1.0, payloads MUST include capability provenance as defined in RFC-0003 (§8).
HSI 1.0 schema reference: docs/HSV_AND_HSI_SPECIFICATION.md

---

## 9. Interpretation Modules

Interpretation modules (e.g., Focus, Emotion) require:
1. Capability to compute the interpretation
2. Explicit interpretation consent
3. All upstream dependencies satisfied

If any dependency is missing:
- Output MUST be null
- Reason MUST be provided

---

## 10. Cloud Access Model

Cloud access is governed by the same model:
- App capability includes cloud export
- User consent includes cloudUpload
- Payload includes capability provenance block

Server-side validation MUST re-enforce capability checks.

---

## 11. Auditability

All access decisions SHOULD be logged with:
- App identifier
- Module
- Operation
- Capability tier
- Consent state
- Outcome
- Reason (if denied or downgraded)

---

## 12. Security Considerations

- Tokens MUST be time-bound and signed (RFC-0003)
- Consent MUST be encrypted at rest (RFC-0002)
- Checks MUST be enforced inside modules
- Bypassing UI checks MUST NOT grant access

---

## 13. Non-Goals

This RFC does NOT define:
- UI flows (see RFC-0002)
- Capability issuance policy (see RFC-0003)
- Data schemas (see RFC-0001 + docs/HSV_AND_HSI_SPECIFICATION.md)

It defines decision logic only.

---

## 14. Future Extensions

- Time-bounded capability or consent windows
- Region-specific access rules
- Per-axis capability gating
- Policy-driven dynamic downgrades

---

## 15. Conclusion

RFC-0004 establishes the canonical access decision procedure for Synheart Core.
All modules and services MUST conform to this model.

