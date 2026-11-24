# Manual testing guide

Automated checks find failures and point at risks. They never confirm that a criterion passes. This guide covers the manual part of a WCAG 2.2 A/AA audit with this extension: what to test, how to use the automated findings, and what evidence to record so a "Supports" rating holds up.

## Before you start

- **Run the audit with the deeper checks** (`"audit": true` in `a11y.config.json`, see the README). Export the report: the **WCAG conformance risk** table is your test plan. Criteria marked *High* have automated failures, *Medium* have findings to verify, *Unknown* have no automated coverage at all.
- **Set up assistive technology.** Use at least one desktop screen reader and browser pair that the client's users have:

  | Platform | Screen reader | Browser |
  |---|---|---|
  | Windows | NVDA (free) or JAWS | Chrome, Firefox or Edge |
  | macOS | VoiceOver | Safari |
  | iOS / Android | VoiceOver / TalkBack | Safari / Chrome |

- **Know the scope.** Test every page and state in the config, plus complete processes (sign-up, checkout): WCAG conformance applies to whole processes, not single screens.

## How to use the automated findings

| Finding | What to do |
|---|---|
| **Automated violation** (verified-fail) | Confirm it is real (rare false positives), rate the criterion Partially Supports or Does Not Support, and add it to the remediation plan. |
| **Needs human review** | Follow the test procedure in the finding. Record the result as evidence (pass or fail) on the criterion. |
| **Information** | Context only, e.g. "no main landmark". Use it while testing related criteria. |
| No finding | Nothing was detected. Test the criterion by hand like any other. |

Each finding card has a **Raw evidence** section (focus before and after a key press, ARIA states, measured contrast) and, where relevant, a screenshot. The focus review page shows the before/after images of each Tab stop.

## Keyboard (2.1.1, 2.1.2, 2.4.3, 2.4.7, 2.4.11, 2.4.13)

1. Put the mouse away. Start at the address bar and press Tab through the whole page.
2. Every interactive element must be reachable and operable: Enter on links and buttons, Space on checkboxes and buttons, arrows inside tabs, menus, radio groups, listboxes, sliders and grids, Escape to close dialogs, menus and popups.
3. Focus order must follow the meaning of the page. Shift+Tab must go back the same way.
4. You must always see where focus is, and the focused element must not be hidden behind sticky headers, cookie banners or chat widgets.
5. Open every dialog: focus moves into it, stays in it while it is modal, and returns to the control that opened it when it closes.
6. After a client-side navigation, focus must land on the new content (or the page heading), not at the top of the document or nowhere.

Evidence: **Keyboard test notes** with the path you tested ("Tab ×7 to Save, Enter opens confirm dialog, Escape returns focus to Save"), plus a screenshot for anything visual (focus indicator, obscured element).

## Screen readers (1.1.1, 1.3.1, 2.4.2, 2.4.4, 2.4.6, 3.3.2, 4.1.2, 4.1.3)

1. Navigate by headings, landmarks, links and form fields (NVDA: H, D, K, F; VoiceOver: rotor). Compare with the accessibility-tree snapshot in the evidence folder.
2. Every control has a name that matches its visible label, the right role ("button", "checkbox", "tab") and its state ("expanded", "selected", "checked").
3. Images: informative images are described; decorative ones are silent.
4. Trigger each status message (save, search results, errors, toasts) **without moving focus to it**: it must be announced.
5. Submit forms with errors: each error is announced, names the field, and is read again when you move to the field.

Evidence: **Screen-reader transcript** with what you heard, verbatim, plus the screen reader, version, browser and OS. Example:

```
NVDA 2026.1, Chrome 142, Windows 11 — /checkout, Tab to the email field after submitting empty:
"Email, edit text, invalid entry, required. Enter an email address like name@example.com"
```

## Zoom, reflow and text spacing (1.4.4, 1.4.10, 1.4.12, 1.3.4)

1. Zoom to 200% and to 400% at 1280 px width (or use the 320 px viewport). Everything must still be readable and usable without horizontal scrolling, except content that needs two dimensions (data tables, maps, diagrams, code).
2. Apply the WCAG text-spacing bookmarklet: no text may be cut off or overlap.
3. Rotate a phone: the content must work in both orientations.

The reflow finding names the element where the overflow starts and the likely cause; check whether it is allowed two-dimensional content. Evidence: screenshots at 400%.

## Contrast and use of color (1.4.1, 1.4.3, 1.4.11)

axe checks text contrast but cannot decide text over images or gradients, states drawn only with color, or most non-text contrast. Use a contrast analyser on the elements axe flagged for review and on icons, input borders and focus indicators. Evidence: screenshot plus the measured ratio.

## Timing, motion and other criteria

Review 2.2.1 (time limits, including toasts that disappear quickly: see the *Message disappears quickly* finding), 2.2.2 (moving content), 2.3.1 (flashes), 2.5.x (pointer gestures, target size, dragging), 3.2.x (predictable behaviour), 3.3.7 and 3.3.8 (redundant entry and accessible authentication) from the checklist's test steps.

## Recording evidence and signing off

In the checklist (**A11y: Open Checklist**), for each criterion:

1. Choose the rating.
2. **Add evidence**: type, pass or fail, page or URL, selector, test date, tester, assistive technology, browser and OS, a screenshot saved in the workspace (e.g. `docs/evidence/2.4.7-save-button.png`), a transcript, and a remediation link for failures.
3. **Sign off**: your name, the assistive technology, browser and OS. The sign-off records the date and the audit run.

The VPAT treats a rating as confirmed only when:

- **Supports** has a sign-off and at least one piece of manual evidence, and no manual evidence that records a failure;
- **Not Applicable** has a reason in the remarks;
- no automated violation contradicts it, or an override explains why for the latest audit run.

Changing a rating removes its sign-off. A new audit run that finds new failures turns the VPAT back into a draft until you review them.

## Re-testing

After fixes, run the audit again and use **A11y: Compare Runs** to see what was fixed, what is new and what remains. Re-test fixed criteria by hand, update the evidence, and sign off again.
