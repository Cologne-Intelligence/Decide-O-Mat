# User Story: Statistics View

**As a** decision participant or owner
**I want** to view statistics about a decision's voting activity
**So that** I can understand engagement and outcomes at a glance without manually tallying votes and arguments.

## Acceptance Criteria

### Access
1. **Trigger — Context Menu**: "View statistics" action in the archive/My Decisions context menu, deep-linking into the decision via a `?stats=true` query param or router navigation state `openStats`.
2. **Trigger — Election Hero**: The statistics icon in `ElectionHero` is clickable and opens the modal directly.
3. **Visibility**: Available in both Voting mode and Results mode of `ElectionHero`; from the archive, available for both open and closed decisions.
4. **Authorization**: Visible to anyone who can already view the decision — statistics are not owner-only, unlike edit/delete.

### Content
5. **Presentation**: Implemented as `StatisticsModal`, a dedicated modal dialog.
6. **Core Metrics** (implemented):
   - Total final votes cast, plus Yes/No split shown as percentages with a progress bar.
   - "Vote balance" — the signed numeric delta between Yes and No vote counts.
   - Total argument count, plus Pro vs. Con split.
   - "Top argument" — the single highest-voted argument across both Pro and Con columns combined, labeled with a Pro/Con badge.
7. **Data Source**: Computed client-side from the decision's live `finalVotes` subscription; falls back to reading the decision document's own `yesVotes`/`noVotes` fields directly if that subscription hasn't finished loading yet.
8. **Live Data**: Statistics reflect real-time data for open decisions and the frozen final tallies for closed decisions.
9. **Empty State**: Graceful empty state when a decision has no votes or arguments yet.

## Technical Notes
- Implemented in `frontend/src/components/StatisticsModal.jsx`.
- `frontend/src/components/ElectionHero.jsx` renders the `StatsIcon` as an interactive, accessible button wired to open `StatisticsModal`.
- `frontend/src/pages/MyDecisions.jsx` `getContextMenuItems` includes a `viewStatistics` entry alongside `view` / `copyLink` / `close-reopen` / `edit` / `delete`.
- All metrics are computed client-side from existing `arguments` and `finalVotes` subcollections (and the decision document's own counters as a fallback) — no server-side aggregation function was needed.

## Implementation Plan
- Implemented directly against the shipped code described above; no separate implementation-plan document was written for this story.
