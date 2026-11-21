# User Story: Edit Argument

**As a** participant who authored a pro or con
**I want to** correct or refine the wording of my own argument after posting it
**So that** I can fix typos or clarify my point without creating a duplicate entry.

## Acceptance Criteria

1. **Authorization & Trigger**: Each argument card shows an edit (pencil) affordance only to the user who authored it (`authorId` matches the current user's uid), and only while the parent decision is open.
2. **Modal**: Clicking it opens the same modal component used for editing the decision question (`EditQuestionModal`), reused here with a different title and a 2000-character max length, pre-filled with the argument's current text.
3. **Validation**: The text is re-validated (1-2000 chars, trimmed) before saving, matching the original creation validation.
4. **Server-Side Enforcement**: Saving calls a dedicated backend function that rejects the edit if the caller is not the original author, or if the decision has been closed in the meantime.
5. **No Edit History**: No edit history or "(edited)" indicator is shown anywhere — the stored text is simply overwritten.
6. **Encryption**: If the decision uses end-to-end encryption, the edited text is re-encrypted with the decision's key before being saved.

## Technical Notes
- Backend: callable function `updateArgumentText({decisionId, argumentId, text})` in `functions/index.js`. Authorization: throws a permission-denied error unless `argumentDoc.authorId === request.auth.uid`. Throws if `decision.status === "closed"`. Overwrites only the `text` field on the argument document — no timestamp or edited-flag field is written alongside it.
- Frontend: `frontend/src/components/StatementCard.jsx` (pencil icon, gated on an `isOwn` check) + `frontend/src/components/EditQuestionModal.jsx` (shared/reused component, not a separate modal).

## Implementation Plan
- Implemented without a dedicated implementation-plan document; this story was backfilled from the shipped code to document existing behavior.
