# Keep Awake detail in the Dynamic Island

Long-pressing the Keep Awake shortcut in the Controls rail opens a detail page inside the island. A short tap still toggles the session. The detail replaces the shortcut rail with a full-height card that shows the current state, exposes configuration normally found only in the app panel, and lets VoiceOver users reach the same page through an accessibility action.

## What the detail shows

**Active session.** A toggle, the remaining time (live countdown) or "until disabled" label, and extend chips (+15 / +30 / +60 min). The extend chips use the same capsule button style as the rest of the island.

**Inactive.** A toggle, a duration picker for the default keep-awake length, and an "Until" row with a time picker and Start button.

**Options (always visible).** Clamshell mode toggle with setup status caption, display sleep toggle, and a button that navigates to the icon picker sub-page. The icon picker animates in from the trailing edge and has its own back button.

## Layout

The detail card uses `keepAwakeSize(active:)` on the expanded geometry: 215 points when active (fewer rows), 255 when inactive (duration + until rows). The size respects the custom height cap and screen inset like every other expanded page.

## Navigation

`NotchService.showKeepAwakeDetail()` guards on `AppFeature.keepAwake.isAvailable` and calls `open(.controls, keepAwake: true)`. The `open` method sets `showingKeepAwake` only when the feature is available. Going back, collapsing, or syncing preferences when the feature has been disabled all clear the flag. Opening sections from the detail preserves `showingKeepAwake`; returning from sections restores it.

The header shows the localized keep-awake title when the detail is active, using the same `showsDetail` path as the app panel and metric details.

## Long-press button

`NotchLongPressButtonStyle` uses a drag gesture to distinguish a short tap from a hold. Pressing and holding past the 0.4-second threshold fires the long-press action (open the detail); releasing before that fires the normal toggle. Dragging outside the tile's visible bounds cancels the gesture entirely — neither action fires. The containment check uses the tile's exact rectangle so releasing just outside cancels cleanly.

## Automated checks

Run on a supported Mac:

```sh
./build.sh --test-suite=notch
./build.sh --dev
./build/VorssaintDeveloper --selftest
```

`NotchDestinationTests.keepAwakeDetailContracts` covers:
- Opening with `keepAwake: true` shows the detail on the controls module
- Going back clears the detail without collapsing the island
- Opening and closing sections preserves and restores the detail flag
- Collapsing the island resets the detail
- Opening the detail with the feature disabled does not show it

## Manual macOS smoke checks

- Long-press the Keep Awake shortcut in the island's Controls section. The detail page should slide in with the toggle, duration and until rows. Toggle on, verify the countdown and extend chips. Toggle off, verify the duration picker and time picker.
- Drag off the Keep Awake tile without releasing — releasing outside should do nothing (no toggle, no detail). A short tap should toggle without opening the detail.
- Open the icon picker from the detail, change the icon, go back. The detail should remain. Open sections from the detail, close sections — the detail should still be there.
- Disable Keep Awake in settings while the detail is open. The island should fall back to the controls home without the detail.
- Test with VoiceOver: navigate to the Keep Awake shortcut and use the "Keep Awake Options" accessibility action. The detail should open.
