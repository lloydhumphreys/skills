# Colors, theming & assets

All styling is **optional**. Diagrams render cleanly with none — only style a node when it
aids comprehension, never for decoration.

## Color names (preferred) vs hex

Reach for a **standard palette name** first — it keeps the diagram on-theme and lets a
repo-wide `vizzy.config` retheme it. Raw `#hex` still works, but `vizzy lint` nudges you
toward a name (the `prefer-theme-color` rule). Never invent `$tokens`.

The palette:

```
red, orange, yellow, green, mint, teal, cyan, blue, indigo, purple, pink,
brown, gray, black, white   (plus primary, secondary, accent)
```

Use them anywhere a color is accepted — `style api fill:blue,stroke:indigo`,
`frame alt healthy color:green`, `phase START: A->B color:blue`.

## Re-theming a whole repo with `vizzy.config`

Re-theming lives in config, **never in the diagrams**. Drop an optional `vizzy/vizzy.config`
that remaps what any standard name resolves to — the diagrams keep using plain names:

```
# vizzy/vizzy.config — optional palette overrides (name = #hex or color name)
red   = #E5484D
blue  = #0A84FF
green = #30A46C
```

Every `stroke:red` / `fill:red` then picks up the new red. Lines starting with `#` are
comments; unknown or malformed lines are ignored. With no config file, the built-in palette
is used — most repos won't need a config at all.

## Assets

Assets are entirely optional. You may reference images from `vizzy/assets/`, but no diagram
needs them. Skip assets unless an icon or logo adds real meaning. (Note that `vizzy render`
also *writes* rendered PNGs into `vizzy/assets/` by default — see [`cli.md`](cli.md).)
