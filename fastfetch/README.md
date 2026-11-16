## 1. Install fastfetch

| System | Command |
|---|---|
| Arch / CachyOS | `sudo pacman -S fastfetch` |
| Debian / Ubuntu | `sudo apt install fastfetch` |
| Fedora | `sudo dnf install fastfetch` |
| macOS | `brew install fastfetch` |
| FreeBSD | `pkg install fastfetch` |
| Termux (Android) | `pkg install fastfetch` |

Then check that the installed version can move the logo automatically:
```sh
fastfetch -h logo-position    # the list must include "auto"
```
If `auto` isn't listed, that version is too old. Install a newer one from https://github.com/fastfetch-cli/fastfetch/releases. Distro repos are often behind.

## 2. Find where fastfetch looks for its config

```sh
fastfetch --list-config-paths
```
On Linux, macOS, BSD and Termux this normally includes `~/.config/fastfetch/`. Use whichever path it lists.

## 3. Create `~/.config/fastfetch/config.jsonc`

```sh
mkdir -p ~/.config/fastfetch
```
Save this as `config.jsonc` in that folder:
```jsonc
// Compact fastfetch layout that works at any width.
// "position": "auto" puts the logo on top on narrow terminals, left otherwise.
{
  "logo": { "type": "small", "position": "auto", "padding": { "top": 0, "left": 1 } },
  "display": {
    "separator": " ",
    "key": { "width": 6 },
    "disableLinewrap": true,
    "duration": { "abbreviation": true, "spaceBeforeUnit": "never" },
    "size": { "binaryPrefix": "jedec", "maxPrefix": "GB", "ndigits": 1 }
  },
  "modules": [
    "title",
    { "type": "os",       "key": "OS",   "format": "{name}" },
    { "type": "kernel",   "key": "Kern", "format": "{release}" },
    { "type": "uptime",   "key": "Up" },
    { "type": "packages", "key": "Pkgs", "format": "{all}" },
    { "type": "shell",    "key": "Sh" },
    { "type": "cpu",      "key": "CPU",  "format": "{name} ({cores-logical})" },
    { "type": "gpu",      "key": "GPU",  "format": "{name}" },
    { "type": "display",  "key": "Res",  "format": "{width}x{height}@{refresh-rate}Hz" },
    { "type": "memory",   "key": "Mem",  "format": "{used}/{total} ({percentage})" },
    { "type": "swap",     "key": "Swap", "format": "{used}/{total} ({percentage})" },
    { "type": "disk",     "key": "Disk", "folders": "/", "format": "{size-used}/{size-total} ({size-percentage})" },
    { "type": "localip",  "key": "IP",   "format": "{ipv4}" },
    { "type": "battery",  "key": "Bat",  "format": "{capacity} {status}" }
  ]
}
```
Items a system doesn't have are left out quietly, since fastfetch hides errors unless you turn them on. For example, a desktop without a battery shows no `Bat` line, and a headless server without a screen shows no `Res` line. That lets one file work on every system.

## 4. Add the greeting to your shell's startup file

Add one line at the top of the file your shell runs at startup:

| Shell | File | Line |
|---|---|---|
| zsh | `~/.zshrc` (above the Powerlevel10k instant-prompt block, if you use it) | `fastfetch` |
| bash | `~/.bashrc` (on macOS, also make sure `~/.bash_profile` sources it) | `[[ $- == *i* ]] && fastfetch` |
| fish | `~/.config/fish/config.fish` | `status is-interactive; and fastfetch` |

Don't wrap `fastfetch` in a shell function or alias; the config handles the width now. If a shell that's already open still has the old function from before, run `unfunction fastfetch` in zsh or `unset -f fastfetch` in bash.

## 5. Test it

```sh
script -qc 'stty rows 50 cols 40; fastfetch' /dev/null    # phone width: logo on top
script -qc 'stty rows 50 cols 120; fastfetch' /dev/null   # wide: logo on the left
```
Set `rows` as well as `cols`. Without a height the fake terminal reports 0 rows and fastfetch always puts the logo on the left. `script` takes different options on macOS and BSD, so there it's easier to just resize a real terminal window.
