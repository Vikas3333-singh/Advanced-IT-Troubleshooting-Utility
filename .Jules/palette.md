## 2025-05-10 - Focus Visible in Dark Theme Case Studies
**Learning:** Single-page HTML case studies with dark design system variables often miss explicit `:focus-visible` ring indicators and dynamic `aria-label`/`title` attributes on theme toggle controls, impeding screen readers and keyboard navigation.
**Action:** Ensure `:focus-visible` outline styles with sufficient contrast and dynamic `aria-label`/`title` state updates are included in the primary `<style>` and theme script block.

## 2025-05-18 - Bypass Blocks and Main Landmark in Fixed Nav Layouts
**Learning:** Fixed header navigation bars create repetitive tab sequences on long case studies. Without an accessible skip link and semantic `<main>` landmark, keyboard users and screen readers must traverse all navigation links before reaching primary content (WCAG 2.4.1).
**Action:** Add an accessible skip link hidden offscreen until `:focus` with high-contrast styling layered above fixed nav elements (`z-index: 1000`), targeting `<main id="main-content" tabindex="-1">`.
