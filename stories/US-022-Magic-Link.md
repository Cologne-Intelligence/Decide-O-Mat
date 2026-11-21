# US-022: Magic Link Identity Transfer

**Role**: Anonymous User
**Goal**: I want to transfer my anonymous profile to another device so I can continue participating without starting over.

## Acceptance Criteria

### 1. Generate Magic Link
- [x] In the User Settings panel, there is a "Transfer Identity" action.
- [x] Clicking it calls the backend `generateMagicLink()` callable, which returns a Firebase **custom auth token** for the caller's entire current identity (not scoped to a single decision).
- [x] The app bundles this token together with **all of the user's locally-stored end-to-end-encryption decision keys** into one link: `.../magic#token=...&keys=...`. The keys payload lets Device B decrypt every E2E-encrypted decision Device A already knew about, not just re-authenticate.
- [x] I can copy this link.

### 2. Consume Magic Link
- [x] When I open this link on a new device (Device B), the app (`MagicHandler.jsx`) detects the `token`/`keys` hash params.
- [x] The app calls `signInWithCustomToken(token)`.
- [x] Device B is now authenticated as the same UID.
- [x] If Device B is already signed in as a **different**, non-matching account, an explicit confirmation screen warns that continuing will switch the active identity (and its vote history/attribution), requiring Cancel/Confirm before the swap proceeds.
- [x] Imported E2E keys are **merged** into local storage (not overwritten), preserving keys for decisions already known on that device.
- [x] Display name is not re-entered manually — it travels implicitly via the shared Firebase identity (same UID), and encrypted per-decision names are recoverable once the matching key is imported.

### 3. Security
- [x] The token/keys payload is carried in the URL **hash fragment**, not a query string, so it is never sent to the server or captured in referer headers/server logs.
- [x] The token and keys are stripped from the visible address bar immediately via `history.replaceState` once consumed, so they don't linger in browser history.
- [ ] The link is not short-lived or signature-restricted beyond Firebase's own custom-token expiry — no additional single-use/expiry enforcement was added.
- [ ] No explicit in-app warning is shown telling the user not to share this link publicly (unresolved from original story).

## Technical Notes
- Backend: callable `generateMagicLink()` in `functions/index.js`, calls `admin.auth().createCustomToken(uid)` for the caller's current uid.
- Frontend: `frontend/src/components/MagicLinkData.jsx` builds the `/magic#token=...&keys=...` link, serializing `EncryptionService.getAllStoredKeys()` as base64url-encoded JSON for the `keys` param.
- Route: `/magic`, handled by `frontend/src/pages/MagicHandler.jsx`.

## Implementation Plan
- [IMP-022-Magic-Link](../plans/IMP-022-Magic-Link.md)
