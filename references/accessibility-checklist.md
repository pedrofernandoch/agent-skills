# Accessibility Checklist

Quick reference for WCAG 2.2 AA compliance. Use alongside the `frontend-ui-engineering` skill.

Accessibility is a first-class implementation requirement, not an optional feature or a separate
"accessibility mode". Whenever you add or modify a component, flow, interaction, or visual state,
evaluate its accessibility implications before considering the work complete.

## Table of Contents

- [Core Principles](#core-principles)
- [Essential Checks](#essential-checks)
- [Motion and System Preferences](#motion-and-system-preferences)
- [Cognitive Accessibility](#cognitive-accessibility)
- [Motor, Touch and Gestures](#motor-touch-and-gestures)
- [Responsive and Reflow](#responsive-and-reflow)
- [Media, Charts and Audio](#media-charts-and-audio)
- [Authentication](#authentication)
- [Interactive Components](#interactive-components)
- [Assistive Technology Compatibility](#assistive-technology-compatibility)
- [Common HTML Patterns](#common-html-patterns)
- [Testing Tools](#testing-tools)
- [Quick Reference: ARIA Live Regions](#quick-reference-aria-live-regions)
- [Common Anti-Patterns](#common-anti-patterns)

## Core Principles

1. Prefer native HTML semantics and controls. Do not build a custom control when an equivalent
   native one exists.
2. Never use `<div>` or `<span>` as an interactive control when `<button>`, `<a>`, `<input>`,
   `<select>` fits.
3. First rule of ARIA: do not use ARIA to compensate for incorrect HTML when native semantics solve
   the problem. Use ARIA only when necessary.
4. Never remove or hide focus indicators without an equally effective replacement.
5. Never communicate error, status, selection, or meaning through color alone.
6. Never make hover the only way to reach important information or functionality.
7. Never make drag-and-drop or a complex gesture the only interaction method.
8. Never use placeholder text as the only label.
9. When a custom component is unavoidable, follow the established WAI-ARIA Authoring Practices
   pattern for it.
10. Every interactive element has an accessible name plus the correct role and state.
11. Focus moves predictably and is never unexpectedly lost or trapped.
12. Dynamic UI changes are communicated to assistive technologies.
13. Preserve user-entered data, especially after validation errors.
14. Avoid unnecessary animation, flashing, time limits, and automatic content changes.
15. Accessibility works in the default UI. Do not hide fundamental accessibility behind an optional
    "Accessibility Mode" toggle.

## Essential Checks

### Keyboard Navigation
- [ ] All interactive elements focusable via Tab key
- [ ] Focus order follows visual/logical order
- [ ] Focus is visible (outline/ring on focused elements)
- [ ] Focused element is not obscured by sticky headers, footers, or overlays (WCAG 2.2 SC 2.4.11)
- [ ] Custom widgets have keyboard support (Enter to activate, Escape to close)
- [ ] No keyboard traps (user can always Tab away from a component)
- [ ] Focus is never silently lost (for example, after removing the focused node from the DOM)
- [ ] Skip-to-content link at top of page - visible (at least) on keyboard focus
- [ ] Modals trap focus while open, return focus on close

### Screen Readers
- [ ] All images have `alt` text (or `alt=""` for decorative images)
- [ ] All form inputs have associated labels (`<label>` or `aria-label`)
- [ ] Buttons and links have descriptive text (not "Click here")
- [ ] Icon-only buttons have `aria-label`
- [ ] Page has one `<h1>` and headings don't skip levels
- [ ] Page uses landmarks (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`); multiple landmarks
      of the same type are distinguished with `aria-label`
- [ ] Current page/step marked with `aria-current` in navigation, breadcrumbs, and steppers
- [ ] Dynamic content changes announced (`aria-live` regions)
- [ ] Loading, progress, toast, notification, success, warning, and error states are perceivable
      without relying on visual or auditory feedback alone
- [ ] Tables have `<th>` headers with scope

### Visual
- [ ] Text contrast >= 4.5:1 (normal text) or >= 3:1 (large text, 18px+)
- [ ] UI components contrast >= 3:1 against background
- [ ] Color is not the only way to convey information
- [ ] Interface stays usable for the common color-vision deficiencies (pair color with text, icon,
      shape, or pattern)
- [ ] Text resizable to 200% without breaking layout
- [ ] Layout survives browser/system zoom and larger default font sizes
- [ ] Works in Windows High Contrast / `forced-colors` mode (no `background-image`-only icons, no
      hard-coded colors that drop out)
- [ ] `prefers-color-scheme` and `prefers-contrast` respected where themes exist
- [ ] No content that flashes more than 3 times per second

### Forms
- [ ] Every input has a visible label
- [ ] Required fields indicated (not by color alone)
- [ ] Related controls grouped with `<fieldset>` and `<legend>` (radio groups, address blocks)
- [ ] Appropriate `type` and `inputmode` used (`email`, `tel`, `url`, `numeric`)
- [ ] Error messages specific and associated with the field (`aria-describedby`, `aria-invalid`)
- [ ] Error state visible by more than color (icon, text, border)
- [ ] Form submission errors summarized and focusable
- [ ] User input preserved after a failed validation or submission
- [ ] Submission outcome announced (`role="status"` for success, `role="alert"` for failure)
- [ ] Known fields use autocomplete (for example `type="email" autocomplete="email"`)

### Content
- [ ] Language declared (`<html lang="en">`)
- [ ] Page has a descriptive `<title>`
- [ ] Links distinguish from surrounding text (not by color alone)
- [ ] Touch targets >= 44x44px on mobile (WCAG 2.2 SC 2.5.8 requires >= 24x24px minimum)
- [ ] Meaningful empty states (not blank screens)

## Motion and System Preferences

- [ ] `prefers-reduced-motion` respected: transitions, parallax, auto-scrolling, and decorative
      animation are reduced or removed
- [ ] Significant motion lasting more than 5 seconds can be paused, stopped, or hidden
- [ ] No autoplaying video/animation without controls
- [ ] Non-animated alternative exists where animation carries meaning
- [ ] No flashing above the 3-per-second threshold
- [ ] `prefers-contrast`, `forced-colors`, and `prefers-color-scheme` handled rather than ignored

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## Cognitive Accessibility

- [ ] Clear, concise language; jargon explained
- [ ] Predictable layouts and consistent navigation across pages
- [ ] Related controls and information grouped logically
- [ ] Progressive disclosure used instead of overwhelming screens
- [ ] Contextual help available where input is non-obvious, and placed consistently across pages
      (WCAG 2.2 SC 3.2.6 Consistent Help)
- [ ] Information already provided in a flow is not requested again, or is auto-filled/selectable
      (WCAG 2.2 SC 3.3.7 Redundant Entry)
- [ ] Instructions and error messages are understandable and say how to fix the problem
- [ ] Destructive actions require confirmation
- [ ] Undo / recovery available; autosave where appropriate
- [ ] No unnecessary time limits; where one exists it can be extended or turned off
- [ ] No unexpected context changes on focus or input

## Motor, Touch and Gestures

- [ ] Everything reachable and operable by keyboard alone
- [ ] Adequate spacing between adjacent targets
- [ ] No action requires fine pointer precision
- [ ] Drag-and-drop has a single-pointer alternative (buttons, menu action, keyboard move)
      (WCAG 2.2 SC 2.5.7 Dragging Movements)
- [ ] Swipe, multi-touch, long-press, and path-based gestures have a simple-tap alternative
- [ ] Works with mouse, touch, keyboard, switch control, and other alternative input devices
- [ ] Actions trigger on pointer-up, so a mis-press can be aborted

## Responsive and Reflow

- [ ] Content reflows at 320 CSS px wide with no two-dimensional scrolling (WCAG SC 1.4.10)
- [ ] Usable at 400% zoom
- [ ] Both portrait and landscape orientations supported (do not lock orientation)
- [ ] Accessibility behavior holds on desktop, tablet, and mobile, not just the primary breakpoint
- [ ] Text spacing overrides (line height, letter/word spacing) don't clip content

## Media, Charts and Audio

### Multimedia
- [ ] Captions/subtitles for video with audio
- [ ] Transcript available for audio and video
- [ ] Audio description where visual information isn't in the audio track
- [ ] Media controls are keyboard operable and have accessible names
- [ ] Play/pause and volume controls exposed; no autoplay with sound

### Images and Graphics
- [ ] Informative images have meaningful `alt`
- [ ] Decorative images have `alt=""`
- [ ] Meaningful icons have accessible names; decorative icons are hidden (`aria-hidden="true"`)
- [ ] Complex graphics (diagrams, infographics) have a longer text alternative nearby

### Charts and Dashboards
- [ ] Meaning does not depend on color or visual position alone (add labels, patterns, direct
      annotation)
- [ ] Series labels, values, and legend text available to screen readers
- [ ] Chart has an accessible name and short description of what it shows
- [ ] Data points reachable by keyboard where the chart is interactive; tooltips are not hover-only
- [ ] Accessible table or text summary provided as an alternative to complex visualizations

### Audio Feedback
- [ ] Sound is never the only signal; important audio cues have a visual and/or haptic equivalent

## Authentication

- [ ] Password managers work (no blocked paste, no split-field inputs, correct `autocomplete`
      tokens: `username`, `current-password`, `new-password`, `one-time-code`)
- [ ] Copy/paste allowed in all credential fields
- [ ] Passkeys, biometrics, magic links, or SSO/OAuth supported where applicable
- [ ] No cognitive function test as the only authentication step - no puzzles, no memorization, no
      transcription of a code the user must retype from another medium (WCAG 2.2 SC 3.3.8)
- [ ] Errors state what failed without leaking account information

## Interactive Components

Custom widgets must expose correct semantics, states, keyboard behavior, focus management, and an
accessible name. Prefer the native element; when building a custom one, follow the matching
[WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/patterns/) pattern.

| Component | Native first | Key requirements |
|---|---|---|
| Button / link | `<button>`, `<a href>` | Correct element for action vs. navigation; accessible name |
| Checkbox / radio | `<input type="checkbox\|radio">` | Label association; group in `<fieldset>` |
| Select / combobox | `<select>` | Custom combobox needs `role="combobox"`, `aria-expanded`, `aria-controls`, `aria-activedescendant`, arrow-key navigation |
| Dialog / modal | `<dialog>` | Initial focus set, focus trapped while open, Escape closes, close control exposed, `aria-labelledby`/`aria-describedby`, focus restored to trigger |
| Accordion | `<details>`/`<summary>` | Header is a `<button>` with `aria-expanded` + `aria-controls` |
| Tabs | - | `role="tablist"/"tab"/"tabpanel"`, arrow-key navigation, roving `tabindex`, `aria-selected` |
| Menu | - | `role="menu"/"menuitem"`, arrow keys, Escape closes and restores focus |
| Tooltip | - | Reachable on focus, not hover-only, Escape dismisses, `aria-describedby` |
| Carousel | - | Pause control, keyboard navigation, slide position announced |
| Slider | `<input type="range">` | `aria-valuenow/min/max/text`, arrow/Home/End keys |
| Date picker | `<input type="date">` | Manual text entry always available; grid keyboard navigation |
| Toast / notification | - | Live region, sufficient dwell time, dismissible, focus not stolen |

## Assistive Technology Compatibility

Verify the UI holds up beyond the screen reader:

- [ ] Voice control and dictation: visible control labels match their accessible names, so "click
      Save" works
- [ ] Screen magnifiers: focused and active content stays within view
- [ ] Braille displays: text alternatives are concise and meaningful out of visual context
- [ ] Switch control and eye tracking: sequential navigation reaches everything; no timing pressure
- [ ] Platform accessibility APIs receive correct role, name, state via standard semantics

## Common HTML Patterns

### Buttons vs. Links

```html
<!-- Use <button> for actions -->
<button onClick={handleDelete}>Delete Task</button>

<!-- Use <a> for navigation -->
<a href="/tasks/123">View Task</a>

<!-- NEVER use div/span as buttons -->
<div onClick={handleDelete}>Delete</div>  <!-- BAD -->
```

### Form Labels

```html
<!-- Explicit label association -->
<label htmlFor="email">Email address</label>
<input id="email" type="email" required />

<!-- Implicit wrapping -->
<label>
  Email address
  <input type="email" required />
</label>

<!-- Hidden label (visible label preferred) -->
<input type="search" aria-label="Search tasks" />
```

### Grouped Fields and Errors

```html
<fieldset>
  <legend>Notification method</legend>
  <label><input type="radio" name="notify" value="email" /> Email</label>
  <label><input type="radio" name="notify" value="sms" /> SMS</label>
</fieldset>

<label htmlFor="title">Title</label>
<input id="title" aria-invalid="true" aria-describedby="title-error" />
<p id="title-error">Title is required. Enter a name up to 60 characters.</p>
```

### ARIA Roles

```html
<!-- Navigation -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer links">...</nav>

<!-- Current page -->
<a href="/tasks" aria-current="page">Tasks</a>

<!-- Status messages -->
<div role="status" aria-live="polite">Task saved</div>

<!-- Alert messages -->
<div role="alert">Error: Title is required</div>

<!-- Modal dialogs -->
<dialog aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</dialog>

<!-- Loading states -->
<div aria-busy="true" aria-label="Loading tasks">
  <Spinner />
</div>
```

### Accessible Lists

```html
<ul role="list" aria-label="Tasks">
  <li>
    <input type="checkbox" id="task-1" aria-label="Complete: Buy groceries" />
    <label htmlFor="task-1">Buy groceries</label>
  </li>
</ul>
```

### Chart with a Text Alternative

```html
<figure>
  <div role="img" aria-labelledby="chart-title" aria-describedby="chart-summary">
    <!-- SVG chart -->
  </div>
  <figcaption id="chart-title">Weekly completed tasks</figcaption>
  <p id="chart-summary">Completed tasks rose from 12 on Monday to 31 on Friday.</p>
  <details>
    <summary>View data as table</summary>
    <table>...</table>
  </details>
</figure>
```

## Testing Tools

```bash
# Automated audit
npx axe-core          # Programmatic accessibility testing
npx pa11y             # CLI accessibility checker

# In browser
# Chrome DevTools → Lighthouse → Accessibility
# Chrome DevTools → Elements → Accessibility tree
# Chrome DevTools → Rendering → Emulate prefers-reduced-motion / prefers-contrast / forced-colors

# Screen reader testing
# macOS: VoiceOver (Cmd + F5)
# Windows: NVDA (free) or JAWS
# Linux: Orca
```

Automated checks catch roughly a third of real issues and are never sufficient on their own. Before
calling UI work done, manually verify:

- keyboard-only navigation, with visible and logical focus
- screen-reader semantics and announcements
- color contrast and color-blind interpretation
- zoom, text resizing, and reflow
- reduced-motion and forced-colors behavior
- form labels, validation, and preserved input
- dialog focus management
- touch target size and alternatives to gestures
- dynamic status and error feedback

## Quick Reference: ARIA Live Regions

| Value | Behavior | Use For |
|-------|----------|---------|
| `aria-live="polite"` | Announced at next pause | Status updates, saved confirmations |
| `aria-live="assertive"` | Announced immediately | Errors, time-sensitive alerts |
| `role="status"` | Same as `polite` | Status messages |
| `role="alert"` | Same as `assertive` | Error messages |

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| `div` as button | Not focusable, no keyboard support | Use `<button>` |
| Missing `alt` text | Images invisible to screen readers | Add descriptive `alt` |
| Color-only states | Invisible to color-blind users | Add icons, text, or patterns |
| Autoplaying media | Disorienting, can't be stopped | Add controls, don't autoplay |
| Custom dropdown with no ARIA | Unusable by keyboard/screen reader | Use native `<select>` or proper ARIA listbox |
| Removing focus outlines | Users can't see where they are | Style outlines, don't remove them |
| Empty links/buttons | "Link" announced with no description | Add text or `aria-label` |
| `tabindex > 0` | Breaks natural tab order | Use `tabindex="0"` or `-1` only |
| Placeholder as the only label | Disappears on input; often fails contrast | Add a real `<label>` |
| Hover-only content | Unreachable by keyboard, touch, magnifier | Also reveal on focus; make it dismissible |
| Drag-only interaction | Excludes motor-impaired and keyboard users | Add a button/menu equivalent |
| ARIA patching bad HTML | Duplicated or conflicting semantics | Use the native element instead |
| Separate "Accessibility Mode" | Default experience stays inaccessible | Build accessibility into the default UI |
| Ignoring `prefers-reduced-motion` | Triggers nausea and vestibular symptoms | Gate animation behind the media query |
| Blocked paste in password fields | Breaks password managers | Allow paste and set `autocomplete` |
| Focus lost after delete/close | Screen reader lands at page top | Move focus to a sensible neighbor |
