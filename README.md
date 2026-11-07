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

Built for [Omarchy](https://omarchy.org) on Hyprland.

## How it works

- **Capture is Omarchy's.** Every shot runs `omarchy-capture-screenshot … save`,
  so the region picker, the screen freeze and the output folder are the same as
  a normal screenshot. Each shot is saved the moment it is taken, and the newest
  is copied, so closing the tray never loses anything.
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

Requirements: Omarchy (for `omarchy-capture-screenshot`), Python 3 with
PyGObject, GTK 4, libadwaita, ImageMagick 7 (`magick`), wl-clipboard, and
`gdbus`. gtk4-layer-shell is optional; without it the tray is an ordinary
window.

```sh
git clone https://github.com/blurxy/shotpile ~/shotpile
ln -s ~/shotpile/shotpile ~/.local/bin/shotpile
```

Bind it to your screenshot key (Omarchy's Lua config, `~/.config/hypr/bindings.lua`):

```lua
o.bind("SUPER + SHIFT + code:11", "Screenshot · again to pile up",
  "command -v shotpile >/dev/null && exec shotpile shot || exec omarchy-capture-screenshot")
```

Commands: `shotpile shot` (take one, or add to an open tray), `shotpile done`
(open the copy/save panel), `shotpile close`.

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

5. **It depends on Omarchy's screenshot script.** It is a wrapper, not a
   standalone capture tool.

## Style

The palette and rules are [SPORE](https://github.com/blurxy/spore)'s
(`src/ui/mycelium.js`): every colour is a state. Cyan is the established pile,
magenta is only ever the newest shot, and everything else is desaturated. No
glows, no gradients; depth comes from a 1px edge.

## License

[AGPL-3.0](LICENSE)
