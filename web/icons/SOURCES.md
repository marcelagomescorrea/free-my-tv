# Shelf icon sources (this TV's customizations)

The project's own bundled icons (`web/icons/*`) are mostly from Simple Icons (CC0) or Wikimedia Commons, used as
plain brand marks to identify each app's tile — the same way every launcher does. These four were replaced or added
for this TV, on top of what the project ships:

## netflix.svg — replaced
Was: the Simple Icons (CC0) "N" mark alone.
Now: the full "NETFLIX" wordmark, via Wikimedia Commons (`Netflix_2015_logo.svg`, PD-textlogo — a plain text logo,
below the threshold of originality), recolored to the brand red `#E50914`. Shown on the shelf's existing near-black
tile color.

## youtube_2.png — replaced (was youtube.png)
The official YouTube brand lockup (icon + wordmark, "almost black" text), from YouTube's own public brand kit
(brand resources page), provided by the TV's owner from their own download
(`YouTube_Logo/Digital/01 Full Color/yt_logo_fullcolor_almostblack_digital.png`). Composited onto a flat `#F2F2F2`
background (the shelf tile's color for this app) and cropped tight to the logo itself, so it fills the tile's fixed
icon box as much as possible.
Renamed from `youtube.png` partway through (twice, `_2`) only to dodge the TV's own asset cache, which kept an
older version after reinstalling with the same filename — not a licensing change.

## crunchyroll_3.png — replaced (was crunchyroll.svg)
The project's existing Simple Icons (CC0) "eye" mark, combined with a plain "crunchyroll" wordmark set by hand (a
generic rounded bold sans-serif, not Crunchyroll's own typeface) — not an official asset, just a lookalike built
from what the project already had. Composited onto a flat `#F47521` (the shelf tile's color for this app) and
cropped tight. First tried as a combined SVG (icon path + `<text>`), which an older/constrained SVG engine on this
TV rendered incorrectly (the text showed, the icon didn't); baking it to a flat PNG instead fixed it, the same way
`youtube_2.png` already works. Renamed to `_3` (after an interim `_2`) to dodge the asset cache, same as YouTube's.

## stremio_2.png — new
Free My TV ships no Stremio icon at all. This is original artwork made for this TV: a simple diamond (a rotated
rounded square) with a purple-to-blue gradient and a white play triangle, plus a plain "stremio" wordmark in a
generic sans-serif — not copied from Stremio's own branding, just built to look similar. Composited onto a flat
`#0A0A0A` background (the shelf tile's color for this app) and cropped tight. Renamed once (`_1` → `_2`) for the
same cache reason as the two above.

## Where these are wired in
- `web/tv.js`: `SAMSUNG_ICON_NAMES` (aliased as `LG_ICON_NAMES`, added for this TV — upstream only had the Samsung
  table) maps each app's name to its icon file; this is what picks these files over whatever icon path the TV's own
  `listLaunchPoints` reports (which failed to load for Crunchyroll on this TV, which is the whole reason `LG_ICON_NAMES`
  exists here).
- `web/home.js`'s `SHELF` table sets each tile's background color/gradient; Crunchyroll and Stremio were added here
  (flat colors, matching the baked-in PNG backgrounds above exactly — no gradient, so there's no visible seam), and
  YouTube's existing entry was changed from near-black to light gray to go with its tile.

## A note on the TV's icon cache
Reinstalling the app with a file at the *same path* can keep showing the old picture: this TV's app runtime seems to
cache images by their file path across reinstalls (and even across closing and relaunching the app), not by their
content. Changing an icon that doesn't visibly update is why several files above went through `_2`/`_3` names
instead of being overwritten in place — a new path is a guaranteed cache miss. A full reboot clears it too, but the
naming trick doesn't need one.
