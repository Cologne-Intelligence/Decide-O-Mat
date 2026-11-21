# US-014: Restrict Decision Closing

## Description
As a decision owner, I want to be the only one allowed to close my decision, so that I can control when the voting period ends.

## Acceptance Criteria
1.  **Owner Only**: If a decision has an owner, only that user can toggle the "Open/Closed" status.
2.  **UI Feedback**: The "Close Decision" button is hidden or disabled for non-owners.
3.  **Security**: Backend rules reject status updates from non-owners.
4.  **No Ownerless Case**: There is no "anonymous decision with no owner" edge case in practice — `ownerId` is always set on every decision to the creator's Firebase `uid`, including anonymous (guest) users, since the app signs every visitor in anonymously before they can create a decision. The owner-only check therefore always has an owner to compare against.

## Technical Notes
-   Enforced at **both** layers, independently:
    -   **Backend (authoritative)**: the `toggleDecisionStatus` callable function (`functions/index.js`) throws a `permission-denied` error if `request.auth.uid !== decision.ownerId`, regardless of what the client sends.
    -   **Frontend (UX)**: `Decision.jsx` / `MyDecisions.jsx` hide or disable the close/reopen control unless `currentUser.uid === decision.ownerId`, so non-owners never see an actionable control in the first place.

## Implementation Plan
- [IMP-014-Restrict-Closing](../plans/IMP-014-Restrict-Closing.md)
