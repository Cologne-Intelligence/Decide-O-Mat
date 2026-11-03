# User Story: Decision Management Actions

**As a** decision owner
**I want** to edit the question, delete a decision, and view statistics from the archive page
**So that** I can manage my decisions without navigating into each one.

## Acceptance Criteria

### Edit Question
1. **Trigger**: "Edit question" (Figma: "Frage bearbeiten") action from the archive context menu, via `EditQuestionModal`.
2. **Modal Edit**: User modifies the question text in a modal dialog, pre-filled with the current question.
3. **Encryption**: If the decision is E2E encrypted, the question is re-encrypted client-side with the decision's stored key before saving.
4. **Authorization**: Only the decision owner can edit the question (server-enforced).
5. **Real-time Update**: The updated question propagates to all subscribed clients via the live Firestore listener.
6. **Note**: Editing is **not blocked when the decision is closed** — a closed decision's question can still be edited, unlike arguments/votes which are blocked once closed (see Technical Notes).

### Delete Decision
7. **Trigger**: "Delete decision" (Figma: "Entscheidung loeschen") action from the archive context menu, via `ConfirmDeleteDialog`.
8. **Confirmation**: User must confirm the deletion in a dialog before it proceeds.
9. **Cascade Delete**: All subcollections are deleted (arguments and each argument's votes, finalVotes, participants), then the decision document itself.
10. **Authorization**: Only the decision owner can delete a decision (server-enforced).
11. **Feedback**: Toast notification after deletion; client navigates away from the deleted decision.

### View Statistics
12. **Trigger**: "View statistics" (Figma: "Statistik anzeigen") from context menu OR statistics icon in the Election hero.
13. **Content**: See [US-036-Statistics-View](US-036-Statistics-View.md) for the full, implemented specification.

## Technical Notes
- Implemented as Firebase callable functions in `functions/index.js`:
  - `updateDecisionQuestion({decisionId, question})` — owner-only (`decision.ownerId === request.auth.uid`, else `permission-denied`); validates question length ≤1000 chars; does **not** check `decision.status`, so it remains callable on closed decisions (flagged as an inconsistency worth a deliberate decision when porting — argument edits and both voting functions explicitly block on `status === "closed"`, this one doesn't).
  - `deleteDecision({decisionId})` — owner-only; cascading hard-delete of all `arguments` docs plus each argument's `votes` subcollection, all `finalVotes` docs, and all `participants` docs, then the decision document itself — all writes are batched in chunks of 490 to stay under Firestore's 500-writes-per-batch limit.
- `functions/deleteUser.js` contains a related but distinct cascade/freeze pattern used for GDPR account deletion (see US-024) — it does not call `deleteDecision` directly, it has its own logic for deciding whether to cascade-delete or freeze a decision depending on whether the deleted user was the sole participant.
- Frontend: `frontend/src/components/EditQuestionModal.jsx` (shared with argument editing, see [US-039-Edit-Argument](US-039-Edit-Argument.md)) and `frontend/src/components/ConfirmDeleteDialog.jsx`.

## Implementation Plan
- [IMP-035-Decision-Management-Actions](../plans/IMP-035-Decision-Management-Actions.md) — implemented; both callable functions and all three frontend flows (edit, delete, statistics) are shipped.
