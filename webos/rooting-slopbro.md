# Rooting an LG TV with SlopBro (before any of the other `webos/*-*.md` docs apply)

This is the step before [`root-autostart.md`](root-autostart.md) and [`magic-mapper-buttons.md`](magic-mapper-buttons.md):
both assume the TV is already rooted with the Homebrew Channel installed. None of this is part of Free My TV itself
— it's a third-party exploit, used entirely at your own risk, on your own TV. Read
[webosbrew.org](https://webosbrew.org)'s own rooting guidance too before doing this on a TV that matters to you.

## 1. Check the TV is actually exploitable

Go to **[cani.rootmy.tv](https://cani.rootmy.tv)** and enter the TV's exact model number and firmware version (on
the TV: Settings → General → System → About This TV, or About → check for updates). It lists every known exploit
for every webOS model/firmware combination, each marked patched or not. SlopBro is the one documented here; if the
checker shows it (or any exploit) as not patched for your exact version, you're good. If every exploit shows
patched, there's no way in at that firmware — don't try. A software update can patch it, so check the current
firmware, not an older remembered one.

## 2. Get SlopBro and run it

[throwaway96/slopbro](https://github.com/throwaway96/slopbro) (AGPL-3.0, no relation to this project). Needs Python
3.8+, the TV's IP address, and the TV and your computer able to reach each other both ways on the network.

```bash
git clone https://github.com/throwaway96/slopbro.git
cd slopbro
python3 slopbro.py <TV_IP>
```

The TV shows a pairing prompt — accept it. The script then serves a small exploit page to the TV over your LAN; it
downloads a fake system service package and runs it as root, which installs the webOS Homebrew Channel and sets up
persistent root. Progress and errors print to the terminal; `--debug` adds more detail, written on the TV at
`/tmp/autoroot.log`. The script itself doesn't need a reboot to finish, but it does finish by asking (as an
on-screen TV alert) whether to reboot now — do that, since Homebrew Channel only fully "appears" after one.

## 3. Before you reboot: uninstall LG's own Developer Mode app, if it's on the TV

This tripped us up the first time: with LG's **Developer Mode** app still installed, the root didn't take properly
(or looked like it hadn't). SlopBro's own script checks for this and refuses to finish setting up persistence while
it finds that app installed — remove it from the TV's app list before running SlopBro (or before the final reboot,
if you already ran it with Developer Mode still on), then reboot.

## 4. After rebooting: confirm it worked

**Homebrew Channel** should now be in the TV's app list. Open it once — this is also how you turn on Telnet or SSH
(in its own Settings), which the other two docs' commands need. The default root password over SSH is `alpine`; **Telnet
has no password at all**, so turn it off again once you're done with it for the session, and prefer SSH with your
own key for anything longer-term. See `webosbrew.org`'s own docs for exactly where those toggles are in Homebrew
Channel's settings screen, since the exact wording has changed between its versions.

## Caveats

- Don't reinstall LG's own **Developer Mode** app once rooted — the two don't coexist cleanly (see step 3).
- An LG software update, or a factory reset, removes the root — re-running SlopBro (after re-checking
  cani.rootmy.tv, in case that update also patched the exploit) is how you'd get it back.
- SlopBro's own README says it isn't expected to work on webOS 10/11 firmware released after it was patched
  (around October 2026); for older webOS 3.5/4.x, its author points at a different tool,
  [dejavuln-autoroot](https://github.com/throwaway96/dejavuln-autoroot), instead.
- Rooting voids whatever LG considers "normal" support for the TV, and carries a small real risk of making the TV
  unusable if something goes wrong mid-process (a power loss during the brief root-install step, say). It worked
  cleanly on the TV this project's root-dependent docs were written against (an LG C1, webOS 6.5.3), but that's one
  data point, not a guarantee for yours.
