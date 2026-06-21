# Advantage360 Pro Swedish Programmer Dvorak Plan

## Goal

Make the Kinesis Advantage360 Pro behave like Programmer Dvorak while the
Windows 11 host layout remains Swedish QWERTY at all times.

The laptop keyboard should continue to behave as ordinary Swedish QWERTY. The
Kinesis should carry the special layout in firmware.

## Current Keyboard/Firmware State

Keyboard: Kinesis Advantage360 Pro.

Version reported by `Mod+V`:

```text
20250210+v3.0+091a5d4+clique
```

Notes:

- The `+` characters are likely the Swedish host layout's interpretation of the
  physical key used for hyphen in the version macro.
- The important parts are `v3.0`, commit/version `091a5d4`, and `clique`.
- Clique/ZMK Studio support is present, but this layout needs GitHub/ZMK source
  editing because Clique cannot edit user-defined behaviors such as mod-morph.

Repo:

```text
/home/dekengren/dev/adv360-swe-programmers-dvorak
```

Branch:

```text
swe-programmer-dvorak
```

Primary file to edit:

```text
config/adv360.keymap
```

Supporting files if needed:

```text
config/macros.dtsi
config/keymap.json
assets/key-positions.md
```

## Hard Requirements

- Windows stays on Swedish QWERTY.
- Do not depend on the Kaufmann Programmer Dvorak Windows layout.
- Moved Shift keys must be preserved.
- The top row must behave like Programmer Dvorak:
  - unshifted top row emits Programmer Dvorak symbols
  - shifted top row emits numbers
- The moved Shift keys must be real ZMK Shift bindings, not macro tricks.
- Prefer firmware-level behavior over host-side tools.

## Advantage2 Remaps To Preserve As Intent

These are from the Advantage2 baseline. They should not be copied literally into
ZMK, but their behavior should be translated to the Advantage360 Pro layout.

### Top Row Rotation

The old SmartSet file moved the Advantage top row by one position:

```text
[=]>[1]
[1]>[2]
[2]>[3]
[3]>[4]
[4]>[5]
[5]>[6]
[6]>[7]
[7]>[8]
[8]>[9]
[9]>[0]
[0]>[hyphen]
[hyphen]>[=]
```

Intent: preserve the user's established Kinesis/Programmer-Dvorak top-row
positioning, but implement final shifted/unshifted behavior with ZMK
`mod-morph` instead of SmartSet remaps/macros.

### Moved Shift Keys

The moved Shift keys are mandatory:

```text
[lshift]>[end]
[end]>[lshift]
[pdown]>[rshift]
[rshift]>[pdown]
```

Intent on Advantage360 Pro:

```text
physical End position       -> Left Shift
physical left Shift position -> End
physical PageDown position  -> Right Shift
physical right Shift position -> Page Down
```

These should be real ZMK `LSHFT`/`RSHFT` bindings so mod-morph sees them as
actual modifiers.

### Other Navigation/Modifier Moves

Preserve these behaviors unless testing shows they no longer make sense on the
Advantage360 physical layout:

```text
[caps]>[home]
[home]>[kp-lwin]
[kp-home]>[kp-lwin]
[prtscr]>[caps]
[delete]>[escape]
[']>[pup]
```

Notes:

- `[caps]>[home]` means the Advantage2 physical Caps position. On the
  Advantage360 Pro, that is the key labeled `Esc`, not the key labeled `Caps`.
- Final decision: do not preserve `[rctrl]>[left]`; the key labeled `Ctrl`
  should remain a real right Ctrl key.
- `[']>[pup]` means the physical quote location was used as Page Up on the
  Advantage2 baseline.

### Punctuation/Edge-Key Moves

Preserve the intent of these physical-key moves where applicable:

