# User Story: Vote

**As a** participant
**I want to** mark specific pros and cons as important, within a limited budget
**So that** the group can see which arguments the participants most want to prioritize.

## Acceptance Criteria
1.  **Voting UI**: Each argument (Pro/Con) has a single toggle "approve" control — there is no separate upvote or downvote, no signed value. A vote is binary: voted or not voted.
2.  **Prioritization Cap (dot-voting)**: Each participant may have an active vote on at most `Math.ceil(totalArguments / 2)` arguments at any one time, counted across BOTH pro and con arguments combined (not per-column). This recalculates live as arguments are added or removed, forcing participants to prioritize once there are more than a couple of arguments.
3.  **Toggle Behavior**: Clicking an already-voted argument removes (un-votes) it. Clicking an un-voted argument while under the cap adds a vote. Attempting to vote while already at the cap is rejected by the backend.
4.  **Vote Count & Voter Chips**: The argument's displayed vote count updates atomically, and a live list of voter display names ("Approved by ...") renders under the argument.
5.  **Identification Required**: Voting requires authentication, satisfied automatically by the app's always-on anonymous sign-in (see US-021); a first-time voter with no display name set is prompted via a name modal before the vote is recorded, and the vote action is queued and replayed automatically once the name is saved.
6.  **Closed Decisions**: Votes are blocked once the parent decision's status is "closed".
7.  **Implicit Participation**: Voting on an argument implicitly registers the voter as a decision participant if not already registered.

## Technical Notes
- Backend: callable function `voteArgument({ decisionId, argumentId })` in `functions/index.js`. The server recomputes the cap from the live count of arguments in the decision and rejects votes beyond it (permission-denied-style error).
- Database: The vote is stored as its own document at `decisions/{id}/arguments/{argumentId}/votes/{userId}` = `{ userId, displayName, createdAt }`; toggling off deletes this document and decrements the parent argument's `votes` counter. Both the create/delete and the counter increment/decrement happen inside one Firestore transaction (`runTransaction`) for atomicity.
- A new vote also adds the caller's uid into the parent decision's `participantIds` array (this is NOT removed again when the vote is retracted).
- This supersedes an earlier, never-implemented upvote/downvote design that this story originally described.

## Implementation Plan
- [IMP-004-Vote](../plans/IMP-004-Vote.md)
