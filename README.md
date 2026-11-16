<div align="center">

# Free My TV — this fork

A personal fork of **[simbasjungles/free-my-tv](https://github.com/simbasjungles/free-my-tv)**, customized for one
LG webOS TV (an LG OLED55C1PSA). For what the project itself is, how to install it, and everything else about it,
see [the original repository](https://github.com/simbasjungles/free-my-tv) and its own README — this fork changes
none of that.

</div>

## What's different here

Everything below is specific to this fork and this TV; none of it is upstream behavior.

- **The shelf's icons** for a few apps (Netflix, YouTube, Crunchyroll, Stremio) were replaced or added, since the
  TV's own reported icon for some of them didn't load. See [`web/icons/SOURCES.md`](web/icons/SOURCES.md) for where
  each one came from and why.
- **A performance fix** (`web/tv.js`): LG TVs reporting 2 GB of memory now get the same shadow-free "flat" rendering
  mode Samsung TVs at that memory already had — the shelf visibly dropped frames moving the pointer across it
  without this.
- **This TV's own setup** (`config.properties`, not committed): shelf app order, city, and TMDB key.

On top of that, this TV is **rooted** (at the owner's own risk, unrelated to Free My TV itself) for a few further
customizations, each documented in its own file:

- [`webos/rooting-slopbro.md`](webos/rooting-slopbro.md) — rooting the TV in the first place, with
  [SlopBro](https://github.com/throwaway96/slopbro).
- [`webos/root-autostart.md`](webos/root-autostart.md) — a script that opens Free My TV on its own, on every real
  reboot and every time the TV wakes from standby.
- [`webos/magic-mapper-buttons.md`](webos/magic-mapper-buttons.md) — remapping the remote's **Home** button to open
  Free My TV directly, with the **red** button as a backup to LG's own launcher, using
  [andrewfraley/magic_mapper](https://github.com/andrewfraley/magic_mapper).

## License

MIT, same as upstream — see [`LICENSE`](LICENSE).
