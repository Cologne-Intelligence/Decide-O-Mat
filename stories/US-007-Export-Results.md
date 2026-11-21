# US-007: Export Results as Image

## Description
As a user, I want to export the decision results (question, pros, cons, net score) as an image so that I can easily share it on social media or messaging apps.

## Acceptance Criteria
1.  **Export Button**: A button labeled "Export as Image" (or similar icon) is available on the Decision page.
2.  **Image Content**: The generated image must include:
    *   The decision question/title.
    *   The net score.
    *   A summary of top pros and cons (or all if feasible).
    *   The Decide-O-Mat branding/URL.
3.  **Download**: Clicking the button triggers a download of the generated image (e.g., PNG or JPEG).
4.  **Performance**: The generation should happen client-side to avoid server costs and latency.

## Technical Notes
-   Implemented using the `html-to-image` library's `toPng()` function, rendering the DOM node that contains the hero + pro/con columns.
-   Fixed solid background color `#050509` (the dark theme's bg-base token) is used regardless of the system/browser theme, with `pixelRatio: 2` for higher-resolution output.
-   Downloaded filename follows the pattern `decision-{decisionId}.png`.
-   Entirely client-side — no server/backend involvement.

## Implementation Plan
- [IMP-007-Export-Results](../plans/IMP-007-Export-Results.md)
