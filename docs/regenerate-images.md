# Regenerating the images

Everything in `docs/` except the architecture diagram is generated from the board
files with `kicad-cli`, so it can be rebuilt after a layout change instead of
re-screenshotted.

On Windows the binary is at:

    "C:/Program Files/KiCad/10.0/bin/kicad-cli.exe"

Run from the repository root. `$N` is `power` or `carrier`, `$B` is the matching
project folder under `hardware/`.

## 3D renders

    kicad-cli pcb render -o docs/$N-iso.png --quality high --perspective \
      --rotate '-30,0,-35' --zoom 0.9 --width 1800 --height 1350 \
      --background transparent hardware/$B/$B.kicad_pcb

    kicad-cli pcb render -o docs/$N-top.png --side top --quality high \
      --width 1800 --height 1350 --background transparent \
      hardware/$B/$B.kicad_pcb

Transparent backgrounds are deliberate. They read correctly on both GitHub's light
and dark themes.

## Copper layers

One SVG per layer, with the board outline for reference:

    for lp in "F.Cu:f-cu" "In1.Cu:in1-cu" "In2.Cu:in2-cu" "B.Cu:b-cu"; do
      L="${lp%%:*}"; S="${lp##*:}"
      kicad-cli pcb export svg -o "docs/$N-layer-$S.svg" --mode-single \
        --layers "$L,Edge.Cuts" --exclude-drawing-sheet --page-size-mode 2 \
        --check-zones hardware/$B/$B.kicad_pcb
    done

Then darken the board outline, since KiCad plots `Edge.Cuts` as `#D0D2CD` which
disappears on a white page:

    sed -i 's/#D0D2CD/#555555/g' docs/*-layer-*.svg

Silkscreen is left out on purpose. KiCad plots it as `#F2EDA1`, which is effectively
invisible against GitHub's light background. `B.Cu` is not mirrored, so every layer
lines up with every other one.

## Notes

- `--check-zones` refills pours before plotting, so the layer images always match the
  current zone settings.
- Close the KiCad PCB editor first. `kicad-cli` reads from disk, and an open editor
  can overwrite the board file when it saves.
- Regenerating always shows every file as modified. Raytraced PNGs differ by a few
  hundred bytes between runs and the SVGs carry a timestamp, so use a diff of the
  drawing elements rather than `git status` to tell whether anything really changed.
