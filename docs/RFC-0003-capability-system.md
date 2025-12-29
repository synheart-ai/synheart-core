

⸻

RFC-0003: Capability System for Synheart Core
	•	Status: Draft
	•	Authors: Synheart AI
	•	Last Updated: 2025-12-25
	•	Target Version: Synheart Core 1.x
	•	Related RFCs:
	•	RFC-0001: Human State Vector (HSV)
	•	RFC-0002: Consent System

⸻

1. Abstract

This RFC defines the Capability System for Synheart Core.
Capabilities control what applications are allowed to access, while consent controls what users permit.

The system enforces strict, multi-layered access control across SDKs, runtime modules, and cloud ingestion, ensuring privacy-preserving and auditable human-state computation.

⸻

2. Core Principle

Capabilities define what apps CAN access.
Consent defines what users ALLOW.
Data access requires BOTH.

Data Access = Capability (app-level) AND Consent (user-level)

Neither capability nor consent alone is sufficient to access human-state data.

⸻

3. Terminology
	•	Capability: App-level authorization issued by the Synheart Platform.
	•	Consent: User-granted permission for specific data domains.
	•	Tier: A predefined capability level (Core, Extended, Research).
	•	Module: A data-producing subsystem (Wear, Phone, Behavior, HSI Runtime).
	•	HSV: Internal Human State Vector representation.
	•	HSI: Serialized Human State Interface (HSI 1.0). See docs/HSV_AND_HSI_SPECIFICATION.md.

⸻

4. Capability Tiers

Synheart Core defines three capability tiers.

4.1 Core Tier (External Applications)

Intended for
	•	Third-party developers
	•	External SDK users
	•	Public integrations

Grants
	•	Basic HSV axes and indices
	•	Standard time windows (30s, 5m, 1h, 24h)
	•	Normalized embeddings only
	•	HSI 1.0 cloud ingestion
	•	Optional basic interpretation modules

Restricts
	•	Raw biosignals
	•	High-frequency data
	•	Fusion internals
	•	Research endpoints

⸻

4.2 Extended Tier (Synheart Applications)

Intended for
	•	Synheart-owned applications (e.g., Syni Life, SWIP, Pulse Focus)

Grants
	•	Full HSV axes
	•	Full 64D embeddings
	•	Higher-frequency updates
	•	Extended behavior metrics
	•	Advanced interpretation modules
	•	HSI 1.0 cloud ingestion (extended limits)

Restricts
	•	Raw biosignals
	•	Internal fusion vectors
	•	Research-only endpoints

⸻

4.3 Research Tier (Privileged Access)

Intended for
	•	Synheart Research
	•	Authorized research partners
	•	Internal analytics and model evaluation

Grants
	•	Full HSV access
	•	Raw biosignal streams (with consent)
	•	Fusion vectors
	•	Event-level behavior data
	•	Unrestricted time windows
	•	Research cloud endpoints (HSI 1.0)

Constraints
	•	User consent is still mandatory
	•	Privacy boundaries MUST be enforced

NOTE: Although named “Research”, this tier represents privileged internal access and MAY be used for diagnostics, audits, or controlled experiments.

⸻

5. Module-Level Capabilities

5.1 Wear Module

Tier	Signals	Frequency	Format
Core	HR, HRV, sleep stages	1-minute	Aggregates
Extended	HR, HRV, sleep, motion	30-second	Derived signals
Research	Full biosignals	Real-time	Raw + derived


⸻

5.2 Phone Module

Tier	Context	Granularity	Privacy
Core	Screen state, motion	Coarse	Hashed
Extended	Screen, motion, app categories	Medium	Categorized
Research	Full context	Fine	Unhashed


⸻

5.3 Behavior Module

Tier	Metrics	Resolution	Data
Core	Basic counts	Aggregated	Counts only
Extended	Timing patterns	Windowed	Derived
Research	Full events	Event-level	Raw


⸻

5.4 HSI Runtime

Tier	Outputs	Embeddings	Internals
Core	Basic axes	Normalized 64D	No fusion
Extended	Full axes	Full 64D	No fusion
Research	Full axes	Full + fusion	Full internals


⸻

6. Capability Semantics (Normative Extension)

Capabilities apply to operations, not just data.

Each module capability MUST be interpreted as a combination of allowed verbs:
	•	collect
	•	compute
	•	store
	•	export
	•	infer

Example Capability Declaration

{
  "tier": "core",
  "permissions": {
    "wear": ["compute"],
    "behavior": ["compute", "store"],
    "hsi": ["compute"],
    "cloud": ["export"]
  }
}

This allows fine-grained restriction of data flow and storage.

⸻

7. Downgrade Semantics (Normative)

When a capability or consent is insufficient, SDKs MUST apply explicit downgrade behavior.

Downgrade Types

Type	Meaning
null	Data unavailable
redacted	Masked or anonymized
derived	Computed from allowed inputs

Example

"valenceStability": {
  "value": null,
  "reason": "capability_insufficient"
}

Silent omission is NOT permitted.

⸻

8. Capability Provenance in HSI (Normative)

All HSI payloads MUST include a capability context block:

"capability_context": {
  "tier": "core",
  "modules": ["wear", "behavior"],
  "limitations": [
    "aggregated_only",
    "no_raw_biosignals",
    "normalized_embeddings"
  ]
}

This ensures downstream safety and auditability.

⸻

9. Capability Tokens

Capabilities are issued as signed JWTs by the Synheart Platform.

Token Fields

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
    }
  },
  "issuedAt": 1704067200,
  "expiresAt": 1704153600
}

Tokens MUST be:
	•	Signed
	•	Time-bound
	•	Validated locally and server-side

⸻

10. Enforcement Model

10.1 SDK Enforcement
	•	Capability checks at module initialization
	•	Capability checks at data access
	•	Capability checks before cloud upload

10.2 Runtime Enforcement

Modules MUST enforce tier-based behavior internally.

10.3 Server Enforcement

Cloud endpoints MUST revalidate:
	•	Token signature
	•	Tier eligibility
	•	Endpoint access

⸻

11. Capability–Consent Interaction

Capability defines the upper bound of access.
Consent defines the current allowance.

If either is missing → no data.

⸻

12. Capability Auditing

SDKs and cloud services MUST log:
	•	Requested capability
	•	Granted capability
	•	Downgrades
	•	Violations

Repeated violations MAY result in token revocation.

⸻

13. Capability Upgrades

External Developers
	•	Start at Core
	•	Extended requires review and agreement
	•	Research tier is NOT available externally

Internal Applications
	•	Automatically granted Extended
	•	Subject to audit and review

⸻

14. Security Considerations
	•	Token replay prevented via expiry
	•	Forgery prevented via signatures
	•	Module bypass prevented via layered enforcement
	•	Transport security enforced via TLS and pinning

⸻

15. Testing and Mocking

SDKs MUST provide capability mocking for tests.

Mocking MUST NOT be available in production builds.

⸻

16. Open Questions / Future Work
	•	Time-bounded capability escalation
	•	Per-axis capability gating
	•	Regional capability restrictions
	•	Policy-driven dynamic downgrades

⸻

17. Conclusion

The Capability System establishes a privacy-first, enforceable, and auditable access control model for human-state computation.

It enables safe developer innovation while protecting users, platforms, and research integrity.

⸻

