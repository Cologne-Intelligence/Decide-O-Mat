# User Story: Close Decision

**As a** participant (or creator)
**I want to** close a decision
**So that** no further votes or arguments can be added, finalizing the result.

## Acceptance Criteria
1.  **Close Action**: A button to "Close Decision" is available on the decision page.
2.  **Visual Indication**: When closed, the decision page clearly indicates the status (e.g., "Decision Closed").
3.  **Read-Only Mode**:
    - Voting buttons are disabled or hidden.
    - "Add Argument" forms are disabled or hidden.
4.  **Re-open**: Re-opening a closed decision is a first-class, fully supported action (not merely optional) and immediately re-enables voting/arguments for everyone.
5.  **Owner always exists**: `ownerId` is always set to the creator's Firebase uid, including anonymous users — there is no "ownerless" decision case to handle.

## Technical Notes
- **Implementation**: Firebase callable function `toggleDecisionStatus({decisionId, status})` in `functions/index.js`. Owner-only — the server checks `decision.ownerId === request.auth.uid` and throws `permission-denied` otherwise. Toggles the decision's `status` field between `"open"` and `"closed"`.
- **Security**: All decision/argument/vote writes go through callable functions (`firestore.rules` denies direct client writes); callables check `status` before allowing argument/vote mutations.
- **Frontend**: The Decision page subscribes to the decision document in real time, so the open/closed UI updates live for all connected viewers without a refresh.
- **Known inconsistency**: a separate account-deletion code path can also freeze a decision using different fields (`isClosed`/`closedAt`) instead of `status` — this does not update `status` and does not trigger the close/reopen push notification. See ARCHITECTURE.md "Known Implementation Inconsistencies" and US-027.

## Implementation Plan
- [IMP-006-Close-Decision](../plans/IMP-006-Close-Decision.md)
