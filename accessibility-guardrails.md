# Accessibility Guardrails

You are building software that real people with disabilities will use. Accessibility is not a feature, a polish pass, or a legal box to tick — it is a baseline quality property of the product, on the same level as "the buttons work" and "the data isn't corrupted." Treat it that way on every change.

These guardrails are aligned with the W3C Web Accessibility Initiative (WAI):

- **Introduction to Web Accessibility** — https://www.w3.org/WAI/fundamentals/accessibility-intro/
- **Web Content Accessibility Guidelines (WCAG) 2.2** — https://www.w3.org/WAI/standards-guidelines/wcag/
- **Essential Components of Web Accessibility** — https://www.w3.org/WAI/fundamentals/components/

The conformance target for every project under these guardrails is **WCAG 2.2 Level AA**, with AAA applied where it is reasonable for the context (e.g. enhanced contrast for text-heavy reading apps). Sections are conditional — apply them when their trigger condition is met.

## 1. The Four Principles: POUR (Always Apply)

WCAG organizes every requirement under four principles. Every UI you build must satisfy all four:

- **Perceivable** — Users must be able to perceive the information presented. It cannot be invisible to all of their senses.
- **Operable** — Users must be able to operate the interface. The interface cannot require interaction a user cannot perform.
- **Understandable** — Users must be able to understand the information and the operation of the interface.
- **Robust** — Content must be robust enough to be reliably interpreted by a wide variety of user agents, including current and future assistive technologies.

If a change you make breaks any one of these for any user group, the change is not done — regardless of how it looks or how the test suite passes.

## 2. Who You Are Building For (Always Apply)

WAI identifies six categories of disability that your interfaces must accommodate. Hold each one in mind when building:

- **Visual** — blindness, low vision, color blindness. Implies: alt text, contrast, scalable text, no color-only signals, screen reader compatibility.
- **Auditory** — deafness, hard of hearing. Implies: captions, transcripts, visual alternatives to audio cues.
- **Motor / physical** — limited fine motor control, tremor, paralysis, missing limbs. Implies: full keyboard operability, large hit targets, no time-pressure interactions, no precise gestures required.
- **Cognitive** — memory, attention, learning differences, dyslexia. Implies: plain language, predictable layouts, forgiving forms, error recovery.
- **Speech** — Implies: don't require voice input as the only path.
- **Neurological** — seizures (photosensitivity), vestibular disorders. Implies: no flashing >3 Hz, respect `prefers-reduced-motion`.

Accessibility also benefits people not on this list: users on phones in bright sun, users on a flaky train connection, users with a broken arm, older users, users in their second language, and the general population on every device. Building for the edges improves the middle.

## 3. The Component Model (Always Apply)

Per the WAI components model, accessible experiences depend on **content, user agents, assistive technologies, authoring tools, evaluation tools, developers, and users** all working together. You are responsible for the parts you control:

- **Content** — your HTML, CSS, JS, images, video, copy. Governed by **WCAG 2.2**.
- **Rich UI behavior** — custom widgets, dynamic regions, interactive components. Governed by **WAI-ARIA 1.2**.
- **Authoring tools you build** — any UI in your app where users create content (rich text editors, comment boxes, page builders) must help authors produce accessible output. Governed by **ATAG 2.0**.
- **Evaluation** — the tooling and process you run to verify the above (see §13).

You do not control the user's browser, screen reader, or strategies — but you must produce content that works with the common ones. See §12 for the test matrix.

## 4. Perceivable (Always Apply)

- **Text alternatives.** Every non-decorative image, icon, chart, and media element has a text alternative that conveys the same information. Decorative images use `alt=""` (not omitted).
- **Captions and transcripts.** Pre-recorded audio has a transcript. Pre-recorded video has synchronized captions. Live audio/video has captions where feasible.
- **Don't rely on color alone.** Color is never the only way information is conveyed (status, validation, required fields, charts). Pair color with text, icon, or pattern.
- **Color contrast.** Body text ≥ 4.5:1, large text (≥18pt or ≥14pt bold) ≥ 3:1, UI components and graphical objects ≥ 3:1 against adjacent colors. Use a contrast checker. Do not eyeball it.
- **Resizable text.** Text must remain readable and functional when zoomed to 200% without horizontal scrolling at 1280px width. Use relative units (`rem`, `em`, `%`) — not fixed `px` for body text.
- **Reflow.** Content reflows to a single column at 320 CSS pixels wide without loss of information or functionality.
- **Spacing override.** Layout must not break when users override line height to 1.5×, paragraph spacing to 2×, letter spacing to 0.12×, and word spacing to 0.16×.
- **Non-text contrast.** Form input borders, focus indicators, and meaningful icons meet 3:1 against their background.
- **Images of text.** Use real text. Reserve images of text for logos and brand wordmarks.

