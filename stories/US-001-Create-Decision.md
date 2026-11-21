# User Story: Create Decision

**As a** host (decision initiator)
**I want to** create a new decision topic
**So that** I can invite others to contribute their opinions.

## Acceptance Criteria
1.  **Landing Page**: The application has a landing page with a prominent input field for the decision question/topic.
2.  **Input Validation**: The topic cannot be empty.
3.  **Creation Action**: Clicking "Start" or pressing Enter creates a new decision instance.
4.  **Redirection**: Upon creation, the user is redirected to the unique URL for that decision.
5.  **Persistence**: The decision topic is saved to the database.

## Technical Notes
- Frontend: Simple form with one text input.
- Backend: there is no REST API in this app — only Firebase callable functions and two Firestore triggers. Creation goes through the callable function `createDecision({ question })`, implemented as a Cloud Function v2 in region `europe-west3` with Firebase App Check enforcement (enforced in production only, per `functions/config.js`). The function validates that `question` is a non-empty, trimmed string ≤1000 chars.
- Database: Creates a `decisions/{id}` document with `{ question, createdAt: serverTimestamp(), ownerId: callerUid, participantIds: [callerUid] }`, and also creates a `decisions/{id}/participants/{uid}` document via an internal `ensureParticipant` helper.
- The caller must be authenticated, but this is frictionless in practice because the app auto-signs-in every visitor anonymously on load (see US-021).
- **Optional E2E encryption**: if `VITE_ENABLE_ENCRYPTION=true` (build-time flag — see US-020), an AES-GCM key is generated client-side and the question is encrypted before being sent to `createDecision`; the key is appended to the resulting redirect URL as `#key=...` and is never sent to the backend.

## Implementation Plan
- [IMP-001-Create-Decision](../plans/IMP-001-Create-Decision.md)
