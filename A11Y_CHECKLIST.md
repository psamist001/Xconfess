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

## Validation

Run before requesting review:

```
npm run frontend:lint && npm run frontend:test && npm run frontend:test:e2e
```
