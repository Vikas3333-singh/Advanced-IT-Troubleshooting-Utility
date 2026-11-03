## 2026-11-03 - Skip Navigation Landmark in Fixed-Header Single-Page Case Studies
**Learning:** Single-page technical case studies featuring fixed navigation bars trap keyboard and screen-reader users into tabbing through all brand links, section anchors, and toggles repeatedly before reaching the substantive content. Furthermore, missing semantic `<main id="main" tabindex="-1">` elements violates landmark region guidelines and prevents programmatic focus shifts when bypassing headers.
**Action:** Always provide an off-screen `.skip-link` as the first interactive element targeting `<main id="main" tabindex="-1">` that animates visibly into view on `:focus`, paired with `prefers-reduced-motion` fallbacks.

## 2025-05-10 - Focus Visible in Dark Theme Case Studies
**Learning:** Single-page HTML case studies with dark design system variables often miss explicit `:focus-visible` ring indicators and dynamic `aria-label`/`title` attributes on theme toggle controls, impeding screen readers and keyboard navigation.
**Action:** Ensure `:focus-visible` outline styles with sufficient contrast and dynamic `aria-label`/`title` state updates are included in the primary `<style>` and theme script block.
