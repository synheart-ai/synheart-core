RFC-0002: Consent System for Synheart Core
	•	Status: Draft
	•	Authors: Synheart AI
	•	Last Updated: 2025-12-25
	•	Target Version: Synheart Core 1.x
	•	Related RFCs:
	•	RFC-0001: Human State Vector (HSV)
	•	RFC-0003: Capability System

⸻

1. Abstract

This RFC defines the Consent System for Synheart Core. The Consent Module manages user permissions and enforces privacy boundaries across all modules (Wear, Phone, Behavior, Interpretation Modules, Cloud Connector). Consent is explicit, granular, revocable, enforced, and transparent.

Consent is required for data collection and state computation. Synheart Core MUST NOT collect data or generate synthetic substitutes when consent is missing.

⸻

2. Core Principles
	1.	Explicit Consent: All data collection requires explicit user consent.
	2.	Granular Control: Users control each module independently.
	3.	Revocable: Consent can be withdrawn at any time.
	4.	Enforced: Missing consent = no data collection (not synthetic).
	5.	Transparent: Users can see exactly what is collected and why.

⸻

3. Terminology
	•	Consent Type: A named permission for a module or interpretation (e.g., biosignals, focusEstimation).
	•	Consent State: {granted, timestamp, sdkVersion} plus optional metadata.
	•	Consent Scope: Per-user, per-device.
	•	Revocation: Transition from granted → denied, effective immediately.
	•	Capability: App-level authorization (RFC-0003).
	•	HSI/HSV: State outputs (RFC-0001).

⸻

4. Consent Types

4.1 Module-Level Consents

Consent Type	Module	Enables	Default
biosignals	Wear	HR, HRV, sleep, motion (derived)	false
phoneContext	Phone	Motion, screen state, app context	false
behavior	Behavior	Taps, scrolls, typing cadence (timing only)	false
cloudUpload	Cloud Connector	Upload derived state (HSI 1.0)	false

4.2 Interpretation Module Consents

Consent Type	Module	Enables	Default
focusEstimation	Synheart Focus	Focus score computation	false
emotionEstimation	Synheart Emotion	Emotion/stress inference	false

Normative requirement: Interpretation consents MUST be independent of collection consents. If an interpretation is enabled but upstream signals are missing, the output MUST be null with an explicit reason (see §8).

⸻

5. Consent Flow

5.1 Initial Consent Request

On SDK initialization, the SDK MUST check for stored consent. If consent is missing:
	1.	Return empty/null state until consent is granted
	2.	Provide consent request callbacks to the host app
	3.	MUST NOT collect any data

5.2 Consent Storage

Consent MUST be stored:
	•	Locally, using on-device encrypted storage
	•	Per-device (not synced by default)
	•	Versioned (tied to SDK version)

Storage shape (illustrative):

{
  "userId": "anon_user_123",
  "consents": {
    "biosignals": {"granted": true, "timestamp": 1704067200, "sdkVersion": "1.0.0"},
    "phoneContext": {"granted": true, "timestamp": 1704067200, "sdkVersion": "1.0.0"},
    "behavior": {"granted": false, "timestamp": null, "sdkVersion": null},
    "cloudUpload": {"granted": false, "timestamp": null, "sdkVersion": null}
  }
}

Comment / recommendation (strong): Add consent_text_version and policy_version to consent records so you can prove what the user agreed to at the time:

"biosignals": {
  "granted": true,
  "timestamp": 1704067200,
  "sdkVersion": "1.0.0",
  "policyVersion": "2025-12-01",
  "consentTextVersion": "biosignals_v2"
}

5.3 Consent Revocation

Users can revoke consent at any time.

Revocation MUST:
	•	Stop data collection immediately
	•	Stop cloud uploads immediately (for cloudUpload)
	•	Cause dependent HSV/HSI axes to become null
	•	Not delete existing local data automatically (separate deletion APIs)

⸻

6. Enforcement Mechanisms

6.1 Module-Level Enforcement

Each module MUST check consent before any data collection or processing. If consent is missing, the module MUST return without collecting data.

6.2 HSV Runtime Enforcement

HSV computation MUST respect consent boundaries. Missing consent MUST result in null values (not 0.0).

6.3 Cloud Upload Enforcement

Cloud upload MUST require explicit cloudUpload consent and MUST be re-checked before every upload.

⸻

7. Consent UI Requirements

Apps integrating Synheart Core SHOULD provide:
	1.	Contextual requests (ask when needed)
	2.	Clear explanations of what/why
	3.	Granular toggles per module
	4.	Easy revocation in settings
	5.	“Learn more” links or disclosures

Normative requirement: The SDK MUST provide enough metadata for apps to render accurate consent screens (e.g., list of data categories per consent type).