## 5. Operable (Always Apply)

- **Full keyboard operability.** Every interactive element is reachable and operable with the keyboard alone. Tab to focus, Enter/Space to activate, arrow keys for composite widgets (menus, tabs, listboxes, sliders).
- **No keyboard traps.** Focus can always move away from any component using standard keys.
- **Visible focus indicator.** Focus is always visible, with a contrast of ≥ 3:1 against the adjacent background and a minimum thickness of 2 CSS pixels (WCAG 2.2 SC 2.4.11/2.4.13).
- **Focus order is logical.** Tab order matches the visual reading order.
- **Skip link.** Every page with repeated navigation has a "Skip to main content" link as the first focusable element.
- **Target size.** Interactive controls are at least **24×24 CSS pixels** (WCAG 2.2 SC 2.5.8 AA). Aim for **44×44** (AAA / mobile guideline) wherever feasible.
- **No time limits, or extendable ones.** Avoid timeouts. If unavoidable, warn the user with at least 20 seconds to extend, and allow at least 10× extension.
- **Pause, stop, hide.** Auto-updating, scrolling, blinking, or moving content can be paused, stopped, or hidden by the user.
- **Three flashes or below.** No content flashes more than 3 times per second.
- **Reduced motion.** Honor `prefers-reduced-motion: reduce`. Disable parallax, autoplay video, large transform animations.
- **Pointer gestures.** Any function operated by a multi-point or path-based gesture (drag, pinch, swipe) has a single-pointer alternative.
- **No drag-only interactions** (WCAG 2.2 SC 2.5.7). Sortable lists, slider thumbs, etc. need a non-drag alternative such as up/down buttons or arrow keys.
- **Click cancellation.** Down-events do not commit the action — the action fires on up-event, and the user can move the pointer away to abort.

## 6. Understandable (Always Apply)

- **Language declared.** `<html lang="en">` (or correct locale). Sub-sections in another language use `lang` on that element.
- **Consistent navigation and identification.** Repeated components (nav, search, footer, account menu) appear in the same relative order and are labeled consistently across pages.
- **Predictable behavior.** Focus or input does not trigger an unexpected change of context (no auto-submit on focus, no redirect on input). Changes of context happen on explicit activation.
- **Plain language.** Reading level should be appropriate for the audience. Avoid jargon; expand acronyms on first use.
- **Form labels.** Every input has a programmatically associated `<label>`. Placeholder text is never a substitute for a label.
- **Required and optional state.** Required fields are marked in text (not color or asterisk alone) and via `aria-required` or the `required` attribute.
- **Inline error messages.** Validation errors are announced to assistive tech (`aria-live="polite"` or `aria-describedby` on the input), point to the failing field, and explain how to fix it.
- **Error prevention.** For legal, financial, or data-modifying submissions: changes are reversible, checked for errors, or confirmed before commit. WCAG 2.2 SC 3.3.7 also forbids re-entering previously provided info in the same session unless essential.
- **Help available** (WCAG 2.2 SC 3.3.5/3.3.6 AA). When help (contact, FAQ link, self-help) is provided across multiple pages, it appears in the same relative location.
- **Accessible authentication** (WCAG 2.2 SC 3.3.8 AA). Do not require a cognitive function test (remembering passwords, transcribing, solving puzzles, identifying images) to log in unless an alternative is provided. Support password managers, passkeys, magic links.

## 7. Robust (Always Apply)

- **Valid, semantic HTML.** Use the correct element for the job: `<button>` for actions, `<a href>` for navigation, `<nav>`, `<main>`, `<header>`, `<footer>`, `<h1>`–`<h6>` in order, `<ul>`/`<ol>` for lists, `<table>` for tabular data with `<th scope>`.
- **One `<h1>` per page**, in document order. Heading levels do not skip.
- **Name, role, value.** Every UI control exposes its name (label), role (what kind of control), and current state to assistive technology. Custom widgets must provide these via ARIA when no native equivalent exists.
- **Status messages.** Non-modal status updates (saved, error, item added) are announced via `role="status"`, `role="alert"`, or `aria-live` regions — without moving focus.
- **No ARIA when HTML will do.** First rule of ARIA: don't use ARIA. Use the native HTML element. Only reach for ARIA when no native control fits.
- **ARIA used correctly.** Required ARIA attributes are present, values are valid, roles are not invented, and `aria-hidden="true"` is never on a focusable element.

