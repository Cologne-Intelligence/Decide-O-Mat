# US-025: My Decisions List

## Description
As a logged-in user, I want to see a list of decisions I have created or participated in, so that I can easily find and revisit them.

## Acceptance Criteria
- [ ] **My Decisions View**: A new page or dashboard section displaying a list of decisions.
- [ ] **Filter**: Separate "Created by Me" and "Participated In" (if possible).
- [ ] **Display**: Show Title, Date, and Status (Open/Closed).
- [ ] **Navigation**: Clicking a decision navigates to the decision page.
- [ ] **Synchronization**: The list updates when I join new decisions.
- [ ] **Legacy Data**: If I upgrade from anonymous to logged-in, my previous decisions (tracked by anonymous ID) should appear here.

## Technical Notes
- Implemented via two Firestore composite indexes (`firestore.indexes.json`): one on `ownerId + createdAt desc`, one on `participantIds array-contains + createdAt desc`. The "My Decisions" list is the **union** of these two queries — decisions I own, plus decisions I've participated in.
- `participantIds` is a denormalized string array directly on the `decisions/{id}` document (not a separate collection/subcollection query). Any vote, argument submission, or final vote server-side adds the acting user's uid into this array, so simply interacting with a decision is sufficient to make it appear in this list — no explicit "join" step is required.

## Implementation Plan
- [IMP-025-My-Decisions](../plans/IMP-025-My-Decisions.md)
