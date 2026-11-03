# US-008: Final Vote

## Description
As a user, I want to be able to cast a final vote (Yes/No) on the decision itself, so that a clear outcome can be reached beyond just the pros and cons list.

## Acceptance Criteria
1.  **Vote Interface**: A clear "Yes" / "No" voting interface is displayed on the decision page.
2.  **One Vote Per User**: Enforced server-side by document path (one `finalVotes/{userId}` doc per user, not local storage). A vote is changeable: casting the same value again is a no-op; casting the opposite value flips it.
3.  **Display Results**: The total count of Yes and No votes is displayed.
4.  **Final Decision**: If the decision is closed, the result is computed as: `"Approved"` if `yesVotes > noVotes`; `"Rejected"` if `noVotes >= yesVotes AND totalVotes > 0` (an exact tie counts as Rejected, not a draw); `"NoVotes"` if nobody voted at all.
5.  **Optimistic UX**: The user's own pending vote is written to `localStorage` (key `decision_vote_{decisionId}`) immediately on click, before the server confirms it, then reconciled against the live Firestore subscription once it loads (governed by an internal `finalVotesLoaded` flag) — this avoids a flash of "unvoted" state on slow connections.

## Technical Notes
-   Backend: callable function `voteDecision({decisionId, vote, displayName})`, `vote` is `"yes"` or `"no"`.
-   One document per user at `decisions/{id}/finalVotes/{userId}` (not a map/array). Casting the opposite value atomically flips it and adjusts the decision's `yesVotes`/`noVotes` counter fields via `FieldValue.increment` inside a Firestore transaction.
-   Voting is blocked once the decision is closed.
-   `Decision.jsx` / `ElectionHero.jsx` render the voting UI.

## Implementation Plan
- [IMP-008-Final-Vote](../plans/IMP-008-Final-Vote.md)
