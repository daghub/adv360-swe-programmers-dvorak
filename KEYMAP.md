# Swedish Programmer Dvorak Keymap

## Goal

This keymap makes a Kinesis Advantage360 Pro behave like Programmer Dvorak while
the host computer remains set to the standard Windows Swedish QWERTY layout.

The layout is carried by the keyboard firmware. The operating system does not
need the Kaufmann Programmer Dvorak keymap installed, and the laptop's built-in
keyboard can remain ordinary Swedish QWERTY.

Maintenance note: when changing the Advantage360 Pro keymap, also review
`advantage2/qwerty.txt` and `advantage2/README.md`. The Advantage2 layout is
maintained as a best-effort matching experience for systems using the Kaufmann
Programmer Dvorak Windows layout.

## What The Keymap Does

The Base layer implements the working text layout. It combines three things:

- Programmer Dvorak letter placement.
- Programmer Dvorak symbol and shifted-number behavior.
- The user's Advantage-style physical key moves, including moved Shift keys.

Important tested behavior:

```text
Q W E                  -> ; , .
Shift + Q W E          -> : < >
Z                      -> '
Shift + Z              -> "

Caps-labeled key       -> -
Shift + Caps-labeled   -> _

key right of Down      -> /
Shift + key right Down -> ?

key right of L         -> \
Shift + key right of L -> |
```

The top row is Programmer Dvorak style: symbols are unshifted and numbers are
shifted.

