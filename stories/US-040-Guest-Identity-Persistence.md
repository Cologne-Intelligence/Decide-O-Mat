# User Story: Guest Identity Persistence, Naming & Reset

**As a** guest (anonymous) user
**I want to** have my display identity persist across page reloads and feel personal, without having to sign up, and be able to recover gracefully from edge cases
**So that** I can participate naturally and be recognized by other participants over time.

## Acceptance Criteria

1. **Automatic Sign-In**: On first app load with no existing Firebase session, the app automatically and silently signs the user in via Firebase Anonymous Authentication (`signInAnonymously`) — there is no explicit "Continue as Guest" button or step; this happens for every visitor, immediately.
2. **Generated Name**: A friendly placeholder display name of the form "Anonymous {Animal}" is generated from a fixed list of roughly 140 animal names and persisted in `localStorage`, alongside the Firebase uid it was generated for and the original/initial name (for later recovery).
3. **UID Change Handling**: If the Firebase auth UID changes (for example, because browser storage was cleared and a brand-new anonymous session was created), a fresh animal name is generated for the new UID rather than reusing a stale stored name tied to the old UID.
4. **Legacy Session Healing**: If the stored name predates UID-tracking (a legacy/first-run session from before this mechanism existed), that name is "claimed"/synced onto the current Firebase user's profile `displayName` rather than being regenerated, to avoid unnecessarily renaming returning users.
5. **Edit & Reset**: The user can edit their display name at any time via the User Settings panel, and can also explicitly reset it back to the original auto-generated name first assigned to them — the reset flow shows a preview of that original name before the user confirms.
6. **Local vs. Per-Decision Name**: The generated/stored local display name is purely a local convenience default — it is not required to match the `authorName`/participant display-name values stored per-decision in Firestore; the two can diverge over time.

## Technical Notes
- Implemented in `frontend/src/contexts/UserContext.jsx` (the name-reconciliation state machine that runs on every auth-state change) and `frontend/src/utils/NameGenerator.js` (the animal name list and random-name generator).
- LocalStorage keys used: `dom_display_name`, `dom_display_name_uid`, `dom_initial_name`.
- The same (approximately 140-entry) animal name list is duplicated in the backend at `functions/deleteUser.js`, which uses it to generate consistent "Deleted {Animal}" pseudonyms during GDPR account deletion/anonymization (see [US-024-User-Profile](US-024-User-Profile.md)) — both lists must be kept in sync; porting to another platform should centralize this into one shared list.
- `resetToInitialName()` and `getInitialName()` are exposed from `UserContext` and surfaced as the "Reset" action in `frontend/src/components/UserSettings.jsx`.

## Implementation Plan
- Implemented without a dedicated implementation-plan document; this story was backfilled from the shipped code to document existing behavior.
