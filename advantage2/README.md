# Advantage2 SmartSet Layout

This directory tracks the Advantage2 `qwerty.txt` layout that should feel close
to the Advantage360 Pro keymap in this repository.

Unlike the Advantage360 Pro ZMK firmware, this Advantage2 layout does not
implement Programmer Dvorak in the keyboard. It assumes the Windows PC already
has the Kaufmann Programmer Dvorak layout installed and active.

## File

Use:

```text
advantage2/qwerty.txt
```

as the contents of the Advantage2 v-Drive file:

```text
active/qwerty.txt
```

The file is intentionally comment-free because Kinesis recommends keeping layout
files to programming lines only.

## Relationship To The Advantage360 Layout

The file is based on the Advantage2 SmartSet baseline used as input when
building the Advantage360 Pro layout.

It keeps the existing Advantage2 behavior for:

- top-row rotation
- moved left and right Shift keys
- Escape on the old Delete position
- Home on the old Caps position
- Page Up on the quote position
- the existing punctuation/edge-key swaps

It adds the same Windows virtual desktop thumb behavior as the Advantage360 Pro:

```text
Home thumb key -> Ctrl+Win+Left
PgUp thumb key -> Ctrl+Win+Right
```

Those macros are present on both the top layer and keypad layer:

```text
{home}>{-lctrl}{-lwin}{left}{+lwin}{+lctrl}
{kp-home}>{-lctrl}{-lwin}{left}{+lwin}{+lctrl}
{pup}>{-lctrl}{-lwin}{right}{+lwin}{+lctrl}
{kp-pup}>{-lctrl}{-lwin}{right}{+lwin}{+lctrl}
```

The Advantage360 Pro Swedish prose/forms layer has no equivalent here. The
Advantage2 setup assumes Windows is using the Kaufmann Programmer Dvorak layout,
so the keyboard cannot rely on the host's Swedish `å`, `ä`, and `ö` key
positions the way the Advantage360 Pro firmware does.

## Installation

1. Enable Power User Mode on the Advantage2 with `Program+Shift+Esc`.
2. Open the v-Drive with `Program+F1`.
3. Back up the keyboard's current `active/qwerty.txt`.
4. Replace the contents of `active/qwerty.txt` with `advantage2/qwerty.txt`.
5. Save the file as plain text.
6. Eject the v-Drive from Windows.
7. Close the v-Drive with `Program+F1`, or replug the keyboard.
8. Make sure Windows is using the Kaufmann Programmer Dvorak layout.

Kinesis documents that layout changes take effect only after the v-Drive is
closed/unmounted or the layout is re-selected.

## Verification

With the Advantage2 connected and Windows set to Programmer Dvorak:

```text
Home thumb key -> previous Windows virtual desktop
PgUp thumb key -> next Windows virtual desktop
both moved Shift keys still work as Shift
old left Shift position still acts as End
old right Shift position still acts as Page Down
Escape and arrow behavior still matches the previous Advantage2 baseline
```

If the desktop-switch macros do not work reliably, create the same shortcuts once
with the SmartSet App or onboard macro recording, then inspect the generated
`qwerty.txt` lines. The exact macro timing may need tuning on some Windows
setups.

## Sources

The Advantage2 user manual documents the direct-editing model:

- layout files are simple text files on the v-Drive
- remaps use `[location]>[action]`
- macros use `{trigger}>{actions}`
- modifier down/up actions use tokens such as `{-lshift}` and `{+lshift}`
- changes take effect after saving and closing/unmounting the v-Drive

Kinesis support page:

```text
https://kinesis-ergo.com/support/advantage2/
```
