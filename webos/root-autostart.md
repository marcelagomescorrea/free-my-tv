# Opening Free My TV on its own (rooted LG TVs)

This is an optional, TV-side setup for people who have **rooted their LG TV** (for example with
[SlopBro](https://github.com/throwaway96/slopbro), which installs the
[webOS Homebrew Channel](https://github.com/webosbrew/webos-homebrew-channel)). It is not part of the app, is not
built or installed by `webos/build.py` or `webos/install.py`, and has nothing to do with the project's own code —
it's a small shell script placed directly on the TV's root filesystem, using a mechanism the Homebrew Channel itself
provides. It's documented here only so the setup survives being forgotten, and so another rooted LG TV can repeat
it.

**Rooting a TV is your own decision and your own risk**, unrelated to this project; see the Homebrew Channel's own
docs and [cani.rootmy.tv](https://cani.rootmy.tv) for whether your model and firmware can be rooted at all.

## What it does

With this script in place, the TV opens straight into Free My TV:

- **On every real reboot** (a software update, a crash, or *Settings → General → Reboot*).
- **On waking from standby** (turning the TV off and back on with the remote) — this is the common, everyday case.
  webOS never reboots the TV for this, so a plain boot-time script (the obvious approach) never even runs here.

Both cases hit the same underlying race: for a while after the screen comes back (whether from a real boot or from
standby), the TV's own firmware keeps putting **Live TV** in front, even after Free My TV has already been launched
once. The script below keeps relaunching Free My TV — but only while something else still has the foreground, so it
never fights Free My TV once it has actually won, which is what caused the shelf to flicker in an earlier version of
this script (every relaunch call resets the page's focus).

## The script

Lives at `/var/lib/webosbrew/init.d/freemytv-autostart` on the TV (not in this repository — nothing here builds or
installs it). The Homebrew Channel's own startup script runs every executable file in that directory once, early at
every real boot (see its `startup.sh`'s `run-parts /var/lib/webosbrew/init.d`), which is the hook this relies on.

```sh
#!/bin/sh
LOG=/tmp/freemytv-autostart.log

# Makes sure Free My TV is the foreground app, retrying every 3s for up to 2
# minutes. Does nothing (no flicker) once it already holds the foreground, and
# stops as soon as it does.
ensure_foreground() {
  i=0
  while [ "$i" -lt 40 ]; do
    fg=$(luna-send -n 1 -a com.webos.service.applicationmanager luna://com.webos.applicationManager/getForegroundAppInfo '{}' 2>&1)
    case "$fg" in
      *'"appId":"io.freemytv"'*)
        echo "[$(date)] ensure_foreground: already foreground (i=$i)" >> "$LOG"
        return 0
      ;;
    esac
    result=$(luna-send -n 1 -a com.webos.service.applicationmanager luna://com.webos.applicationManager/launch '{"id":"io.freemytv"}' 2>&1)
    echo "[$(date)] ensure_foreground attempt $i: fg=$fg launch=$result" >> "$LOG"
    i=$((i+1))
    sleep 3
  done
}

# Fixes the race right after boot: the TV shows Live TV for a while even after
# our first launch succeeds.
ensure_foreground

# Waking from standby (turning the TV off/on with the remote) never reboots it,
# so this script doesn't run again then -- yet the TV shows Live TV there too.
# This watches the TV's power state for the rest of this boot session and
# re-runs the same fix every time it wakes back up to "Active".
(
  prevstate="unknown"
  luna-send -i luna://com.webos.service.tvpower/power/getPowerState '{"subscribe":true}' 2>/dev/null | while IFS= read -r line; do
    case "$line" in
      *processing*)
        : # mid-transition; ignore
      ;;
      *'"state":"Active"'*|*'"state": "Active"'*)
        if [ "$prevstate" = "Suspend" ]; then
          echo "[$(date)] woke from standby, re-checking" >> "$LOG"
          ensure_foreground
        fi
        prevstate="Active"
      ;;
      *'"state":"Suspend"'*|*'"state": "Suspend"'*)
        prevstate="Suspend"
      ;;
    esac
  done
) >> "$LOG" 2>&1 &

exit 0
```

The background part (the last `( ... ) &`) is what covers the standby case: it stays subscribed to
`com.webos.service.tvpower`'s power state for as long as the TV stays on this boot, and only acts on the clean,
settled `{"state":"Active"}` / `{"state":"Suspend"}` lines (not the many in-between `"processing"` ones a real
power-cycle also sends).

## Installing it

With the Homebrew Channel's SSH or Telnet enabled (its own *Settings*), from a computer on the same network:

```bash
ssh root@<TV IP>          # or: telnet <TV IP>  (Homebrew Channel's default: no password over Telnet)
```

Then on the TV, write the script above to `/var/lib/webosbrew/init.d/freemytv-autostart` and make it executable:

```sh
cat > /var/lib/webosbrew/init.d/freemytv-autostart << 'SCRIPTEOF'
# ...the script above...
SCRIPTEOF
chmod +x /var/lib/webosbrew/init.d/freemytv-autostart
```

It takes effect on the next real reboot (`luna-send -n 1 -a com.webos.service.sleep luna://com.webos.service.sleep/shutdown/machineReboot '{"reason":"remoteKey"}'`, or *Settings → General → Reboot*). To also protect the TV's *current*
running session without rebooting it, launch just the background watcher block (the `( ... ) &` part) by hand over
the same SSH/Telnet session, with `nohup` so it survives the session closing:

```sh
nohup sh -c '... the ( ... ) block's body, unindented ...' >> /tmp/freemytv-autostart.log 2>&1 < /dev/null &
```

## Checking it

```sh
cat /tmp/freemytv-autostart.log                       # every attempt and every standby wake, timestamped
ps -ef | grep getPowerState                            # the background watcher, if it's running
luna-send -n 1 -a com.webos.service.applicationmanager luna://com.webos.applicationManager/getForegroundAppInfo '{}'
```

## Caveats

- **Don't touch** `/media/developer/apps/usr/palm/services/org.webosbrew.hbchannel.service/jumpstart.sh` or
  `/var/lib/webosbrew/startup.sh` — only add scripts under `init.d/`, as the Homebrew Channel's own warning in
  `jumpstart.sh` says.
- An LG software update, a factory reset, or re-running the root exploit from scratch removes this (and the root
  itself) — reinstall the script the same way afterward.
- This fights Live TV only for a short window after boot/wake (2 minutes for the boot-time loop; the standby watcher
  reacts within a few seconds of waking). If you deliberately switch to Live TV yourself during that window, nothing
  stops you — the loop only keeps relaunching Free My TV while something *else* already holds the foreground, never
  the other way around.
