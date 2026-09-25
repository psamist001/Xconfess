# Accessibility Checklist

Use this checklist when reviewing UI changes. It covers the primary workflows and
is intended to be applied per pull request.

## Touch targets

- [ ] Interactive controls (buttons, links, icon buttons, list rows, checkboxes,
      toggles, tabs) are at least **44x44 CSS px** in their tappable area.
- [ ] Icon-only buttons use padding or a minimum size so the hit area meets the
      guidance even when the glyph is smaller.
- [ ] Adjacent targets have enough spacing (>= 8px) to avoid mis-taps.
- [ ] Targets remain >= 44x44 CSS px at 200% zoom and with large system fonts.
- [ ] Controls inside dense lists/toolbars are audited individually; do not rely
      on the container size alone.

## Gesture alternatives

- [ ] Every gesture-only action (swipe-to-delete, swipe-to-reveal, long-press,
      drag, pinch) has a visible button or menu alternative.
- [ ] The button alternative is reachable without performing the gesture and is
      exposed to assistive technology (accessible name + role).
- [ ] Gesture affordances are discoverable (visible hint or label), not hidden.
- [ ] Keyboard users can trigger the same action without pointer gestures.

## Destructive actions on touch

- [ ] Destructive actions (delete, remove, revoke, discard) require explicit
      confirmation or a guarded activation (e.g. confirm dialog or undo).
- [ ] Swipe-to-delete does not fire on incidental scroll or partial swipe; a
      threshold and/or confirmation is required.
- [ ] Undo or a confirmation step is available after a destructive action.
- [ ] Accidental activation prevention is verified on touch, not only mouse.

## Testing

- [ ] Touch-focused tests cover destructive actions (confirm/undo paths).
- [ ] Tests cover the button alternative for each gesture-only action.
- [ ] Failure paths (cancel, dismiss, undo) are covered.
- [ ] Tests run on a touch-emulated viewport.

## Contributor test harness

The accessibility harness runs the same automated checks on every route so
contributors get consistent, low-friction feedback from a clean checkout.

### Fixtures

Fixtures live in `tests/a11y/fixtures/`. Each fixture is a JSON file that names a
route and the rules to assert against it:

```json
{
  "name": "settings",
  "route": "/settings",
  "rules": ["color-contrast", "label", "button-name"]
}
```

Add a fixture for any new route or workflow you touch. Keep the `route` value in
sync with the app router so failures point at a real page.

### Running the harness

From a clean checkout:

```
npm install
npm run frontend:lint && npm run frontend:test && npm run frontend:test:e2e
```

The harness is wired into `frontend:test:e2e`, so the validation command above
exercises it. To run only the accessibility harness:

```
npm run frontend:test:a11y
```

### Output

Every failure names the **route** and the **violated rule**, for example:

```
FAIL /settings  rule=color-contrast  node=button.submit
```

Use the route to reproduce the page and the rule to look up the fix in the
checklist sections above.

### Browser setup

The harness drives a headless browser. Install the browser once per machine:

```
npx playwright install --with-deps chromium
```

Set `A11Y_BASE_URL` to point the harness at a running app (defaults to the local
dev server). CI installs the browser automatically.

### Known limitations

- Automated rules cannot judge meaning, reading order, or focus management.
- Dynamic content (modals, toasts, async lists) may need an explicit wait in the
  fixture before it is scanned.
- Canvas, video, and third-party embeds are not covered by the automated rules.

### When manual testing is required

Run the manual checklist above whenever a change involves:

- Keyboard navigation, focus order, or focus trapping (modals, menus).
- Screen-reader announcements or live regions.
- Gesture-only interactions and their button alternatives.
- Destructive actions and their confirm/undo paths.
- Color, contrast, or motion changes that automated rules cannot fully judge.

### Failure triage

1. Reproduce the route named in the output.
2. Match the rule to the relevant checklist section and apply the fix.
3. If the rule is a false positive, document why in the fixture and note it in
   the pull request description.
4. Re-run `npm run frontend:test:a11y` before requesting review.

## Validation

Run before requesting review:

```
npm run frontend:lint && npm run frontend:test && npm run frontend:test:e2e
```