## 8. If the App Has Forms

- Label every input. Group related inputs with `<fieldset>`/`<legend>` (radio groups, checkbox groups, address blocks).
- Autocomplete tokens (`autocomplete="email"`, `"new-password"`, `"street-address"`, etc.) on personal-data fields. Required by WCAG SC 1.3.5.
- Use the right input type (`type="email"`, `type="tel"`, `type="number"`, `type="date"`) so mobile keyboards and assistive tech adapt.
- Errors: surface a summary at the top of the form with anchor links to each failing field, AND an inline message at each field. Move focus to the summary on submit failure.
- Don't disable the submit button on invalid state without explanation — disabled buttons are not announced clearly. Prefer always-enabled with on-submit validation, or pair `disabled` with a visible `aria-describedby` reason.
- Preserve user input on error. Never wipe a form because validation failed.

## 9. If the App Has Audio or Video

- Pre-recorded video: synchronized captions (Level A) and audio description for visual-only information (Level AA).
- Pre-recorded audio: full text transcript.
- Live audio/video: captions for live audio (Level AA where feasible).
- Auto-play is off by default. If audio plays for more than 3 seconds, provide a mechanism to pause or stop it.
- Media players themselves must be keyboard-accessible and screen-reader-labeled. Test the player you ship, not just the media.

## 10. If the App Has Custom or Dynamic UI (SPA, Modals, Menus, Tabs, Trees)

