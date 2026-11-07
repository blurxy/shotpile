# shotpile

Take several screenshots in a row, then choose how to copy or save them all.

Press your screenshot key: you get a normal screenshot (saved, and on the
clipboard) and a small tray in the corner. Press the key again while the tray is
up and the next shot joins the pile. Hit **done** and choose:

- **copy all**: one paste delivers every shot, the way a multi-file copy does
  in a file manager
- **copy as one image**: the pile stitched together (stacked, side by side, or
  grid)
- **save all** or **save as one image**: to the folder you last used, or pick
  one
- **format**: PNG, JPEG, WebP, AVIF or JXL

Each shot in the tray also works on its own, the way CleanShot X's quick-access
card does:

- **drag it** straight into Chrome, a chat app or a file manager
- **hover it** for copy · edit · pin · text (OCR) · QR · beautify
  - *edit* opens Omasnap's annotation editor
  - *pin* floats it above every workspace (`omasnap --pin`)
  - *text* copies the words in it (tesseract, offline)
  - *QR* copies a code in it and opens it if it is a link (zbarimg, offline)
  - *beautify* adds rounded corners, a shadow and a gradient in your Omarchy
    theme's colours, swaps it into the pile and copies it
- **keys** (click the tray first): Ctrl+C copies the newest, Ctrl+Shift+C the
  whole pile, Delete drops the newest, P pins it, T copies its text, Enter
  opens the panel, Esc closes
- **+ scrolling** captures a scrolling page into the pile; `shotpile restore`
  reopens the last pile you closed

Built for [Omarchy](https://omarchy.org) on Hyprland.

## How it works

- **Capture is Omarchy's.** Every shot runs Omasnap (`omasnap <mode> --save`),
  Omarchy's own screenshot tool since 4.5, so the region picker, the screen
  freeze and the output folder are the same as a normal screenshot. Omasnap
  prints no path, so shotpile takes the newest image saved since the capture
  began (Esc saves nothing, so nothing is added). Each shot is saved the moment
  it is taken, and the newest is copied, so closing the tray never loses
  anything. Without Omasnap it falls back to `omarchy-capture-screenshot`.
- **The tray is a GTK4 layer surface** (via gtk4-layer-shell). It floats above
  windows, takes keyboard focus only when you click it, and hides itself while
  you pick a region so it never lands in a shot.
- **copy all offers three formats in one clipboard entry**: a file list
  (`text/uri-list`), GNOME's copied-files format (what Nautilus reads), and one
  stitched PNG. Each app takes the richest one it understands. The process stays
  alive, serving the paste, until something else is copied.
- **The key stays fast.** GTK and libadwaita take ~300 ms to load, so the first
  press starts the picker before anything heavy loads, and a press while the
  tray is open reaches it over D-Bus without a second GTK process.

## Install

Requirements: Omarchy 4.5+ (for `omasnap`), Python 3 with PyGObject, GTK 4
(4.8+ for drag and drop), libadwaita, ImageMagick 7 (`magick`), wl-clipboard,
and `gdbus`. Optional: gtk4-layer-shell (without it the tray is an ordinary
window), `tesseract` for text, `zbarimg` for QR codes.

```sh
git clone https://github.com/blurxy/shotpile ~/shotpile
ln -s ~/shotpile/shotpile ~/.local/bin/shotpile
```

Bind it to your screenshot key (Omarchy's Lua config, `~/.config/hypr/bindings.lua`):

```lua
o.bind("SUPER + SHIFT + code:11", "Screenshot · again to pile up",
  "command -v shotpile >/dev/null && exec shotpile shot || exec omarchy-capture-screenshot")
```

Commands: `shotpile shot` (take one, or add to an open tray), `shotpile scroll`
(a scrolling capture), `shotpile restore` (reopen the last closed pile),
`shotpile done` (open the copy/save panel), `shotpile close`, and
`shotpile selftest`.

`shotpile selftest` takes one silent fullscreen shot into a temporary folder
and checks shotpile can find it. Run it after Omarchy updates: the 4.5 update
broke shotpile silently by swapping the capture tool. To run it automatically:

```sh
cat > ~/.config/omarchy/hooks/post-update.d/shotpile-selftest.hook <<'HOOK'
out=$("$HOME/.local/bin/shotpile" selftest 2>&1) && exit 0
notify-send -u critical "Screenshot key broke after the Omarchy update" "$out"
HOOK
```

## What it cannot tell you

Each of these was measured on the machine it was built on, not inferred.

1. **Which apps accept "copy all" as several files is untested.** The clipboard
   entry is verifiable (`wl-paste --list-types` shows all three formats), and
   Nautilus is designed to read GNOME's copied-files format. Whether a given
   browser, chat app or editor pastes the file list, the single stitched image,
   or nothing is up to that app, and hasn't been checked app by app yet.

2. **Nothing moves.** *save all* writes copies; the originals stay in the
   screenshot folder, where Omarchy saved them. In PNG to that same folder, it
   reports that everything is already there instead of writing duplicates.

3. **Stitching is ImageMagick, and not instant.** Three 2560×1600 shots took
   1.0–1.5 s to stitch as PNG, and 0.35 s as JPEG. It runs off the UI thread,
   with the tray showing progress.

4. **Latency depends on the power profile, so compare like with like.** On the
   power-saver profile, Omarchy's own picker appeared 320–360 ms after the key
   press and shotpile's 380–410 ms. An earlier 168 ms reading for Omarchy came
   from a boosted CPU. Adding to an open tray takes ~100 ms before its capture
   starts.

5. **It depends on Omarchy's screenshot tool.** It is a wrapper, not a
   standalone capture tool, and an Omarchy update can change the contract
   underneath it (4.5 did). `shotpile selftest` exists to catch that.

6. **Per-shot actions, measured on a 4K screen.** OCR read two test lines
   exactly in 0.7 s. Beautify takes ~2.8 s on a 3840×2160 shot: shots wider
   than 2560 px are scaled down to 2560 first (it was ~5 s at full size).
   A QR code needs its white margin: one cut off at a screen edge or covered
   by another window won't read.

7. **Scrolling capture is Omasnap's flow.** Drag the region, choose *Auto*,
   then *Done* when it reports the end; only then does the stitched image join
   the pile. It scrolls whichever way the page scrolls, sideways included.

## Style

The palette and rules are [SPORE](https://github.com/blurxy/spore)'s
(`src/ui/mycelium.js`): every colour is a state. Cyan is the established pile,
magenta is only ever the newest shot, and everything else is desaturated. No
glows, no gradients; depth comes from a 1px edge. (Beautify's gradient is in
the exported image, in your Omarchy theme's colours, not in the tray.)

## License

[AGPL-3.0](LICENSE). You may use, study, change and share shotpile; if you
distribute it, or a changed version, or run a changed version as a network
service, you must publish your source under the same license.

**Commercial license.** If you want to use shotpile in a product without the
AGPL's obligations (for example, a closed-source version), a commercial license
is available from the author: open an issue or contact
[@blurxy](https://github.com/blurxy).

**Contributions.** So that shotpile can stay dual-licensed, contributions are
accepted on the understanding that the author may also release them under the
commercial license. Say so in your pull request if you do not agree.