⸻

8. Consent Outcome Semantics (Normative Extension)

When consent is missing, outputs MUST be explicit and machine-readable.

8.1 Null + Reason Codes

Any computed axis that depends on a denied consent MUST be:
	•	value: null
	•	reason: one of:
	•	consent_denied
	•	consent_missing
	•	consent_expired (see §12)
	•	capability_insufficient (RFC-0003)
	•	dependency_missing

Example (illustrative):

"engagementStability": {
  "value": null,
  "reason": "consent_denied",
  "dependsOn": ["behavior"]
}

This prevents silent failure and makes downstream systems safe by default.

⸻

9. Consent & Capability Interaction

Consent and capability are distinct and both required.

Concept	Enforces	Scope
Consent	User permission to collect/process	Per-user, per-device
Capability	App authorization	Per-app, server-issued

Normative rule:

Data Collection/Processing = Consent (user) AND Capability (app)

Important correction vs your draft text: You wrote “Consent can compute arousalIndex” for an external app. That is true ONLY if (a) consent is granted AND (b) the app capability includes the required module verbs (e.g., collect/compute for wear). Keep the combined rule consistent everywhere.

⸻

10. Data Retention & Deletion

10.1 Local Data

Synheart Core SHOULD store minimal data:
	•	HSV snapshots (rolling window)
	•	Consent preferences
	•	Module caches

User controls MUST include:
	•	Delete all local data
	•	Delete per-module data

10.2 Cloud Data

If cloudUpload is granted, derived snapshots (HSI 1.0) may be uploaded.

User controls MUST include:
	•	Stop uploads via revocation
	•	Delete cloud data via platform API

Comment / recommendation: Consider “local deletion on revoke” as an optional policy flag for stricter deployments (e.g., healthcare pilots), but keep your default behavior as-is.

⸻

11. Privacy Guarantees

11.1 Prohibited Collection

Even with full consent, Synheart Core MUST NOT collect:
	•	Raw ECG/PPG waveforms
	•	Message content / URLs
	•	Keyboard content
	•	Specific app names (outside privileged tiers)
	•	Audio/microphone
	•	Location/GPS
	•	Photos/media
	•	Contacts / direct identifiers

11.2 Allowed Collection (with consent)

With appropriate consent, Synheart Core MAY collect:
	•	Derived biosignals (HR, HRV, sleep stages)
	•	Motion patterns
	•	Screen state
	•	Interaction timing (not content)
	•	Hashed app context
	•	HSV state representations

⸻

12. Consent Versioning & Re-consent

Consent MUST be re-requested when:
	1.	New data types are collected under an existing consent
	2.	Privacy policy meaningfully changes
	3.	Module behavior changes in ways that affect user expectations

SDK MUST expose:
	•	isConsentValid(type) for the current SDK version/policy version
	•	A consistent re-consent flow

Normative extension: Define “meaningful change” via consentTextVersion and policyVersion fields (see §5.2). This makes audits and legal review straightforward.

⸻

13. Auditability (Normative Extension)

SDK MUST emit audit events (local log + optional app callback):
	•	consent_requested
	•	consent_granted
	•	consent_denied
	•	consent_revoked
	•	consent_invalidated (due to versioning)

Each event SHOULD include:
	•	consent type
	•	timestamp
	•	sdk version
	•	policy/consent text version
	•	app identifier (if available)

⸻

14. Testing Requirements

SDK MUST support:
	•	Consent mocking for tests
	•	Guaranteed disabling of mocks in production builds

A compliance test suite SHOULD include:
	•	“revocation makes outputs null”
	•	“no synthetic data without consent”
	•	“cloud upload stops immediately on revoke”
	•	“interpretation requires interpretation consent + dependencies”

⸻

15. Security Considerations
	•	Consent storage MUST be encrypted at rest
	•	Consent checks MUST be enforced inside modules (not UI only)
	•	Consent revocation MUST take effect immediately
	•	Capability and consent checks MUST be defense-in-depth (SDK + server)

⸻

16. Future Work
	•	Time-bounded consent (study windows)
	•	Region- or jurisdiction-specific consent text variants
	•	Optional “delete on revoke” policies
	•	Per-axis consent granularity (only if truly needed)

⸻

17. Conclusion

The Consent System provides explicit, granular, revocable, enforced, and transparent privacy control for Synheart Core. It composes with the Capability System to ensure app authorization and user permission are both required for access, computation, and export of human-state data.

⸻

If you want, I can now:
	•	Align RFC-0002 + RFC-0003 terminology (so the combined “Data Access = capability AND consent” appears identically in both), and/or
	•	Produce a single “Access Control Model” diagram that shows: App → Capabilities → Modules → Consent Gate → HSV → HSI → Cloud.