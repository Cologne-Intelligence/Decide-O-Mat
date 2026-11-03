# Architecture: Decide-O-Mat

## 1. Overview
Decide-O-Mat is a serverless, single-page web application designed for high scalability and minimal operational cost. It leverages Google Cloud Platform (GCP) services to ensure the application scales to zero when not in use and handles high traffic spikes automatically.

## 2. System Design

### 2.1. High-Level Architecture
The system follows a standard 3-tier serverless architecture:
1.  **Client**: A Single Page Application (SPA) running in the user's browser.
2.  **API Layer**: Stateless Cloud Functions serving as the backend API.
3.  **Data Layer**: A managed NoSQL database for persistence and real-time updates.

```mermaid
graph LR
    Client[Client (Browser)]
    CDN[CDN / Hosting]
    API[Cloud Functions]
    DB[(Firestore)]

    Client -- Loads Assets --> CDN
    Client -- API Calls (HTTPS) --> API
    API -- Read/Write --> DB
    Client -- Real-time Listeners --> DB
```

### 2.2. Data Flow
1.  **Read**: The client subscribes directly to Firestore documents for real-time updates (Optimistic UI).
2.  **Write**: Critical writes (creating decisions, voting) are routed through Cloud Functions to ensure business logic validation and data integrity before hitting the database.

## 3. Technology Choices & Rationale

### 3.1. Frontend
- **Technology**: React (built with Vite)
- **Rationale**:
    - **React**: Component-based architecture suitable for interactive UIs ("doodle-like").
    - **Vite**: Fast build tool and development server.
    - **Hosting**: Firebase Hosting (or GCS Bucket behind Cloud CDN) for global low-latency delivery.

### 3.2. Backend
- **Technology**: Google Cloud Functions (2nd Gen)
- **Runtime**: Node.js
- **Rationale**:
    - **Scale to Zero**: No cost when idle, perfect for sporadic usage patterns.
    - **Scalability**: Automatically handles concurrent requests.
    - **Node.js**: Shared language (JavaScript/TypeScript) with frontend, simplifying development.

### 3.3. Database
- **Technology**: Google Cloud Firestore
- **Rationale**:
    - **Real-time**: Built-in support for real-time listeners, essential for the collaborative aspect.
    - **Scalability**: Horizontal scaling for massive read/write loads.
    - **Flexible Schema**: NoSQL nature allows easy iteration on data models (e.g., adding new argument types).

## 4. Data Model (Schema Design)

> The schema below reflects the actual implementation (`functions/index.js`, `firestore.rules`) as of v2.0.2, not the original MVP proposal. All writes except `participants.fcmToken` go through Firebase callable functions — there is no REST API.

### 4.1. Collection: `decisions`
- `question` (string, ≤1000 chars, or ciphertext if E2E encryption is enabled — see US-020)
- `createdAt` (timestamp)
- `ownerId` (string): Firebase uid of the creator. Always set — including for anonymous creators. There is no ownerless decision.
- `participantIds` (string[]): uids of everyone who has created, argued, or voted on this decision. Backs the "My Decisions" owner+participant query (US-025).
- `status` ("open" | "closed"): set exclusively by the `toggleDecisionStatus` callable (US-006). This is the canonical closed/open signal used by the UI and by the `onDecisionStatusChange` push-notification trigger.
- `yesVotes`, `noVotes` (number): counters maintained atomically by `voteDecision` (US-008).
- `isClosed` (bool), `closedAt` (timestamp), `ownerDisplayName` (string): written only by the GDPR account-deletion "freeze" path (`functions/deleteUser.js`, US-024) when a decision's sole owner deletes their account but other participants remain. **This is a second, parallel "closed" representation that does not update `status` and therefore does not fire `onDecisionStatusChange` — a known inconsistency, not an intentional two-state design (see §6 below).**

### 4.2. Sub-collection: `decisions/{decisionId}/arguments`
- `type` ("pro" | "con")
- `text` (string, 1-2000 chars, or ciphertext)
- `votes` (number): counter, maintained atomically.
- `createdAt` (timestamp)
- `authorId` (string): always server-set to `request.auth.uid`, never trusted from the client.
- `authorName` (string, optional): client-supplied, plaintext, not cross-validated against the author's registered participant name.

Editable post-creation via `updateArgumentText` (author-only, blocked once the decision is closed — US-039).

#### Sub-collection: `decisions/{decisionId}/arguments/{argumentId}/votes`
- Document ID = voter's uid (enforces one vote per user per argument).
- `userId`, `displayName`, `createdAt`.
- A user may hold an active vote on at most `ceil(totalArguments / 2)` arguments at once, counted across pros and cons combined — a dot-voting prioritization cap, not an upvote/downvote score (US-004).

