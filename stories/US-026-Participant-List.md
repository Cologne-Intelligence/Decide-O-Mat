# US-026: Participant List Sidebar

## Description
As a user viewing a decision, I want to see a list of all participants involved, so that I know who else is collaborating.

## Acceptance Criteria
- [ ] **Web Layout**: A sidebar (or distinct section) displaying the list of users who have joined the decision.
- [ ] **Mobile Layout**: A collapsible drawer or a section at the bottom/top.
- [ ] **Account Type**: Show an icon with the user level of accountability:
    - 👤 **Anonymous / Guest** account
    - 🟢 **Verified / Authenticated** account
  (determined by an `isAnonymous` boolean captured from the Firebase auth token at registration time, not inferred client-side). A short legend at the bottom of the list explains what the two icons mean.
- [ ] **Details**: Show Display Name and Avatar — the user's Firebase `photoURL` if present (Google accounts), otherwise the account-type emoji badge above acts as the avatar fallback.
- [ ] **Ordering**: The decision's owner is always pinned to the top of the list, regardless of join order.
- [ ] **E2E Privacy**: For encrypted decisions, names must be decrypted client-side. Unknown/unencrypted participants should show appropriate placeholders or fallback.

## Technical Notes
- Data source: `decisions/{id}/participants/{userId}` documents, each holding either `encryptedDisplayName` (E2EE-enabled decisions, decrypted client-side) or `plainDisplayName` / legacy `displayName` fields (non-encrypted decisions or older data), plus `isAnonymous` and optional `photoURL`.
- Subscribed to via `ParticipantService`. This story is mainly about the *UI* realization (Sidebar).

## Implementation Plan
- [IMP-026-Participant-List](../plans/IMP-026-Participant-List.md)