```text
unshifted: $ & [ { } ( = * ) + ] ! #
shifted:   ~ % 7 5 3 1 9 0 2 4 6 8 `
```

The moved Shift keys are real ZMK Shift bindings. This is important because the
symbol keys use modifier-aware ZMK behavior; they must see Shift as an actual
modifier, not as a macro side effect.

The circled `4` hotkey toggles a Swedish prose/forms layer. On that layer, the
physical keys that normally type Programmer Dvorak `[`, `{`, and `}` instead
type Swedish letters:

```text
[ key -> å
{ key -> ä
} key -> ö
```

Holding either real Shift key produces the uppercase forms: `Å`, `Ä`, and `Ö`.
Other keys fall through to the normal Base layer.

## Source Files

The main implementation is in:

```text
config/adv360.keymap
```

Supporting files from the upstream Advantage360 Pro config are still used:

```text
config/macros.dtsi
config/version.dtsi
config/west.yml
assets/key-positions.md
```

`assets/key-positions.md` documents the physical key matrix positions used by
the Advantage360 Pro.

## ZMK Implementation

### Base Layer

`default_layer` is the main typing layer. It contains the custom Programmer
Dvorak layout and the physical key moves.

The letter rows are placed directly with ordinary `&kp` bindings where possible.
For example, the Dvorak home row uses direct keypresses such as `&kp A`,
`&kp O`, `&kp E`, `&kp U`, `&kp I`, then `&kp D`, `&kp H`, `&kp T`,
`&kp N`, `&kp S`.

Navigation and modifier moves are also direct bindings. For example, the moved
Shift keys are `&kp LSHFT` and `&kp RSHFT`, while the old Shift positions are
used for `End` and `Page Down`. The labeled thumb-cluster `Alt` and `Windows`
keys are swapped so `Alt` is available on the right thumb for shortcuts such as
`Alt+Tab`. Two redundant lower thumb keys send Windows virtual desktop cycling
shortcuts: previous desktop is `Ctrl+Win+Left`, and next desktop is
`Ctrl+Win+Right`.

### Mod-Morph Behaviors

Most Programmer Dvorak symbol keys are implemented with
`zmk,behavior-mod-morph`.

Each mod-morph has two outputs:

- output 1 when no Shift modifier is held
- output 2 when either left or right Shift is held

The behaviors use:

```text
mods = <(MOD_LSFT|MOD_RSFT)>;
```

This lets a single physical key produce the Programmer Dvorak unshifted and
shifted outputs. For example:

```text
dvp_lbrack_7:
  unshifted -> [
  shifted   -> 7
```

Because Windows is using Swedish QWERTY, the bindings do not always look like
the final character. They send the Swedish Windows key sequence that produces
the desired character. Examples:

```text
[  -> AltGr+8       -> &kp RA(N8)
{  -> AltGr+7       -> &kp RA(N7)
]  -> AltGr+9       -> &kp RA(N9)
}  -> AltGr+0       -> &kp RA(N0)
-  -> Swedish / key -> &kp FSLH
_  -> Shift+/       -> &kp LS(FSLH)
```

This is the central idea of the keymap: the keyboard emits Swedish-layout
keystrokes, but the resulting text is Programmer Dvorak.

### Dead-Key Macros

Some Swedish symbols are dead keys on Windows. They need a second `Space`
keypress to commit the standalone character.

These are implemented with `zmk,behavior-macro`:

```text
dvp_grave -> Swedish grave dead key, then Space
dvp_tilde -> Swedish tilde dead key, then Space
dvp_caret -> Swedish caret dead key, then Space
```

Those macros are used inside mod-morph behaviors where the shifted or unshifted
Programmer Dvorak output needs one of those symbols.

### Other Layers

The keymap keeps the standard Advantage360-style layer structure:

- `default_layer`: the custom Swedish Programmer Dvorak typing layer.
- `keypad`: numeric/keypad layer, mostly inherited from the upstream layout.
- `fn`: function-key layer.
- `mod`: keyboard-management layer for Bluetooth selection, bootloader,
  Studio unlock, version macro, battery status, backlight, and RGB controls.
- `swedish_email`: toggled by circled `4`; overrides the Programmer Dvorak
  `[`, `{`, and `}` keys with `å`, `ä`, and `ö`.
- `extra2` through `extra4`: reserved color layers.

The `keypad` layer is not intended to be a second fully tuned Programmer Dvorak
text layer. The Base layer is the layout that should be used for normal typing.

The `swedish_email` layer has a higher layer number than `fn` and `mod`. If it
is left on, those three Swedish letter positions continue to override the lower
layers until circled `4` is pressed again.

## Building Firmware

### GitHub Actions Build

The usual workflow is to build in GitHub Actions:

1. Commit changes on the `swe-programmer-dvorak` branch.
2. Push the branch to GitHub.
3. Open the repository's Actions tab.
4. Open the latest successful workflow run.
5. Download the `firmware-clique` artifact.
6. Extract the artifact.

Use `firmware-clique`, not `firmware-no-clique`, because this keyboard reports
firmware with Clique support.

The extracted files are timestamped and include a commit hash in the filename.
The important suffixes are:

```text
left.uf2
right.uf2
```

### Local Build

The upstream repo also supports a local container build with Docker or Podman:

```shell
make
```

The generated firmware appears in the `firmware` directory. Use:

```shell
make clean_firmware
```

to remove generated firmware without rebuilding the container image.

## Installing Firmware On The Keyboard

Flash the left and right halves separately.

1. Extract the `firmware-clique` artifact.
2. Connect the left half by USB.
3. Put the left half in bootloader mode with `Mod+macro1`.
4. Copy the `left.uf2` file to the mounted keyboard drive.
5. Wait for the drive to disconnect.
6. Power-cycle the keyboard halves.
7. Connect the right half by USB.
8. Put the right half in bootloader mode with `Mod+macro3`.
9. Copy the `right.uf2` file to the mounted keyboard drive.
10. Wait for the drive to disconnect, then power-cycle/reconnect normally.

The physical reset buttons can also enter bootloader mode if the keyboard
shortcuts are unavailable.

On Windows, a message such as "the wrong diskette is in the drive" can appear
after copying the UF2. If the drive disconnects and the keyboard restarts, the
flash usually succeeded.

## Post-Flash Verification

Keep Windows set to Swedish QWERTY while testing.

Minimum smoke test:

```text
Q W E                  -> ; , .
Shift + Q W E          -> : < >
Z                      -> '
Shift + Z              -> "
Caps-labeled key       -> -
Shift + Caps-labeled   -> _
key right of Down      -> /
Shift + key right Down -> ?
top row unshifted      -> $&[{}(=*)+]!#
top row shifted        -> ~%7531902468`
```

Swedish layer smoke test:

```text
circled 4              -> toggle Swe layer on
normal [ key           -> å
normal { key           -> ä
normal } key           -> ö
Shift + those keys     -> Å Ä Ö
circled 4              -> toggle Swe layer off
normal [ { } keys      -> [ { }
```

Also verify that the preserved physical moves still work:

```text
both moved Shift keys work as Shift
old left Shift position acts as End
old right Shift position acts as Page Down
arrow keys move correctly
right Ctrl-labeled key works as Ctrl
Ctrl, Alt, and Windows shortcuts still work
lower thumb desktop keys switch to previous/next Windows virtual desktop
```

`Mod+V` prints the build date, branch fragment, commit hash, and whether the
firmware is a Clique build. On a Swedish host layout, punctuation in that macro
may look slightly different, but the date/hash text should still identify the
firmware.