### 4.3. Sub-collection: `decisions/{decisionId}/finalVotes`
- Document ID = voter's uid (one final yes/no vote per user, changeable).
- `vote` ("yes" | "no"), `userId`, `displayName`, `createdAt`/`updatedAt`.
- Closed-decision outcome: "Approved" if `yesVotes > noVotes`; "Rejected" if `noVotes >= yesVotes && totalVotes > 0` (exact ties are Rejected); "NoVotes" if nobody voted (US-008).

### 4.4. Sub-collection: `decisions/{decisionId}/participants`
- Document ID = participant's uid.
- `encryptedDisplayName` (E2EE decisions) or `plainDisplayName` / legacy `displayName` (non-encrypted).
- `isAnonymous` (bool, from the auth token's `sign_in_provider`), `photoURL` (optional, Google accounts).
- `fcmToken` (string, optional): the **only** field in the entire schema a client may write directly (not via a callable) — `firestore.rules` permits a participant to update just this one field on their own document, to register for push notifications (US-027).

There is no `visibility`/invite-list field anywhere in the schema — "Private Decisions" (US-012) is an unimplemented proposal, not current behavior. Any authenticated user (including anonymous) who has a decision's id/URL can read it in full.

## 5. Non-Functional Requirements (NFRs)

### 5.1. Scalability
- **Horizontal Scaling**: The system must handle 1000+ concurrent users on a single decision without degradation.
- **Database Limits**: Firestore has a write limit of ~1 write/sec per document. To bypass this for voting, we will use **Distributed Counters** (sharding counters) if high-concurrency voting becomes a bottleneck.

### 5.2. Performance
- **Cold Starts**: Cloud Functions can have cold starts. We will use `min-instances: 0` to save costs but optimize code size to keep cold starts under 2s.
- **Latency**: Global CDN for static assets to ensure <1s Time to Interactive (TTI).

### 5.3. Security (as actually implemented)
- **Capability URLs**: Decision IDs are Firestore auto-generated document IDs (not UUIDv4), long/random enough to resist enumeration. For E2E-encrypted decisions, the decryption key travels only in the URL hash fragment (`#key=...`), never sent to the server (US-020).
- **No REST surface, no CORS policy needed**: every mutation is a Firebase callable function invoked via the Firebase client SDK (`httpsCallable`), not a hand-rolled HTTP API — CORS is handled by the callable-functions transport itself (`cors: true` in the function options), not a custom policy.
- **App Check**: enforced on all callable functions, but only in the production project (`GCLOUD_PROJECT === "decide-o-mat"` — see `functions/config.js`); staging and the local emulator run without it.
- **Firestore rules**: deny all direct client writes except one field (`participants/{uid}.fcmToken`, for push notification registration — US-027); reads require any authenticated session (including anonymous), with no per-decision membership check (so "Private Decisions," US-012, is not enforced server-side).
- **Validation**: input length/enum validation (question/text length caps, vote enum values) happens inside each callable function, not at a separate API gateway layer.

### 5.4. Cost Efficiency
- **Free Tier**: The architecture should fit within the GCP Free Tier for low-to-moderate usage.
- **Optimization**: Use Firestore caching and limit snapshot listeners to active windows to minimize read costs.

## 6. Known Implementation Inconsistencies

These are real quirks in the current codebase, not intentional design — worth a deliberate decision (fix or consciously replicate) when porting to another platform:

1. **Two incompatible "decision closed" representations.** `toggleDecisionStatus` (US-006) sets a `status` enum (`"open"`/`"closed"`), which both the UI and the `onDecisionStatusChange` push trigger key off. Separately, the GDPR account-deletion "freeze" path (`functions/deleteUser.js`, US-024) closes an orphaned decision by setting `isClosed`/`closedAt`/`ownerDisplayName` instead — which the UI and trigger never check, so a frozen decision silently never fires the "Decision Closed" notification and may not render as closed everywhere `status` is checked directly.
2. **Inconsistent closed-state enforcement across mutators.** `updateArgumentText`, `voteArgument`, and `voteDecision` all reject writes once `decision.status === "closed"`. `updateDecisionQuestion` (US-035) does not check `status` at all — the question remains editable after closing.
3. **Push notification copy is hardcoded English-only** (`onArgumentCreate`, `onDecisionStatusChange` in `functions/index.js`) — unlike every other user-facing string in the app, which goes through i18next with `en.json`/`de.json`.
4. **The "vote" on an argument is a toggle, not a signed value.** It's dot-voting with a shared prioritization cap (`ceil(totalArguments/2)` across pros+cons combined) via `voteArgument` — there is no downvote, and the original MVP spec's "Upvote/Downvote" language (US-004, pre-correction) never matched what was built.
5. **The anonymous-user animal-name list is duplicated** between `frontend/src/utils/NameGenerator.js` (guest display names, US-040) and `functions/deleteUser.js` ("Deleted {Animal}" pseudonyms, US-024). Keeping these in sync is manual; a port should centralize them into one source of truth.