- Follow the **ARIA Authoring Practices Guide** (https://www.w3.org/WAI/ARIA/apg/) patterns for any custom widget — modal dialog, combobox, tabs, menu, tree, listbox, slider, disclosure. Do not invent your own keyboard model.
- **Modals/dialogs:** trap focus inside while open, return focus to the trigger on close, close on `Escape`, mark with `role="dialog"` and `aria-modal="true"`, label with `aria-labelledby`.
- **Route changes in SPAs:** on navigation, move focus to the new page's `<h1>` or main landmark, and announce the new page title via `document.title` plus an `aria-live` region. Without this, screen reader users have no idea the page changed.
- **Live regions:** use `aria-live="polite"` for non-urgent updates, `aria-live="assertive"` (or `role="alert"`) sparingly for genuine errors. Do not toggle `aria-live` dynamically — declare the region empty in initial markup and update its contents.
- **Toasts and notifications:** announced via live region, dismissible by keyboard, do not steal focus, persist long enough to be read (4–6 seconds minimum, longer for longer messages, or sticky with manual dismiss).
- **Loading states:** announced (`aria-busy="true"` on the region, or a polite live region message). Do not leave screen reader users in silence.

## 11. If the App Has Data Tables, Charts, or Visualizations

- Use real `<table>` markup with `<caption>`, `<thead>`, `<th scope="col">` / `scope="row"`. Don't fake tables with `<div>`s.
- Sortable column headers expose sort state via `aria-sort`.
- Charts have a text alternative: a summary `aria-label`, a visually-hidden data table, or a "View as table" toggle. A screenshot with no description is not accessible.
- Color in charts is paired with pattern, label, or shape so color-blind users can distinguish series.

## 12. If the App Will Be Tested with Assistive Technology

The realistic test matrix for an AA project, in priority order:

1. **Keyboard only** — no mouse, no trackpad. Tab, Shift+Tab, Enter, Space, arrow keys, Escape.
2. **macOS VoiceOver + Safari** — Cmd+F5 to toggle.
3. **Windows NVDA + Firefox or Chrome** — free, the most common screen reader for testing.
4. **iOS VoiceOver on Safari** — for any responsive/mobile flow.
5. **TalkBack on Chrome (Android)** — for Android-targeted flows.
6. **200% browser zoom** in Chrome or Firefox.
7. **Forced colors / high contrast mode** in Windows or via DevTools emulation.
8. **`prefers-reduced-motion`** via DevTools emulation.

If you cannot run these in the environment you're working in, say so explicitly in your task report and provide a manual test checklist for the user (per `qa-standards.md` Tier 3).

## 13. Tooling and Verification

- **Automated checks** are necessary but not sufficient. They catch ~30–40% of WCAG issues. Required tools:
  - **axe-core** (via `@axe-core/react`, `@axe-core/playwright`, or the **axe DevTools** browser extension) on every page in CI.
  - **Lighthouse** accessibility audit (Chrome DevTools or CI). Score ≥ 95 is the floor; the floor is not the goal.
  - **eslint-plugin-jsx-a11y** (React) or framework equivalent in lint config. No suppressions without a comment explaining why.
- **Manual checks** that no tool catches and you must do every time:
  - Tab through the entire feature. Focus is always visible, order is logical, nothing is skipped or trapped.
  - Use the feature with a screen reader for the golden path.
  - Zoom to 200%. Reflow to 320px. Toggle reduced motion. Toggle forced colors.
  - Read every error message aloud. Does it tell a non-technical user how to fix the problem?
- **Authored content** (CMS, rich text editors, comment fields): if your app lets users create content, the editor must support and encourage accessible output (heading structure, alt text prompts on image upload, sufficient contrast in theme picker). This is the ATAG layer.
- **Third-party components.** Any imported component (date picker, modal, dropdown, charting library, embed) is your responsibility. Test it. If it fails, replace it or wrap it. "It's a library bug" is not an exemption.

## 14. Process — When Accessibility Engages in the Build

Accessibility is cheapest at design time and most expensive after launch. Engage it at every stage:

- **Design / spec.** State the keyboard interaction model, the screen-reader announcement, the focus management plan, and the responsive/zoom behavior alongside the visual design. If the spec doesn't cover these, the spec is incomplete.
- **Build.** Use semantic HTML first. Reach for ARIA only when semantics run out. Run the linter and axe in dev. Test with the keyboard as you build, not at the end.
- **Review.** Every PR touching UI has an accessibility check in the review (see §15). Reviewers run keyboard + axe at minimum.
- **Release.** Run the full §12 manual sweep on the release candidate for any feature that touches navigation, forms, modals, custom widgets, or media.
- **Post-release.** Track accessibility issues in the same backlog as functional bugs, with the same severity scale. A blocked screen reader user is a P1, not a "polish" ticket.

## 15. Before Declaring an Accessibility-Touching Change Done (Always Apply)

Walk through this checklist for any change to UI, content, or data presentation:

- [ ] **Semantics:** correct native element used; ARIA only where native doesn't fit; no role inventions.
- [ ] **Keyboard:** every interactive element reachable, operable, with logical Tab order and visible focus.
- [ ] **No keyboard traps**, including in modals and embedded widgets.
- [ ] **Screen reader smoke test** of the golden path with VoiceOver or NVDA.
- [ ] **Names, roles, states** exposed for every control (verified in the browser's accessibility tree).
- [ ] **Color contrast** verified with a contrast checker for text and meaningful UI.
- [ ] **No color-only signaling.**
- [ ] **Form labels, autocomplete, error summary + inline errors** present.
- [ ] **Reflow at 320px** and **200% zoom** without loss of content or function.
- [ ] **`prefers-reduced-motion` honored**; no >3 Hz flashing.
- [ ] **Live regions** announce dynamic updates; toasts and route changes don't go silent.
- [ ] **axe DevTools** (or CI equivalent) returns no violations on the changed pages.
- [ ] **Lighthouse accessibility score ≥ 95** on the changed pages.
- [ ] **Linter (`jsx-a11y` or equivalent)** passes with no new suppressions.
- [ ] **WCAG 2.2 AA criteria** specifically affected by this change have been re-verified, not just assumed.

## 16. Honesty Rule (Always Apply)

- If you have not actually tested with a screen reader, **say so**. Do not claim a change is "accessible" when only axe was run.
- If a component is partially accessible (works with keyboard but not screen reader, or vice versa), **document the gap** and file it.
- If a third-party dependency blocks AA conformance and replacing it is out of scope, **flag it explicitly** in the task report and in `REGRESSIONS.md`.
- Never disable an accessibility lint rule, axe rule, or test to make CI green. If it's a false positive, leave a comment explaining why and link the rule's documentation. If it's a real issue, fix it.

## 17. When in Doubt

- If a WCAG success criterion is ambiguous in your context, consult the **Understanding WCAG 2.2** document for that criterion (https://www.w3.org/WAI/WCAG22/Understanding/). Do not guess.
- If a custom widget pattern isn't in the **ARIA Authoring Practices Guide**, the widget is probably too custom. Use a more standard pattern.
- If you are about to write `role="button"` on a `<div>`, stop. Use `<button>`.
- If a stakeholder asks you to skip accessibility "for now," surface the cost: retrofitting is 5–10× more expensive than building it in, and excludes real users in the meantime. Get the trade-off in writing before agreeing.

## 18. The Prime Directive

Accessibility is not about compliance. It is about whether the people you are building for can actually use the thing. **A feature that excludes a category of users is not done — it is broken in a way the test suite doesn't catch.** Build it so it works for everyone, the first time.
