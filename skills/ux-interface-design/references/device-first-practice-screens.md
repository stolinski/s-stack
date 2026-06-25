# Device-first practice screens

Use this reference when tightening an app/workspace that teaches a physical device, instrument, controller, or hands-on workflow.

## Prime directive

The device and the current action are the workspace. Everything else is support.

A good practice screen should answer, in order:

1. What am I working on?
2. What do I touch/do now?
3. What should happen when I did it right?
4. How do I move to the next step?
5. Where can I recover/reference details if I need them?

If visible text or chrome does not serve one of those, delete it or demote it.

## Deletion pass checklist

Before styling, mark each visible element:

- Keep: directly supports the current action, device location, check/result, or next step.
- Shrink: useful but over-prioritized, such as progress, reset, time, pack metadata.
- Move: useful later, such as reference notes, excerpts, taxonomy, setup detail.
- Delete: repeats what the layout already says or exposes implementation/data-model labels.

Common deletes:

- visible labels like “Device,” “Registry,” “Lesson packs,” “Missions,” “Song map,” “Current Move” when structure already communicates them
- duplicate control pills if the device overlay and “Touch Now” already name/highlight the controls
- source/reference pills inside the active task panel
- helper copy explaining that highlighted controls are highlighted
- decorative cards whose only job is to wrap another card

Keep accessible names, semantic regions, and nav labels even when removing visible labels.

## Layout rules

- Root/landing routes may use hero/marketing treatment. Deep practice routes should be workspace-first.
- Avoid card-inside-card-inside-card. One main surface, then direct controls.
- Dense is fine. Dead space is not.
- Use hierarchy and proximity instead of borders around every idea.
- Make “touch/do now” the strongest short text near the device.
- Put metadata in a compact strip or disclosure: pack name, duration, move count, references.
- “Do” and “Done when” are useful, but do not trap them in bulky twin boxes if width is constrained. Inline/stacked rules often read cleaner.
- Progress is state, not the hero. Keep it compact.

## OP-XY workbench case study

In the Teenage Eng OP-XY app, the useful practice shape was:

- photo/diagram of the OP-XY as the main visual surface
- invisible/SVG hit regions over the photo for highlights and interaction
- “Touch Now” directly below the device image
- compact move map below the device
- current move panel to the side
- collapsed reference/details at the bottom

The cleanup pass removed:

- device explainer card
- device registry card
- visible section labels for lesson packs, missions, and song map
- current-move kicker label
- duplicate controls list
- source pill in the active move panel
- nested boxes around the instruction/check pair

The important distinction: the photo was the face; the SVG overlay was the touch map. Do not replace a good realistic device image with SVG art just to have SVG. Use SVG regions as invisible/semantic hit areas on top of the real image.

## Verification

For UI cleanup, do not stop at DOM inspection. Open the app visually and compare before/after for:

- fewer visible labels
- less double padding
- fewer nested boxes
- device/action occupying more of the first viewport
- right-panel text with enough width to avoid awkward word stacks
- no broken layout at common desktop width

Then run the project’s normal type/build/content checks.