```text
[\]>[obrack]
[obrack]>[\]
[cbrack]>[`]
[`]>[cbrack]
[intl-\]>[']
```

Notes:

- On the Advantage360 Pro, the key labeled `Caps` is used for the Programmer
  Dvorak `-`/`_` key.
- These moves are likely tied to the old Windows Programmer Dvorak/Kinesis
  setup, so validate them carefully before baking them into the final ZMK
  layout.

## Programmer Dvorak Target Behavior

The target top-row behavior should match Programmer Dvorak, but emitted through
Windows Swedish.

Known top-row target, left to right:

```text
unshifted: $ & [ { } ( = * ) + ] ! #
shifted:   ~ % 7 5 3 1 9 0 2 4 6 8 `
```

This must be checked against the final physical Advantage360 key positions. The
Advantage360 default top row is not a normal ANSI row; it has Kinesis-specific
positions such as `EQUAL` on the far left and `MINUS` on the far right.

## Swedish Windows Output Layer

Because Windows is Swedish QWERTY, symbols must be produced using Swedish
Windows key sequences, not US key assumptions.

Examples that will likely be needed:

```text
@  -> AltGr+2
{  -> AltGr+7
[  -> AltGr+8
]  -> AltGr+9
}  -> AltGr+0
\  -> AltGr+plus/acute area, verify on Windows
|  -> AltGr+less-than key, verify on Windows
^  -> Swedish dead key then Space, verify exact key
`  -> Swedish dead key then Space, verify exact key
```

Do not use Linux XKB as the source of truth for this. Validate on the actual
Windows 11 Swedish layout.

## Implementation Strategy

### Phase 1: Baseline Physical Layout

Edit `config/adv360.keymap` to preserve the user's physical layout preferences
without implementing Programmer Dvorak yet.

Minimum test targets:

```text
moved Left Shift + a      -> A
moved Right Shift + a     -> A
original left Shift key   -> End
original right Shift key  -> Page Down
Delete position           -> Escape
Caps position             -> Home
arrow keys                -> left/right/up/down correctly
```

Build and flash this first if there is any uncertainty about key positions.

### Phase 2: Dvorak Letter Placement

Move alphabetic output to Programmer Dvorak letter positions. Since Swedish
QWERTY keeps ordinary A-Z letter keycodes in normal positions, the letter layer
should mostly be ordinary ZMK keycodes placed on different physical keys.

Test:

```text
abcdefghijklmnopqrstuvwxyz
ABCDEFGHIJKLMNOPQRSTUVWXYZ
common shortcuts: Ctrl+C, Ctrl+V, Ctrl+Z, Ctrl+S
```

### Phase 3: Top Row With Mod-Morph

Define one ZMK `mod-morph` behavior per top-row key where necessary:

```text
tap key alone          -> Programmer Dvorak symbol
hold either Shift+key  -> Programmer Dvorak number
```

The mod-morph should react to both left and right Shift:

```text
MOD_LSFT | MOD_RSFT
```

Do not set `keep-mods` for these number outputs, because Shift must not be sent
along with the number.

### Phase 4: Swedish-Specific Symbol Behaviors

For each Programmer Dvorak symbol that is not a plain Swedish keypress, create
the needed ZMK macro or behavior:

```text
AltGr sequences
Shift sequences
dead-key + Space sequences
```

Keep these isolated and named clearly so they can be corrected if a Swedish
Windows sequence is wrong.

### Phase 5: Build And Flash

Use the existing GitHub Actions build flow from the Kinesis fork.

Expected firmware artifacts:

```text
left.uf2
right.uf2
```

Flash flow:

```text
left half  -> bootloader -> copy left.uf2
right half -> bootloader -> copy right.uf2
```

Use the current Kinesis V3/Clique flashing instructions for bootloader entry.

### Phase 6: Acceptance Test On Windows Swedish

Test with Windows input layout set to Swedish QWERTY.

Checklist:

```text
Moved left Shift works as Shift
Moved right Shift works as Shift
Original Shift locations produce End/PageDown as intended
Unshifted top row produces Programmer Dvorak symbols
Shifted top row produces numbers immediately on keypress
Letters match Programmer Dvorak
Swedish laptop keyboard still works normally
No Kaufmann Programmer Dvorak host layout is active
```

## Workflow Rules

- Treat GitHub/ZMK files as source of truth.
- Avoid making lasting layout edits in Clique while developing this branch.
- Keep changes small and testable:
  1. physical key moves
  2. letters
  3. top-row mod-morph
  4. Swedish symbols
- Do not overwrite a working firmware without retaining the corresponding Git
  commit that produced it.
