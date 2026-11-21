# US-011: User Dashboard

> **Superseded.** The dashboard concept described below shipped as the "My Decisions" / Archive page. See US-025 (My Decisions List) and US-034 (Archive Page Redesign) for the accurate, current specification (route `/my-decisions`, component `frontend/src/pages/MyDecisions.jsx`). This story is retained for historical context only.

## Description
As a logged-in user, I want to see a list of my decisions so that I can easily revisit and manage them.

## Acceptance Criteria
1.  **My Decisions**: A dashboard page listing decisions created by the user.
2.  **Filter**: Separate lists for "Open" and "Closed" decisions.
3.  **Navigation**: Clicking a decision navigates to its detail page.
4.  **Self Service**: Users can update their profile (display name, avatar if applicable).

## Technical Notes
-   Store `ownerId` in Decision documents.
-   Query decisions where `ownerId == currentUser.uid`.
-   Create `/dashboard` route.

## Implementation Plan
- [IMP-011-User-Dashboard](../plans/IMP-011-User-Dashboard.md)
