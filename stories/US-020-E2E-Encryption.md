# US-020: End-to-End Encryption

## Description
As a user, I want my private decisions to be end-to-end encrypted so that even the server administrator cannot read them.

## Acceptance Criteria
1.  **Encryption**: Decision data (question, arguments) is encrypted on the client before sending to Firestore.
2.  **Key Management**: The decryption key is part of the URL (hash fragment) and never sent to the server.
3.  **Decryption**: Data is decrypted on the client side.

## Technical Notes
-   Implemented in `frontend/src/services/EncryptionService.js` using the `Web Crypto API`: AES-GCM, 256-bit keys.
-   Key export/import format: a raw JWK `k` parameter — a compact base64url string — appended to the decision's URL as a hash fragment `#key=...`. The key is never transmitted to or stored by the backend.
-   Ciphertext format: 12 random IV bytes are prepended to the AES-GCM output, and the combined buffer is base64-encoded for storage in Firestore / transport in URLs.
-   **Opt-in, not universal**: encryption is gated behind a build-time environment flag (`VITE_ENABLE_ENCRYPTION=true`). In the current deployment it is not unconditionally applied to every decision. *Decision needed for the target platform*: make E2E encryption always-on, or keep it an opt-in flag.
-   **Graceful degradation (load-bearing, must be preserved)**: on decrypt, any string shorter than 28 bytes, or any string starting with the literal prefix `"Deleted "`, is passed through as plaintext rather than decrypted. This supports (a) legacy pre-encryption data written before this feature existed, and (b) the backend's GDPR anonymization path (`functions/deleteUser.js`), which writes plaintext `"Deleted <Animal>"` names directly into fields (e.g. `authorName`, `displayName`) that are otherwise end-to-end encrypted.
-   When encryption is enabled for a decision, every subsequent write to that decision's encrypted fields must re-encrypt client-side with that decision's key before saving — this includes editing the question (US-035), adding/editing arguments (US-003/US-039), and registering a display name (US-021/US-026).

## Implementation Plan
- [IMP-020-E2E-Encryption](../plans/IMP-020-E2E-Encryption.md)
