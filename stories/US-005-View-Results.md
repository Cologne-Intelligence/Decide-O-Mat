# User Story: View Results

**As a** participant
**I want to** see the current consensus/score
**So that** I know which option is currently favored.

## Acceptance Criteria
1.  **Net Score**: The page displays a "Net Score" calculated as (Total Pro Votes - Total Con Votes).
2.  **Visual Indicator**: A visual cue (e.g., color or icon) indicates whether the decision is currently leaning "Yes" (Positive Score) or "No" (Negative Score).
3.  **Live/Refresh**: The score updates when the page is refreshed or via real-time updates.

## Technical Notes
- Frontend: Calculate the sum of votes from the loaded arguments. Arguments are sorted by vote count descending within each pro/con column (not by creation order).
- There is no separate dedicated "results page" — the same `Decision.jsx` page/component renders both a "voting" visual mode and a "results" visual mode depending on the decision's `status` field (open vs closed), using different Election Hero variants (thumbs-up/down hero for voting, ballot-icon hero for results — see US-032/US-033).
- Backend: Can optionally aggregate scores if performance becomes an issue, but client-side aggregation is fine for MVP.
- For a deeper breakdown (totals, yes/no split %, vote balance, argument counts, top argument), see US-036 (Statistics View).

## Implementation Plan
- [IMP-005-View-Results](../plans/IMP-005-View-Results.md)
