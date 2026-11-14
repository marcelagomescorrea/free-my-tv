# Remapping the remote's Home and a colored button (rooted LG TVs)

Another optional, TV-side setup for people who have **rooted their LG TV** (see
[`root-autostart.md`](root-autostart.md) for the autostart script this complements). It is not part of the app,
not built or installed by `webos/build.py`/`webos/install.py`, and lives entirely on the TV's own filesystem via a
third-party tool — documented here only so the setup survives being forgotten.

## What it does

- The remote's **Home** button opens **Free My TV** directly, instead of LG's own launcher.
- The remote's **red** button opens LG's own launcher (`com.webos.app.home`) — a backup way back to it, since Home
  no longer does that.
- Every other button on the remote (arrows, OK, Back, numbers, app buttons, volume, etc.) keeps working exactly as
  before.

## The tool: Magic Mapper

[andrewfraley/magic_mapper](https://github.com/andrewfraley/magic_mapper) (not part of this project, MIT-licensed,
root required). It grabs the Magic Remote's input device exclusively, re-emits every button it isn't told to
remap to a second device so the TV keeps seeing them, and runs a small JSON-configured action for the ones it is
told to remap.

**Why this one, and not [LG Input Hook](https://repo.webosbrew.org/apps/org.webosbrew.inputhook/)**: LG Input Hook
(the Homebrew Channel app, documented as a general option) was tried first — it correctly *detected* every key
press (visible in its own logs) but its "Launch App" and "Execute" actions never actually ran anything, for any
button tested (Home, and the TV's own secondary color button), on this TV's exact firmware. Reading the remote's
raw kernel input device directly (`/dev/input/event*`) was tried next and saw nothing at all, on any of the 32
device nodes, across several attempts — the Magic Remote doesn't surface through a plain `cat`/`od` read the way a
regular USB keyboard would. Magic Mapper's own Python code opens the same device
(`LGE M-RCU - Builtin [0]`, `/dev/input/event3` on this TV) through Python's evdev handling (with the exclusive-grab
ioctl), which is what actually sees the button presses; a bare read without that doesn't.

## Installing it

With root Telnet/SSH (see `root-autostart.md`'s instructions for enabling it), on the TV:

```sh
curl -sL -o /home/root/magic_mapper.py "https://raw.githubusercontent.com/andrewfraley/magic_mapper/main/magic_mapper.py"
curl -sL -o /var/lib/webosbrew/init.d/start_magic_mapper "https://raw.githubusercontent.com/andrewfraley/magic_mapper/main/start_magic_mapper"
chmod +x /home/root/magic_mapper.py /var/lib/webosbrew/init.d/start_magic_mapper
```

The config (`/home/root/magic_mapper.py`'s directory — i.e. `/home/root/magic_mapper_config.json`) needs writing as
a **single line**: this TV's interactive Telnet session dropped lines from a multi-line `cat << 'EOF'` heredoc paste
(the likely cause of the first attempt silently doing nothing), while a one-line `echo '...' > file` worked
reliably:

```sh
echo '{"magic_mapper_settings":{"block_mouse":false,"exclusive_mode":true,"ir_input_enabled":false,"ir_exclusive_mode":true},"home":{"function":"launch_app","inputs":{"app_id":"io.freemytv"}},"red":{"function":"launch_app","inputs":{"app_id":"com.webos.app.home"}}}' > /home/root/magic_mapper_config.json
```

Then start it (`/var/lib/webosbrew/init.d/start_magic_mapper`, already in place, also runs this at every real boot —
see `root-autostart.md`'s note on what counts as a real boot vs. waking from standby, which doesn't apply here since
Magic Mapper doesn't need to re-grab anything on wake, only on a fresh process start):

```sh
/var/lib/webosbrew/init.d/start_magic_mapper start    # or: restart, after editing the config
```

## Checking it

```sh
cat /tmp/magic_mapper.log       # which input/output devices it resolved, and each button's action
ps -ef | grep magic             # the restart-loop shell and the running magic_mapper.py
```

A healthy log ends with `Input loop started, waiting for button presses on 1 device(s)`.

## Changing the mapping

Edit `/home/root/magic_mapper_config.json` (single line, as above — add more button names as more top-level keys,
each `{"function": "launch_app", "inputs": {"app_id": "<app id>"}}` to open an app, or see the script's own
docstrings for its other functions), then `start_magic_mapper restart`. Button names Magic Mapper knows (from its
own source): `home`, `back`, `ok`, `up`/`down`/`left`/`right`, `red`/`green`/`yellow`/`blue`, number keys `0`-`9`,
`guide`, `info`, `settings`, `search`, `voice`, the service app buttons (`netflix`, `prime`, `disney`, `alexa`,
`lg_channels`, …) and a few more — `grep -n '": "' magic_mapper.py` near its `KEYCODE_MAP` lists them all.

## Caveats

- **`exclusive_mode: true`** (the default, kept here) means Magic Mapper's own process has to be the one re-sending
  every unmapped button; if it crashes or is stopped without restarting, the remote can stop responding until it's
  started again (`start_magic_mapper start`) or the TV reboots. The `start_magic_mapper run` loop restarts it
  automatically after a crash, with a back-off after repeated fast crashes — see the script's own comments.
- An LG software update, a factory reset, or re-rooting the TV removes this — reinstall the same way afterward.
- Unrelated to `root-autostart.md`'s script, which still separately makes sure Free My TV is what's on screen after
  a real reboot or waking from standby; this one only changes what two remote buttons do while the TV is already on.
