# UX Guardrails

The slowest, most dangerous failure mode in vibe-coded apps is not bugs — it's UX drift. Every update adds a little more clutter, a little more inconsistency, a little more cognitive load, until the app is technically working but nobody wants to use it. These rules exist to prevent that. Apply them to every UI change.

## 1. The Complexity Budget (Always Apply)

Every screen has a budget. When you exceed it, redesign — don't just add more.

- **One primary action per screen.** If there are two equally important things to do, there should probably be two screens.
- **Maximum ~7 controls per screen** (buttons, inputs, links, toggles). More than that, group them, hide them behind progressive disclosure, or split the screen.
- **Maximum 3 levels of visual hierarchy.** Heading, subheading, body. If you need more, the content needs restructuring, not more font sizes.
- **If a screen needs a tutorial to understand, the screen is wrong.** Fix the screen, don't write the tutorial.

## 2. Consistency Beats Cleverness (Always Apply)

The same thing should look and behave the same way everywhere.

- **One spacing scale.** Pick a scale (4/8/16/24/32/48px is fine) and use only those values. No `margin: 13px`.
- **One color palette.** Define semantic tokens (primary, secondary, success, warning, danger, neutral) and use only those. No one-off hex codes.
- **One type scale.** Define heading sizes once, use them everywhere. No inline `font-size`.
- **One button hierarchy.** Primary, secondary, tertiary, destructive. Same styling across the app. The "save" button on screen A should look identical to the "save" button on screen B.
- **One iconography style.** Don't mix outline and filled, don't mix icon libraries.
- **One radius, one shadow style, one transition timing.** Pick once, reuse everywhere.

If the project has a design system or token file, use it. If it doesn't, create one before adding new visual styles.

## 3. The Five States Rule (Always Apply to Any Component That Loads or Displays Data)

Every data-displaying component must handle all five states:

1. **Empty** — no data yet. Friendly message, clear next action ("Create your first X").
2. **Loading** — data is being fetched. Skeleton, spinner, or placeholder. Never a blank screen.
3. **Partial** — some data loaded, more loading. Show what you have, indicate the rest is coming.
4. **Loaded** — the happy path.
5. **Error** — something failed. Clear message, suggested action, way to retry. Never a raw stack trace.

If you build a list, table, or detail view without all five states, you're not done.

## 4. Destructive Action Protection (Always Apply)

- **Confirm before destroying.** Delete, archive, reset, "discard changes," "leave page with unsaved edits" — all require explicit confirmation.
- **Confirmation must name the thing.** "Delete *Q3 Strategy.doc*?" not "Are you sure?"
- **Destructive buttons are visually distinct.** Red, or clearly separated from safe actions.
- **Undo > confirm** when feasible. A 10-second undo toast is better UX than a confirmation dialog.
- **Never make destructive the default.** The Enter key should never delete anything.

## 5. Input & Form Standards (Always Apply to Any Form)

- **Labels above inputs**, not as placeholders. Placeholders disappear and hurt accessibility.
- **Required fields are marked**, optional fields aren't.
- **Validation happens on blur, not on every keystroke.** Inline errors appear next to the field, in plain language, telling the user how to fix the problem.
- **Submit buttons are disabled until the form is valid**, OR the button is always enabled and clicking surfaces all errors at once. Pick one, be consistent.
- **Form submission shows a loading state.** The button should not be clickable twice.
- **Success state is unmistakable.** Toast, redirect, inline confirmation. Never silent success.
- **Preserve user input on error.** Never wipe a form because validation failed.
- **Autofocus the first field** on forms the user clearly came to fill out.

## 6. Navigation & Wayfinding (Always Apply)

- **The user always knows where they are.** Page title, breadcrumb, or highlighted nav item.
- **The user can always go back.** Back button works. Browser back button works. There is always a way out of any screen.
- **No dead ends.** Every screen has at least one clear next action or way home.
- **Active state is visible.** The current nav item, the current tab, the current step in a flow — all visually distinct.
- **Links look like links.** Buttons look like buttons. Never make a link styled as a button do something destructive, and never make a button styled as a link do navigation.

