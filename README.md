# what

ena maps a Linux keyboard and mouse onto Android touch for fullscreen (probably not)
[scrcpy](https://github.com/Genymobile/scrcpy). pretty cool huh  
check out [phantom](https://github.com/oliviermugishak/phantom) for original documentation thanks


## prep

- Linux, preferably a Wayland session
- scrcpy
- access to `/dev/input/event*` (root via `sudo -E`, or your user in group `input`)
- a known Waydroid screen size written into `~/.config/ena/config.toml`

you only need `/dev/uinput` if you use the legacy `uinput` backend

Hyprland, Sway, niri, and KDE are the compositors where the owned cursor overlay
works; GNOME can still inject menu-touch but it will not show that overlay

## who

don't mix the privileges or you'll break your first install

| what | who starts it | privilege |
|---|---|---|
| `ena --daemon` | you, from the **graphical** session | `sudo -E`, or unprivileged after udev + group `input` |
| `enanan` | you, as the desktop user | never sudo |
| `ena status`, `load`, `enter-capture` | desktop user | talks to `$XDG_RUNTIME_DIR/phantom.sock` |

`sudo -E` matters. plain `sudo` or `pkexec` drops `WAYLAND_DISPLAY`,
`XDG_RUNTIME_DIR`, and cursor-theme variables => the overlay cannot see the
compositor and the owned cursor disappears  
please do not enable the system systemd unit as your daily driver

AppImage is a GUI plus a portable jar; and cannot start the privileged daemon
from its FUSE mount
instead, use a `.deb`, Arch package, tarball, or `./install.sh`
for the daemon

## installation

**from source**, after you have Rust and Android SDK command-line tools:

```bash
git clone https://github.com/helloreunified/ena.git
cd ena
./install.sh
```

That puts `ena` and `enanan` in `~/.local/bin`, the Android jar in
`~/.local/share/ena/android/`, a sudo-visible launcher in `/usr/local/bin`
when possible, and seeds missing profiles into `~/.config/ena/profiles/`

**from a GitHub Release**, pick the asset that matches how you install software:

- Debian / Ubuntu: not provided
- Arch: not provided
- portable: `phantom-v<version>-linux-x86_64.tar.gz`
- GUI tryout only: the AppImage

packaged installs already contain `phantom-server.jar` (don't need to build it)  
the daemon finds it under `/usr/lib/phantom/` or `../lib/phantom/` next to the
binary  
the first GUI or daemon start writes `~/.config/phantom/config.toml` only if that file is missing, and copies shipped profiles into the user library
the same way.

full setup, including udev and Android SDK, is in [docs/INSTALL.md](docs/INSTALL.md) of [phantom](https://github.com/oliviermugishak/phantom) repository

## how to start a session

1. start scrcpy (you may want to adjust commandline parameters if needed e.g. multiple adb entries)

```bash
scrcpy &>/dev/null &
```

2. (blah blah yapping yappity this is no way correct yet) Set `[screen]` in `~/.config/phantom/config.toml` to the real Waydroid
   surface. `waydroid prop get persist.waydroid.width` and `...height` are the
   usual source of truth.

3. start the daemon from the same graphical session:

```bash
sudo -E ena --daemon
```

add `--trace` when you need lifecycle logs
use `PHANTOM_TRACE_DETAIL=1` only when you want raw evdev bugs

4. in a normal shell:

```bash
ena status
ena load ~/.config/ena/profiles/pjsekai.json
enanan
```

## What do capture, aim, and menu-touch actually mean?
explanation is in [docs/INSTALL.md](docs/INSTALL.md) of [phantom](https://github.com/oliviermugishak/phantom) repository

## Where do profiles live, and why is the GUI empty?

two directories:

- shipped starters in the repo (probably removing those presets yeah)
- where the thing actually reads: `~/.config/ena/profiles/`

it never live-reads the repository. at first launch, goto Settings → "Seed shipped
profiles", or `./install.sh` copies missing files into the user library and
does not overwrite edits

shipped starters are layouts, not finished configs for every device:

`pubg.json`, `pubg-small.json`, `genshin.json`, `efootball-template.json`,
`temple-run.json`, `subway-surfers.json`, `asphalt8.json`, `asphalt9.json`

a profile is rejected if its `screen` does not match the daemon screen (that's intentional  
it's not guessing things and only take explicitly true profiles)

how to author nodes is in [docs/PROFILES.md](docs/PROFILES.md) of [phantom](https://github.com/oliviermugishak/phantom) repository  
how to structure a shooter or a swipe runner is in [docs/GAME_PATTERNS.md](docs/GAME_PATTERNS.md) of [phantom](https://github.com/oliviermugishak/phantom) repository  

## What can I bind, and what should I not expect?
read the thing in [phantom](https://github.com/oliviermugishak/phantom) repository

## Why does the owned cursor vanish, or land in the wrong place?
read the thing in [phantom](https://github.com/oliviermugishak/phantom) repository

## which for what

```bash
ena --daemon          # start the mapper; use sudo -E from a desktop session
ena status            # capture, mouse mode, grab, menu-touch backend
ena load <profile>    # replace the live profile
ena enter-capture     # same as F8 on
ena exit-capture      # same as F8 off
ena toggle-mouse      # same as F1: aim <-> menu-touch
ena audit <profile>   # validate without loading
enanan               # editor and runtime controls
```

`grab-mouse` / `release-mouse` are the older names for switching aim and
menu-touch; they do not ungrab the physical mouse while capture is on

## next reads
probably you should read the thing in [phantom](https://github.com/oliviermugishak/phantom) repository instead

operator path:

1. [docs/INSTALL.md](docs/INSTALL.md)
2. [docs/OPERATIONS.md](docs/OPERATIONS.md)
3. [docs/TROUBLESHOOT.md](docs/TROUBLESHOOT.md)
4. [docs/PROFILES.md](docs/PROFILES.md)
5. [docs/GAME_PATTERNS.md](docs/GAME_PATTERNS.md)

maintainer path:

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- [docs/TESTING.md](docs/TESTING.md)
- [docs/RELEASING.md](docs/RELEASING.md)
- [docs/ROADMAP.md](docs/ROADMAP.md)
- [AGENTS.md](AGENTS.md) and [CONTRIBUTING.md](CONTRIBUTING.md)
