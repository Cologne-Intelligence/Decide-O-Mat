# US-027: Decision Notifications

## Description
As a participant, I want to be notified when there are updates to a decision (new arguments, votes, or status changes), so that I can stay engaged.

## Acceptance Criteria
- [ ] **Opt-In**: Users can toggle notifications on/off per decision (e.g., bell icon).
- [ ] **Channel**: Browser Push Notifications (FCM).
- [ ] **Triggers**:
    - [ ] Decision Closed/Re-opened.
    - [ ] New Argument added.
- [ ] **Link**: Notifications contain a direct link to the decision.
- [ ] **Privacy**: No PII (Email) stored for notifications.

## Technical Notes
- **Token registration**: `frontend/src/services/NotificationService.js`'s `requestPermission()` requests the browser Notification permission, obtains an FCM token via a VAPID key, and writes it **directly onto the caller's own participant document** at `decisions/{id}/participants/{userId}.fcmToken` via a direct client-side Firestore write — the one deliberate exception, in the whole app, to the rule that all writes go through Cloud Functions. `firestore.rules` permits a participant to update *only* their own `fcmToken` field (string, <500 chars) on their own participant doc; every other field/path is denied.
- **Service Worker**: `firebase-messaging-sw.js` is registered unconditionally on every app load, regardless of whether the user has opted in.
- **Triggers** (both in `functions/index.js`): `onArgumentCreate` (fires on creation of any `decisions/{id}/arguments/*` doc) notifies every participant with a stored `fcmToken` except the argument's own author — title "New Argument", body "New argument in: {question, truncated to 47 chars + '...' if longer}". `onDecisionStatusChange` (fires only when the decision's `status` field actually changes) notifies **all** participants with a stored `fcmToken`, including whoever triggered the change — "Decision Closed"/"A decision has been reached." or "Decision Re-opened"/"Additional input is requested."
- **Known gap**: both triggers' notification copy is hardcoded in English only — no i18n is applied, unlike every other user-facing string in the app. Flag as a bug to fix, not a pattern to replicate.
- **Known gap**: the GDPR account-freeze path (`functions/deleteUser.js`) closes an orphaned decision via `isClosed`/`closedAt` fields instead of `status`, so it does **not** fire the "Decision Closed" push — an inconsistency inherited from having two separate close mechanisms (see US-006 and ARCHITECTURE.md's "Known Implementation Inconsistencies").
- **Opt-in state**: tracked purely client-side, derived from whether an `fcmToken` is currently present / `Notification.permission === 'granted'` — there is no separate explicit opt-out flag stored server-side.

## Implementation Plan
- [IMP-027-Notifications](../plans/IMP-027-Notifications.md)