## 7. Feedback & Responsiveness (Always Apply)

- **Every user action gets feedback within 100ms.** Even if the work takes longer, *acknowledgment* must be instant. Button press = visual change immediately.
- **Long operations show progress.** Anything over 1 second needs a spinner. Anything over 5 seconds needs a progress indicator or "this may take a moment" message.
- **Background operations are visible.** Auto-save shows a "saved" indicator. Sync shows a sync icon. The user should never wonder if something is happening.
- **Errors are loud, success is quiet.** Errors interrupt; success confirms briefly and gets out of the way.

## 8. Mobile & Responsive (Always Apply Unless App Is Explicitly Desktop-Only)

- **Design mobile-first.** If it works on mobile, it'll work on desktop. The reverse is rarely true.
- **Tap targets are at least 44×44px.** Smaller than that and fingers miss.
- **No hover-only interactions.** Hover doesn't exist on touch. Anything reachable by hover must also be reachable by tap or focus.
- **Test at 320px width.** That's the floor for modern phones. If it breaks below 375px, fix it.
- **Horizontal scroll is forbidden** unless it's an intentional carousel.

## 9. Accessibility Floor (Always Apply)

These are non-negotiable, not nice-to-haves. They are the **minimum** — the full standard lives in `accessibility-guardrails.md` (WCAG 2.2 AA, the POUR principles, conditional sections for forms, media, custom widgets, and the AT test matrix). Load that file alongside this one for any user-facing project.

- **Color contrast meets WCAG AA.** 4.5:1 for body text, 3:1 for large text. Use a contrast checker, don't eyeball it.
- **Keyboard navigation works for everything.** Tab order is logical. Focus is always visible. No keyboard traps.
- **Every form input has a label** programmatically associated with it.
- **Every image has alt text** (or `alt=""` if decorative).
- **Every button and link has accessible text** — no icon-only buttons without an `aria-label`.
- **Color is never the only signal.** Don't use red/green alone to communicate state — pair with an icon or label.
- **Respect `prefers-reduced-motion`.** No autoplay, no surprise animations for users who've opted out.

## 10. Copy & Microcopy (Always Apply)

- **Write like a human, not a robot.** "Something went wrong" beats "Error 500: Internal Server Error."
- **Be specific.** "Save your changes before leaving" beats "Unsaved changes."
- **Tell the user what to do, not what happened.** "Check your internet connection and try again" beats "Network error."
- **Sentence case for everything.** Headings, buttons, labels. No Title Case Everywhere.
- **No jargon, no marketing language inside the app.** Save it for landing pages.
- **Button text is a verb.** "Save changes," "Delete account," "Send invite." Not "OK," not "Submit."

## 11. Performance Is UX (Always Apply)

- **Largest Contentful Paint under 2.5s** on a mid-range mobile device.
- **No layout shift** after initial render. Reserve space for images, ads, and async content.
- **Lazy-load images and offscreen content.**
- **Optimistic UI** for fast-feeling interactions: update the UI immediately, reconcile with the server in the background, roll back if it fails.

## 12. Before Declaring a UI Change Done (Always Apply)

Walk through this checklist:

- [ ] All five component states present (empty, loading, partial, loaded, error)
- [ ] Works at 320px, 768px, and 1280px widths
- [ ] Keyboard navigable, focus visible
- [ ] Color contrast passes
- [ ] No console errors or warnings
- [ ] No layout shift on load
- [ ] Destructive actions confirmed or undoable
- [ ] Copy is specific, sentence case, and human
- [ ] Consistent with existing spacing, color, type, and component patterns
- [ ] Nothing was added that wasn't asked for

## 13. The Test That Matters Most

Show the screen to someone who has never seen the app before. If they hesitate for more than 3 seconds about what to do next, the screen has failed — regardless of how clean the code is.

You can't run this test in code, but you can ask yourself: **"If I came to this screen cold, would I know what to do?"** If the honest answer is "only because I built it," redesign.
