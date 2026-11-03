# US-010: Firebase Auth Integration

## Description
As a user, I want to be able to sign in so that I can securely manage my identity and access my decisions across devices.

## Acceptance Criteria
1.  **Sign In/Register**: Users can sign in using Google (OAuth) or Email/Password.
2.  **Sign Out**: Users can sign out.
3.  **Session Management**: User session persists across reloads.
4.  **Anonymous Upgrade**: If a user was anonymous, their local data (if possible) or future actions are linked to the new account.

## Technical Notes
-   Enable Authentication in Firebase Console (Google, Email/Password).
-   Implement `AuthContext` in React to manage user state.
-   Update UI to show Login/Logout buttons.
-   The app automatically calls `signInAnonymously()` on first load whenever there is no existing Firebase session — there is no explicit "continue as guest" step; anonymous auth is the default, always-on identity layer that Google/Email sign-in upgrades from (via `linkWithPopup`/`linkWithCredential` when the user opts to link rather than create a fresh account — see USER_CONCEPTS.md).

## Implementation Plan
- [IMP-010-Auth-Integration](../plans/IMP-010-Auth-Integration.md)
