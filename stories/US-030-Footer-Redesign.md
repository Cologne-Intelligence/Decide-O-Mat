# User Story: Footer Redesign

**As a** user
**I want** a footer with legal links and application branding
**So that** I can access legal information and know which version of the app I'm using.

## Acceptance Criteria
1. **Dark Background**: Full-width dark footer consistent with overall theme.
2. **Legal Links** (left-aligned): "Terms" (Figma: "AGB"), "Privacy" (Figma: "Datenschutz"), "Legal Notice" (Figma: "Impressum").
3. **Branding** (right-aligned): "Decide-O-Mat" text and version number (e.g., "v1.6.0").
4. **Link Targets**: Legal links navigate to their respective pages (placeholder routes until US-016 is implemented).
5. **Encryption Indicator**: Resolved — no encryption-status indicator was added to the footer; this remains an open/unimplemented idea.

## Design Gaps
- **Encryption status indicator**: Resolved — not implemented. The shipped footer contains only legal links, a "Cookie Settings" button, and branding/version; no E2E encryption status is shown anywhere in `Footer.jsx`.

## Technical Notes
- Current component: `frontend/src/components/Footer.jsx`
- Legal pages (US-016) are not yet implemented — links can point to placeholder routes.
- Version is injected via Vite's `__APP_VERSION__` define.
- The "Cookie Settings" button dispatches a global `openCookiePreferences()` event (`frontend/src/services/ConsentService.js`) that reopens the `CookieConsent` component's customize panel after the user's initial consent decision — see US-018 (GDPR Compliance).

## Figma Reference
- Node ID: `555:1457` (Footer component)

## Implementation Plan
- [IMP-030-Footer-Redesign](../plans/IMP-030-Footer-Redesign.md)
