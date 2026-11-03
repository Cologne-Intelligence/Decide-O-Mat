# US-012: Private Decision Mode

> **Status: NOT IMPLEMENTED.** Current Firestore security rules (`firestore.rules`) grant read access to `decisions/*` and all its subcollections to **any authenticated user**, including anonymous users — there is no per-decision membership check. There is no `visibility` field on decision documents, no invite-by-email mechanism, and no participant-restricted read rule anywhere in the current codebase. Anyone who has or guesses a decision's URL/ID can read its full contents (subject to end-to-end encryption, if enabled — see US-020, which protects content confidentiality but not access/existence). This story remains a valid, unimplemented product proposal for a future platform — it does NOT describe current behavior. Treat it as a roadmap item, not a porting target.

## Description
As a user, I want to create private decisions that are only accessible to specific people, so that sensitive topics can be discussed securely.

## Acceptance Criteria
1.  **Visibility Toggle**: When creating a decision, choose between "Public" (anyone with link) and "Private" (invite only).
2.  **Invite System**: Add participants by email.
3.  **Access Control**: Only invited users (and owner) can view/interact with the decision.
4.  **Security**: Firestore rules enforce access based on the participant list.

## Technical Notes
-   Add `visibility` ('public', 'private') and `participants` (array of emails/uids) to Decision schema.
-   Update Firestore rules to check `request.auth.uid` against `participants` or `ownerId` for private decisions.
-   UI for managing participants.

## Implementation Plan
- [IMP-012-Private-Decisions](../plans/IMP-012-Private-Decisions.md)
