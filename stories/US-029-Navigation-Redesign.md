# User Story: Navigation Bar Redesign

**As a** user
**I want** a clear, dark-themed navigation bar with structured links and user controls
**So that** I can easily navigate between creating decisions, viewing my activities, and managing my account.

## Acceptance Criteria
1. **Dark Background**: Full-width dark navigation bar, seamless with page background.
2. **Logo**: "Decide-O-Mat" in bold display font (left-aligned).
3. **Navigation Links**: "Decision" (Figma: "Entscheidung") for new decision/home and "Activities" (Figma: "Aktivitaeten") for archive/my decisions.
4. **Dark Mode Toggle**: Resolved — not implemented. The app is dark-only (per US-028's resolution); no light/dark toggle exists or is planned.
5. **User Section** (right-aligned):
   - User avatar (emoji or photo)
   - Display name
   - Edit icon (pencil) to edit name
   - "Log in" link/button for unauthenticated users
6. **Active State**: Current page link is visually highlighted.
7. **Responsive**: Resolved — superseded by and fully specified in US-038 (Mobile-Friendly Navigation & Responsive Experience), which implements a hamburger/drawer nav pattern at ≤768px.

## Technical Notes
- Current component: `frontend/src/components/Header.jsx`
- Currently includes `UserSettings` component — needs integration with new layout.
- Navigation links map to existing routes: `/` (home) and `/my-decisions` (archive).
- `Header.jsx` also includes a manual EN/DE language-toggle button (independent of this redesign's original scope, but present in the shipped header) and parses the current decision id + E2EE key out of the URL — even on routes outside `/d/:id` — purely to pass them into the `UserSettings` panel for participant-name updates.

## Figma Reference
- Node ID: `552:1424` (Navigation bar component)

## Implementation Plan
- [IMP-029-Navigation-Redesign](../plans/IMP-029-Navigation-Redesign.md)
