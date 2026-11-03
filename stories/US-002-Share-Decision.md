# User Story: Share Decision

**As a** host
**I want to** share the decision URL with others
**So that** they can participate in the decision-making process.

## Acceptance Criteria
1.  **Unique URL**: Each decision has a unique, unguessable URL (e.g., using a UUID or random string).
2.  **Copy to Clipboard**: The decision page has a "Copy Link" button that copies the current URL to the user's clipboard.
3.  **No Login**: Accessing the URL requires no authentication; possession of the link grants access.

## Technical Notes
- The URL ID should be the document ID from the database.
- Frontend: "Copy Link" copies BOTH an HTML and a plain-text clipboard representation simultaneously via the `ClipboardItem` API (`copyRichLink` in `frontend/src/utils/ClipboardUtils.js`) — HTML payload is `<a href="url">Title (by Creator)</a>`, plain-text payload is `Title (by Creator)\n<url>`, with all interpolated values HTML-escaped. Falls back to plain `navigator.clipboard.writeText(url)` if the rich clipboard write fails, and further falls back to the legacy `document.execCommand('copy')` if the Clipboard API itself is unavailable.
- Sharing is also available via a QR code (see US-037) and via a floating action button (FAB) with a speed-dial menu offering "Copy Link" and "Show QR Code".
- When end-to-end encryption is enabled, the copied URL must include the full hash fragment (`#key=...`) — the implementation copies `window.location.href`, not just a path/id.

## Implementation Plan
- [IMP-002-Share-Decision](../plans/IMP-002-Share-Decision.md)
