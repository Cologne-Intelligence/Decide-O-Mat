# User Story: Add Argument (Pro/Con)

**As a** participant
**I want to** add arguments for (Pros) or against (Cons) the decision
**So that** my perspective is considered.

## Acceptance Criteria
1.  **Two Lists**: The decision page displays two distinct lists: "Pros" and "Cons".
2.  **Add Item**: Users can add a text item to either list.
3.  **Validation**: Argument text cannot be empty.
4.  **Real-time Update**: The new item appears immediately for the user and other connected users (if real-time is implemented) or upon refresh.
5.  **Edit Own Argument**: Argument authors can later edit their own argument's text in place — see US-039 (Edit Argument) for the full specification; not covered further here.

## Technical Notes
- Backend: Firebase callable function `addArgument({ decisionId, type, text })`. `type` must be exactly `"pro"` or `"con"`; `text` is trimmed and must be 1-2000 chars. The backend always forces `authorId = request.auth.uid` server-side — any client-supplied authorId is ignored, preventing impersonation. An optional client-supplied `authorName` string is stored as-is (plaintext, not cross-checked against the participant's registered display name).
- Database: Arguments are a sub-collection at `decisions/{id}/arguments/{autoId}` with fields `{ type, text, votes: 0, createdAt: serverTimestamp(), authorId, authorName? }`.
- Submitting an argument implicitly registers the submitter as a decision participant (`ensureParticipant`) if they weren't already recorded.
- There is no cap on the number of arguments a decision or a single user can submit (no spam/rate limiting beyond requiring Firebase App Check + auth).

## Implementation Plan
- [IMP-003-Add-Argument](../plans/IMP-003-Add-Argument.md)
